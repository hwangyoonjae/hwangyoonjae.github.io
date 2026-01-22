---
layout: post
title: "Helm Chart를 통해 Cloud Native Gitlab 설치하기"
date: 2026-01-08
categories: [컨테이너, Helm] 
tags: [Helm, Gitlab]
image: /assets/img/post-title/helm-wallpaper.jpg
---

> Omnibus(통합 컨테이너 1개)로 설치했던 Gitlab을 Cloud Native(서비스 분리, 여러 Pod) 형태로 변경하여 설치해도록 하겠습니다.
{: .prompt-info}

> 해당 구축을 진행하게 된 계기는 고객사 환경의 통합 컨테이너로 구축하였으나, 리소스 부족으로 인해 지속적으로 서버가 다운되는 경우가 있었습니다.
>
> 이에 필자는 해결방안을 고민하던 중, 통합된 컨테이너를 분리하여 구성하면 좋을거 같아 설치 진행하였습니다.
{: .prompt-warning}

* * *

## 1. GitLab Cloud Native 서비스 목록 :

| 컴포넌트 | 역할 | 비고 |
|:--------:|:------:|:------:|
| Webservice (Rails / Puma) | GitLab UI 및 API 제공 | 메인 웹 애플리케이션 |
| Sidekiq | 백그라운드 잡 처리 | CI 처리, 메일, 정리 작업 등 |
| Gitaly | Git 저장소 접근 (RPC) | Git 데이터 실질 처리 |
| GitLab Shell | SSH 기반 Git 접속 | git clone/push over SSH |
| NGINX (Ingress 역할) | 외부 트래픽 라우팅 | HTTP/HTTPS 진입점 |
| Registry | 컨테이너 이미지 레지스트리 | 선택 사항 (Harbor 대체 가능) |
| KAS (Kubernetes Agent Server) | GitLab ↔ Kubernetes 연동 | 선택 사항 |
| Praefect | Gitaly HA 라우팅 계층 | Gitaly 다중화 시 사용 |
| DB (PostgreSQL) | GitLab 메타데이터 저장 | 내장 또는 외부 연동 |
| Redis | 캐시 및 큐 처리 | 내장 또는 외부 연동 |
| Object Storage | 아티팩트/LFS/업로드/백업 저장 | MinIO / S3 계열 |

* * *

## 2. Cloud Native Gitlab 설치하기 :
### 2.1 Gitlab Helm Chart 다운로드 :

```bash
$ helm repo add gitlab https://charts.gitlab.io
$ helm repo update
$ helm search repo gitlab/gitlab
```

![gitlab helm chart 목록 조회](/assets/img/post/helm/gitlab%20helm%20chart%20목록%20조회.png)

* * *

- 폐쇄망에서 진행하는 경우 아래와 같이 진행하면됩니다.

```bash
$ helm pull gitlab/gitlab --version 9.6.1 --destination .
```

* * *

### 2.2 Gitlab Container Image 다운로드 :

- helm template을 통한 컨테이너 이미지 추출하여 이미지를 다운받습니다.

```bash
# values.yaml global.ingress.configureCertmanager: false로 지정 후 아래 작업을 진행합니다.
# values.yaml을 적용했을 때 Kubernetes 리소스(YAML)가 어떻게 생성되는지 미리 렌더링해서 파일로 뽑기
$ helm template gitlab gitlab/gitlab \
  --version {gitlab-version} \
  -n native-gitlab \
  -f values.yaml \
  > rendered.yaml

# 이미지 목록 추출(단순/강력)
$ grep -RohE 'image:\s*[^ ]+' rendered.yaml | awk '{print $2}' | sort -u > images.txt
```

> 위 과정에서 추출한 이미지 목록을 다운받아 이미지 레지스트리에 업로드합니다.
{: .prompt-info}

> 추출한 이미지와 실제 적용하는 이미지의 버전이 약간의 차이가 있을 수 있습니다.
{: .prompt-warning}

* * *

### 2.3 values.yaml 수정하기 :

```yaml
global:
  edition: ce

  hosts:
    domain: test.com
    hostSuffix: "dev" # 서브도메인 필요한 경우 작성 Ex) dev-test.com

  communityImages:
    # Default repositories used to pull Gitlab Community Edition images.
    # See the image.repository and workhorse.repository template helpers.
    migrations:
      repository: harbor.test.com/native-gitlab/gitlab-org/build/cng/gitlab-toolbox-ce
    sidekiq:
      repository: harbor.test.com/native-gitlab/gitlab-org/build/cng/gitlab-sidekiq-ce
    toolbox:
      repository: harbor.test.com/native-gitlab/gitlab-org/build/cng/gitlab-toolbox-ce
    webservice:
      repository: harbor.test.com/native-gitlab/gitlab-org/build/cng/gitlab-webservice-ce
    workhorse:
      repository: harbor.test.com/native-gitlab/gitlab-org/build/cng/gitlab-workhorse-ce

  ingress:
    apiVersion: ""
    configureCertmanager: false
    useNewIngressForCerts: false
    provider: nginx
    class: nginx
    annotations: {}
    enabled: true
    tls:
      enabled: true
      secretName: gitlab-ingress-tls
    path: /
    pathType: Prefix

  initialRootPassword:
    # secret: RELEASE-gitlab-initial-root-password
    # key: password
    plaintext: "qwe1212!Q"

  # 파이프라인 실행 시 artifacts 사용할 경우 활성화
  minio:
    enabled: true

  certificates:
    image:
      repository: harbor.test.com/native-gitlab/gitlab-org/build/cng/certificates

  kubectl:
    image:
      repository: harbor.test.com/native-gitlab/gitlab-org/build/cng/kubectl

  gitlabBase:
    image:
      repository: harbor.test.com/native-gitlab/gitlab-org/build/cng/gitlab-base

installCertmanager: false

certmanager:
  installCRDs: false

nginx-ingress: &nginx-ingress
  enabled: false

haproxy:
  install: false

prometheus:
  install: false

redis:
  install: true
  image:
    registry: harbor.test.com
    repository: native-gitlab/bitnamilegacy/redis
  master:
    persistence:
      enabled: true
      storageClass: nfs-client
      accessModes:
        - ReadWriteMany
      size: 10Gi
  cluster:
    enabled: false
  metrics:
    enabled: true
    image:
      registry: harbor.test.com
      repository: native-gitlab/bitnamilegacy/redis-exporter

postgresql:
  install: true
  image:
    registry: harbor.test.com
    repository: native-gitlab/bitnamilegacy/postgresql
    tag: 16.6.0
  primary:
    persistence:
      enabled: true
      storageClass: nfs-client
      accessModes:
        - ReadWriteMany
      size: 20Gi
  metrics:
    enabled: true
    image:
      registry: harbor.test.com
      repository: native-gitlab/bitnamilegacy/postgres-exporter

shared-secrets:
  enabled: true
  rbac:
    create: true
  selfsign:
    image:
      repository: harbor.test.com/native-gitlab/gitlab-org/build/cng/cfssl-self-sign

gitlab-runner:
  install: false

gitlab:
  ## https://docs.gitlab.com/charts/charts/gitlab/toolbox
  toolbox:
    backups:
      enabled: false
      cron:
        enabled: false
      objectStorage:
        enabled: false
        config:
          secret: gitlab-backup-objectstorage

  gitaly:
    image:
      repository: harbor.test.com/native-gitlab/gitlab-org/build/cng/gitaly
    persistence:
      enabled: true
      storageClass: nfs-client
      accessMode: ReadWriteMany
      size: 50Gi

  gitlab-exporter:
    image:
      repository: harbor.test.com/native-gitlab/gitlab-org/build/cng/gitlab-exporter

  gitlab-shell:
    image:
      repository: harbor.test.com/native-gitlab/gitlab-org/build/cng/gitlab-shell

  kas:
    image:
      repository: harbor.test.com/native-gitlab/gitlab-org/build/cng/gitlab-kas
```

* * *

- minio 서브차트 values.yaml 수정하기

```yaml
# charts/minio/values.yaml
image: "harbor.test.com/native-gitlab/minio/minio"
imageTag: "RELEASE.2017-12-28T01-21-00Z"

minioMc:
  image: "harbor.test.com/native-gitlab/minio/mc"
  tag: "RELEASE.2018-07-13T00-53-22Z"

persistence:
  storageClass: "nfs-client"
  accessMode: ReadWriteMany
  size: 10Gi
```

* * *

- registry 서브차트 values.yaml 수정하기

```yaml
# charts/registry/values.yaml
image:
  repository: harbor.test.com/native-gitlab/gitlab-org/build/cng/gitlab-container-registry
  tag: 'v4.33.0-gitlab'
```

* * *

- 추가로 Gitlab 최초 설치 시 root 계정의 초기 비밀번호를 Kubernetes Secret으로 생성하도록 수정합니다.

> 기본적으로 root 패스워드는 Helm Chart에서 랜덤 비밀번호를 생성하기에 사용자가 직접 비밀번호를 지정해야하는 경우 아래와 같이 변경하면됩니다.
{: .prompt-tip}

{% raw %}
```bash
$ vi templates/shared-secrets/_generate_secrets.sh.tpl

# Initial root password
#generate_secret_if_needed {{ template "gitlab.migrations.initialRootPassword.secret" . }} --from-literal={{ template "gitlab.migrations.initialRootPassword.key" . }}=$(gen_random 'a-zA-Z0-9' 64)
# Initial root password
generate_secret_if_needed {{ template "gitlab.migrations.initialRootPassword.secret" . }} \
  --from-literal={{ template "gitlab.migrations.initialRootPassword.key" . }}={{- if .Values.global.initialRootPassword.plaintext -}}{{ .Values.global.initialRootPassword.plaintext | quote }}{{- else -}}$(gen_random 'a-zA-Z0-9' 64){{- end -}}
```
{% endraw %}

* * *

### 2.4 Gitlab Helm Chart 설치하기 :

```bash
# native-gitlab 설치된 상태에서 진행하는 경우 생성 필요없습니다.
$ kubectl create namespace native-gitlab

# 설치 진행
$ helm upgrade --install native-gitlab ./ -n native-gitlab
```

* * * 

- 설치 후 리소스들이 정상적으로 되었는지 확인합니다.

```bash
# native-gitlab 리소스 전체 확인
$ kubectl get all -n native-gitlab

# native-gitlab pvc 확인
$ kubectl get pvc -n native-gitlab
```

![cloud native gitlab workload 목록 확인](/assets/img/post/helm/cloud%20native%20gitlab%20workload%20목록%20확인.png)

* * *