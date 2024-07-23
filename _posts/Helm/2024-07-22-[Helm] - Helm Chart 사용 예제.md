---
layout: post
title: "[Helm] - Helm Chart 사용 예제"
date: 2024-07-22
categories: Helm
tags: [Helm, Kubernetes]
image: /assets/img/post-title/helm-wallpaper.jpg
---

## Helm Chart란? :
- Kubernetes 애플리케이션의 패키징 형식
- Kubernetes에서 애플리케이션 배포를 관리하기 위한 도구이며, Helm Chart는 이러한 배포를 정의하는 템플릿

* * *

## Helm Chart 기본 구조 생성 :
- helm create 명령어를 통해 차트 기본 디렉토리를 생성한다.

```bash
$ helm create {폴더명}
```

- 위와 같이 생성된 기본 구조는 아래와 같다.

```bash
폴더명
|-- Chart.yaml
|-- charts
|-- templates
|   |-- NOTES.txt
|   |-- _helpers.tpl
|   |-- hpa.yaml
|   |-- deployment.yaml
|   |-- ingress.yaml
|   |-- service.yaml
|   `-- serviceaccount.yaml
`-- values.yaml
```

| 이름              | 설명              |
| :---------------- | :-------------------------------------------- |
| Chart.yaml        | Chart에 대한 이름, 버전, 설명 등이 정의된 파일 |
| requirements.yaml | Chart에 대한 종속 Chart 정보를 정의한 파일 |
| values.yaml       | Chart설치시 사용할 환경변수를 정의한 파일 |
| charts/           | Chart에서 사용하는 종속 Chart들이 압축파일(tgz)로 존재 |
| templates/        | 설치할 resource 들의 기본 틀을 정의한 manifest yaml파일 |
| _helpers.tpl      | template manifest파일들에서 공유하는 변수 정의 |

* * *

## Chart.yaml 작성하기 :
- Chart.yaml에 정의하는 내용은 아래와 같다.

| 이름 | 설명 |
| :---------------- | :-------------------------------------------- |
| apiVersion | Helm 3 사용하므로 'v2'로 표기 |
| name | Chart명 |
| kubeVersion | Chart설치와 실행을 보장하는 최소 k8s버전으로 SemVer형식 범위로 정의 |
| description | Chart에 대한 요약 설명 |
| keywords | Chart에 대한 keyword이며, 여러개면 대시(-)로 구분된 새로운 라인으로 정의 |
| home | Chart프로젝트의 홈페이지 URL |
| sources | Chart소스를 볼 수 있는 URL, 대시로 구분된 새로운 라인으로 여러개 등록 가능 |
| maintainers | name: 저작자 이름 (required for each maintainer)<br>email: 저작자 email (optional for each maintainer)<br>url: 저작자의 개인 페이지 URL (optional for each maintainer) |
| engine | yaml파일을 생성하는 template엔진명이며 기본값은 gotpl |
| icon | SVG 또는 PNG 포맷의 아이콘 URL이며, 카탈로그에서 차트 표시시 사용 |
| appVersion | Chart를 이용해 서비스되는 앱의 버전이며 SemVer형식을 따르지 않아도 됨 |
| deprecated | deprecated된 차트인지 여부를 true/false로 명시 |
| tillerVersion | 보장하는 tiller버전의 SemVer범위 |

* * *

## values.yaml 작성하기 :
- template 폴더에 있는 manifest yaml파일에 정의된 값을 불러온다.
- '{{ .Values.image.tag }}' 와 같이 사용  인덴테이션에 유의하여 작성하여야한다.

* * *

## templates 폴더에 배포할 resource 정의하기 :
- 기본적으로 deployment, service, serviceaccount,ingress가 존재하고, 필요에 따라 persistentvolume, configmap 등을 정의한다.

* * *

## Helm의 template 문법 정의 :
- {{ }} 로 변수를 사용한다.
  - .Values -> values.yaml 파일에서 정의된 변수
  - .Charts -> Charts.yaml 파일에서 정의 된 변수
  - .Release -> 배포할 때에 할당한 정보들을 사용 (예: --namespeace test 로 install 시 .Release.Namespace 에 test로 할당)
  - include … -> _helpers.tpl 파일에서 정의된 변수
  - -with ~ end -> 변수에 대한 scope을 정하는 문법으로 해당 구간은 . 가 설정한 변수에 속함을 정의

* * *

## Helm Chart 검사하기 :
- 정의한 chart가 문법적으로 이상이 없는지 확인

```bash
$ helm lint <Chart.yaml 경로>
```

- templates을 기반으로 변수들 참조하여 리소스 배포시 결과 미리보기

```bash
$ helm template <Chart.yaml 경로>
```

- chart를 시험설치하여 오류 확인

```bash
$ helm install <release name> <Chart.yaml경로>  --debug --dry-run
```
> --dry-run : 실제 클러스터에 설치 하지 않고 chart를 시험 설치하는 옵션 <br>
> --debug: 배포를 위한 manifest 파일 내용을 보여줌
{: .prompt-info }

* * *

## Chart 생성하고 Repo에 등록하기 :
> helm repository는 차트 저장소의 각 차트의 대한 정보를 담고 있는 index.yaml파일이 있어야한다.
{: .prompt-warning }

```bash
# Chart.yaml에 정의한 <name>-<version>.tgz 로 압축파일이 생성
$ helm package <Chart.yaml 경로>
```

> 저장소에 등록하지않고 생성한 압축파일로 로컬에서 사용할거라면 helm install <name> 가능하다.
{: .prompt-tip }

- chart를 생성하였으니 차트정보를 담고있는 index파일 업데이트

```bash
$ helm repo index <index파일 경로>
```

- 깃헙repository인경우 git push를 진행한 후, chart목록을 repository에서 다시 가져와 업데이트를 진행

```bash
$ helm repo update
```

- 업데이트되었는지 검색하여 확인을 진행

```bash
$ helm search repo 차트명
```

* * *

## 생성한 Chart Repo의 푸시하기 :
- 생성한 Chart를 Repo의 푸시한다.

```bash
$ helm push {package.tgz} {repo명}
```

- 위 명령어 실행 후 아래와 같은 오류 메세지가 발생한다.

> Error: scheme prefix missing from remote (e.g. "oci://") 에러 발생
{: .prompt-warning }

> 발생 이유
>
> helm push 명령을 사용할 때 차트를 업로드할 원격 저장소 주소에 프로토콜 접두사가 없어서 발생한 원인으로
> Helm 3.7.0 이상에서는 차트를 OCI 레지스트리나 ChartMuseum으로 푸시할 때 각각의 프로토콜 접두사를 명시해야한다.
{: .prompt-info }

### 조치 방법 :
```bash
$ helm plugin install https://github.com/chartmuseum/helm-push
```

* * *

## 생성한 Chart를 이용해 설치하기 :
- 생성한 차트를 아래 명령어로 설치한다.

```bash
$ helm install 배포할이름 레포지토리명/chart명 -n 네임스페이스
```

* * *