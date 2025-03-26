---
layout: post
title: "[Gitlab] - CI/CD 파이프라인 모니터링"
date: 2025.02.12
categories: Gitlab Install
tags: [Git, branch]
image: /assets/img/post-title/gitLab-wallpaper.jpg
---

## gitlab-ci-pipelines-exporter 다운로드 :
- Gitlab Repo 통해서 gitlab-ci-pipelines-exporter 다운로드
> * [gitlab-ci-pipelines-exporter 다운로드](https://github.com/mvisonneau/gitlab-ci-pipelines-exporter "gitlab-ci-pipelines-exporter 다운로드")

* * *

- Container Image 버전 확인하기

```bash
# gitlab-ci-pipeline-exporter, grafana, Prometheus 이미지 다운로드
$ vi {PATH}/gitlab-ci-pipelines-exporter-main/examples/quickstart/docker-compose.yml
```

```yaml
---
version: '3.8'
services:
  gitlab-ci-pipelines-exporter:
    # image: quay.io/mvisonneau/gitlab-ci-pipelines-exporter:v0.5.10
    image: mvisonneau/gitlab-ci-pipelines-exporter:latest
    .
    .
  prometheus:
    image: prom/prometheus:v2.44.0
    .
    .
  grafana:
    image: grafana/grafana:9.5.2
    .
    .
```

> 필자는 docker hub 통해서 이미지 다운로드 받아 docker.io는 생략하여 이미지 지정하였습니다.
{: .prompt-info}

* * *

- Container Image 다운로드

```bash
$ vi image_list.txt

# 다운받을 이미지 목록 작성
docker.io/mvisonneau/gitlab-ci-pipelines-exporter:latest
docker.io/prom/prometheus:v2.44.0
docker.io/grafana/grafana:9.5.2
```

```bash
$ vi save-image.sh
# 아래 스크립트 내용 복사하여 스크립트 생성 후 실행
----------------------------------
#!/bin/bash

# 이미지 목록을 저장한 파일
image_list_file="image_list.txt"

# 저장할 디렉토리
output_dir="."

# 이미지 목록 파일이 존재하는지 확인
if [ ! -f "$image_list_file" ]; then
    echo "이미지 목록 파일 '$image_list_file'을 찾을 수 없습니다."
    exit 1
fi

# 디렉토리를 생성하고 이미지를 pull & save
mkdir -p $output_dir

while IFS= read -r image; do
    echo "Pulling image: $image"
    podman pull $image

    # 이미지 이름과 태그를 추출하여 .tar 파일 이름 생성
    image_name=$(echo $image | awk -F/ '{print $NF}')
    output_file="$output_dir/${image_name//:/_}.tar"

    echo "Saving image to $output_file"
    podman save -o $output_file $image

    echo "Image saved: $output_file"
done < "$image_list_file"

echo "모든 이미지가 pull 및 save되었습니다."
```

```bash
# 다운로드 받은 이미지 Pull 진행
$ docker load -i gitlab-ci-pipelines-exporter_latest.tar
$ docker load -i grafana_9.5.2.tar
$ docker load -i prometheus_v2.44.0.tar
```

* * *

- Container Image 다운로드 확인하기

```bash
$ docker images
```

* * *

## gitlab-ci-pipelines-exporter 설치하기 :

- docker-compose.yml 내용 수정하기

```bash
$ vi /gitlab-ci-pipelines-exporter-main/examples/quickstart/docker-compose.yml
```

```yaml
---
version: '3.8'
services:
  gitlab-ci-pipelines-exporter:
    image: mvisonneau/gitlab-ci-pipelines-exporter:latest
    # You can comment out the image name and use the following statement
    # to build the image against the current version of the repository
    # build: ../..
    ports:
      - 8080:8080
    environment:
      #GCPE_GITLAB_TOKEN: ${GCPE_GITLAB_TOKEN}
      GCPE_GITLAB_TOKEN: "{깃랩 토큰 값}" # Access Token 입력
      GCPE_CONFIG: /etc/gitlab-ci-pipelines-exporter.yml
      GCPE_INTERNAL_MONITORING_LISTENER_ADDRESS: tcp://127.0.0.1:8082
      GCPE_GITLAB_SSL_VERIFY: "false"
    volumes:
      - type: bind
        source: ./gitlab-ci-pipelines-exporter.yml
        target: /etc/gitlab-ci-pipelines-exporter.yml
      - /etc/hosts:/etc/hosts
      - /etc/localtime:/etc/localtime:ro
      - ./ssl/gitlab.com.crt:/etc/ssl/certs/ca-certificates.crt
      # ssl 인증서는 Gitlab에서 사용하는 인증서로 지정하면됩니다.

  prometheus:
    image: prom/prometheus:v2.44.0
    ports:
      - 9090:9090
    links:
      - gitlab-ci-pipelines-exporter
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - ./prometheus/config.yml:/etc/prometheus/prometheus.yml

  grafana:
    image: grafana/grafana:9.5.2
    ports:
      - 3000:3000
    environment:
      GF_AUTH_ANONYMOUS_ENABLED: 'true'
      GF_INSTALL_PLUGINS: grafana-polystat-panel,yesoreyeram-boomtable-panel
    links:
      - prometheus
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - ./grafana/dashboards.yml:/etc/grafana/provisioning/dashboards/default.yml
      - ./grafana/datasources.yml:/etc/grafana/provisioning/datasources/default.yml
      - ./grafana/dashboards:/var/lib/grafana/dashboards

networks:
  default:
    driver: bridge
```

> 필자는 Gitlab 접속 주소를 도메인으로 지정하여 컨테이너 內 hosts 파일을 복사하였습니다.
{: .prompt-warning}

* * *

- gitlab-ci-pipelines-exporter.yml 내용 수정하기

```bash
$ cd /gitlab-ci-pipelines-exporter-main/examples/quickstart/
$ vi gitlab-ci-pipelines-exporter.yml
```

```yaml
---
log:
  level: debug
gitlab:
  url: https://gitlab.com/  # Gitlab 주소 입력
  token: {깃랩 토큰 값}    # Access Token 입력
wildcards:
  - group: "my-project"  # 그룹/서브그룹을 여기에 설정
# 프로젝트들을 자동으로 검색하여 가져옵니다.
projects:
  - wildcard: true  # wildcard를 사용하여 자동으로 프로젝트를 조회
    pull:
      pipeline:
        jobs:
          enabled: true
```

* * *

- Gitlab Repo 주소 변경하기

```bash
$ cd /gitlab-ci-pipelines-exporter-main/examples/quickstart/grafana/dashboards/
$ vi dashboard_pipelines.json

# 아래와 같이 도메인 주소 변경
:%s/gitlab.com/gitlab.test.com/g
```

> Gitlab Repo 도메인 주소는 고객사마다 다를 수 있으니 참고 부탁드립니다.
{: .prompt-warning}

* * *

## gitlab-ci-pipelines-exporter 다운로드(폐쇄망인 경우우) :

- grafana 플러그인 다운로드

> * [yesoreyeram-boomtable-panel 플러그인 다운로드](https://github.com/yesoreyeram/yesoreyeram-boomtable-panel/releases "yesoreyeram-boomtable-panel 플러그인 다운로드")
> * [grafana-polystat-panel 플러그인 다운로드](https://github.com/grafana/grafana-polystat-panel/releases "grafana-polystat-panel 플러그인")

![grafana 모니터링 플러그인 사용](/assets/img/post/Gitlab/grafana%20모니터링%20플러그인%20사용.png)

- grafana 플러그인 압축 파일 복사

```bash
$ unzip yesoreyeram-boomtable-panel-1.6.0.zip
$ unzip grafana-polystat-panel-2.1.14.any.zip

$ cd /gitlab-ci-pipelines-exporter-main/examples/quickstart/grafana/
$ mkdir plugins

$ cp yesoreyeram-boomtable-panel-1.6.0 {grafana path}
$ cp grafana-polystat-panel-2.1.14.any {grafana path}
```

- grafana plugin 컨테이너 볼륨 설정

```bash
$ vi /gitlab-ci-pipelines-exporter-main/examples/quickstart/docker-compose.yml

---
version: '3.8'
services:
  .
  .
  .
grafana:
  .
  .
  volumes:
      - ./grafana/plugins:/var/lib/grafana/plugins  # 내용 추가가
```

- grafana container 재시작

```bash
$ docker restart <grafana-container-name>
```

* * *

## gitlab-ci-pipelines-exporter(Gitlab CI/CD 파이프라인 메트릭 수집기) 접속하기 :

- gitlab-ci-pipelines-exporter 주소 (http://서버IP주소:8080/metrics) 접속하여 CI/CD 파이프라인 메트릭 수집 데이터가 표기되는지 확인합니다.

> Prometheus가 이 URL을 스크랩하여 GitLab의 빌드, 배포, 실패율 등 CI/CD 관련 메트릭을 수집합니다.
{: .prompt-info}

* * *

### 주요 메트릭 예시 :

```bash
# 파이프라인 실행 상태
gitlab_ci_pipeline_last_run_status{project="my-project", ref="main", status="success"} 1
```

```bash
# 파이프라인 실행 시간
gitlab_ci_pipeline_duration_seconds{project="my-project", ref="main"} 120
```

```bash
# 파이프라인 실행 상태
gitlab_ci_pipeline_status{kind="branch", project="my-project", ref="main", source="web", status="success", topics="", variables=""} 1
```

* * *

## prometheus(모니터링) 페이지 접속하기 :
- Prometheus (http://서버IP주소:9090/targets) 접속하여 모니터링하는 대상의 상태를 확인합니다.

> Status가 "UP"으로 되어있어야합니다.
{: .prompt-warning}

![gitlab pipeline prometheus 모니터링](/assets/img/post/Gitlab/gitlab%20pipeline%20prometheus%20모니터링.png)

* * *

## grafana(대시보드) 페이지 접속하기 :

- Grafana 접속(http://서버IP주소:3000)하여 오른쪽 상단에 “Sign in” 버튼을 눌러 로그인 진행합니다.

![gitlab pipeline grafana 로그인 1](/assets/img/post/Gitlab/gitlab%20pipeline%20grafana%20로그인%201.png)

- 초기 패스워드는 아이디와 동일하게 입력합니다.

> ID : admin / PW : admin
{: .prompt-example}

![gitlab pipeline grafana 로그인 2](/assets/img/post/Gitlab/gitlab%20pipeline%20grafana%20로그인%202.png)

- 필요에 따라 패스워드 변경하시거나, “Submit“ 버튼 왼쪽 아래의 “Skip“ 버튼을 클릭하여 넘어갑니다.

![gitlab pipeline grafana 로그인 3](/assets/img/post/Gitlab/gitlab%20pipeline%20grafana%20로그인%203.png)

- 왼쪽 상단에 토글 버튼 > Dashboards > Gitlab CI pipelines 클릭합니다.

![gitlab pipeline grafana 대시보드](/assets/img/post/Gitlab/gitlab%20pipeline%20grafana%20대시보드.png)

- 모니터링 화면에서 불필요한 목록은 삭제합니다.

![gitlab pipeline grafana 모니터링 초기 화면](/assets/img/post/Gitlab/gitlab%20pipeline%20grafana%20모니터링%20초기%20화면.png)
![gitlab pipeline grafana 모니터링 수정 화면](/assets/img/post/Gitlab/gitlab%20pipeline%20grafana%20모니터링%20수정%20화면.png)

- 추후 사용을 위해 대시보드 데이터 파일을 저장합니다.

> 만약 서버 고장으로 인해 재설치가 필요한 경우 해당 데이터 파일 사용을 위해 저장합니다.
{: .prompt-warning}

![gitlab pipeline grafana 모니터링 저장](/assets/img/post/Gitlab/gitlab%20pipeline%20grafana%20모니터링%20저장.png)
![gitlab pipeline grafana 모니터링 json 저장](/assets/img/post/Gitlab/gitlab%20pipeline%20grafana%20모니터링%20json%20저장.png)

- 수정한 대시보드 사용을 위해 다른 이름으로 저장합니다.

> 다름 이름으로 저장 시 Dashboard Name은 운영자가 구별하기 쉽게 지정하시면 됩니다.
{: .prompt-warning}

![gitlab pipeline grafana 모니터링 대시보드 세팅](/assets/img/post/Gitlab/gitlab%20pipeline%20grafana%20모니터링%20대시보드%20세팅.png)
![gitlab pipeline grafana 모니터링 대시보드 세팅 저장](/assets/img/post/Gitlab/gitlab%20pipeline%20grafana%20모니터링%20대시보드%20세팅%20저장.png)
![gitlab pipeline grafana 모니터링 대시보드 세팅 다른이름저장](/assets/img/post/Gitlab/gitlab%20pipeline%20grafana%20모니터링%20대시보드%20세팅%20다른이름저장.png)
![gitlab pipeline grafana 모니터링 대시보드 세팅 다른이름저장 후 화면](/assets/img/post/Gitlab/gitlab%20pipeline%20grafana%20모니터링%20대시보드%20세팅%20다른이름저장%20후%20화면.png)

* * *