---
layout: post
title: "Helm Chart를 통해 ArgoCD 설치하기"
date: 2025-10-28
categories: [컨테이너, Helm]
tags: [Helm, ArgoCD]
image: /assets/img/post-title/helm-wallpaper.jpg
---

## 1. ArgoCD 설치하기 :
### 1.1 ArgoCD Helm Chart 다운로드 :
```bash
$ helm repo add argo https://argoproj.github.io/argo-helm
$ helm repo update
```

* * *

- 폐쇄망에서 진행하는 경우 아래와 같이 진행하면 됩니다.

```bash
# 압축 파일 다운로드
$ helm pull argo/argo-cd --version v9.5.16 --destination .

# 압축 파일 풀기
$ tar -zxvf rargo-cd-9.5.16.tgz
```

* * *
### 1.2 ArgoCD Container Image 다운로드 : 

```bash
$ docker pull ghcr.io/dexidp/dex:v2.45.1
$ docker pull ecr-public.aws.com/docker/library/redis:8.2.3-alpine
$ docker pull ghcr.io/oliver006/redis_exporter:v1.84.0
$ docker pull ghcr.io/oliver006/redis_exporter:v1.75.0
$ docker pull ecr-public.aws.com/docker/library/haproxy:3.1.7-alpine
$ docker pull quay.io/argoproj/argocd:v3.4.3

$ docker tag ghcr.io/dexidp/dex:v2.45.1                               harbor.test.com/argocd/dex:v2.45.1
$ docker tag ecr-public.aws.com/docker/library/redis:8.2.3-alpine     harbor.test.com/argocd/redis:8.2.3-alpine
$ docker tag ghcr.io/oliver006/redis_exporter:v1.84.0                 harbor.test.com/argocd/redis_exporter:v1.84.0
$ docker tag ghcr.io/oliver006/redis_exporter:v1.75.0                 harbor.test.com/argocd/redis_exporter:v1.75.0
$ docker tag ecr-public.aws.com/docker/library/haproxy:3.1.7-alpine   harbor.test.com/argocd/haproxy:3.1.7-alpine
$ docker tag quay.io/argoproj/argocd:v3.4.3                           harbor.test.com/argocd/argocd:v3.4.3

$ docker push harbor.test.com/argocd/dex:v2.45.1
$ docker push harbor.test.com/argocd/redis:8.2.3-alpine
$ docker push harbor.test.com/argocd/redis_exporter:v1.84.0
$ docker push harbor.test.com/argocd/redis_exporter:v1.75.0
$ docker push harbor.test.com/argocd/haproxy:3.1.7-alpine
$ docker push harbor.test.com/argocd/argocd:v3.4.3
```

* * *

### 1.3 values.yaml 수정하기 :
- values.yaml 파일에 값을 수정합니다.

```bash
$ vi values.yaml
```
```yaml
global:
  domain: argocd.example.com
  image:
    repository: harbor.test.com/argocd/argocd
    tag: "v3.4.3" # 빈칸으로 둘 경우, Chart.yaml에 있는 appVersion으로 지정됩니다.
    imagePullPolicy: IfNotPresent

# 테스트 환경에서는 redis 추천
redis:
  enabled: true
  image:
    repository: harbor.test.com/argocd/redis
    tag: 8.2.3-alpine
    imagePullPolicy: ""
  exporter:
    enabled: false
    env: []
    image:
      repository: harbor.test.com/argocd/redis_exporter
      tag: v1.84.0
      imagePullPolicy: ""

# 운영 환경에서는 redis-ha 추천
redis-ha:
  enabled: true
  image:
    repository: harbor.test.com/argocd/redis
    tag: 8.2.3-alpine
  exporter:
    enabled: true # Argo CD 상태 정보를 Prometheus가 읽을 수 있게 제공
    image: harbor.test.com/argocd/redis_exporter
    tag: v1.75.0
  persistentVolume: # Redis HA PVC 사용 시
    enabled: true
    storageClass: ceph-block # 필자는 rook-ceph를 사용하여 스토리지클래스를 지정
  haproxy:
    enabled: true
    labels:
      app.kubernetes.io/name: argocd-redis-ha-haproxy
    image:
      repository: harbor.test.com/argocd/haproxy

# SSO 관련
dex:
  enabled: true
  image:
    repository: harbor.test.com/argocd/dex
    tag: v2.45.1
    imagePullPolicy: ""

server:
  ingress:
    enabled: true
    ingressClassName: nginx
    hostname: argocd.example.com
    path: /
    pathType: Prefix
    tls: true
    certificateSecret:
      enabled: true
      secretName: wildcard-tls
```

* * *

- argocd 설치합니다.

```bash
$ helm install argocd ./ -n argocd -f values.yaml
```

![helm chart를 통한 argocd 배포](/assets/img/post/helm/helm%20chart를%20통한%20argocd%20배포.png)

* * *

- 초기 비밀번호는 아래 명령어를 통해서 확인하여 로그인합니다.

```bash
$ kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

![argocd 초기 비밀번호 확인](/assets/img/post/helm/argocd%20초기%20비밀번호%20확인.png)

* * *

## 2. ArgoCD 설치 과정에서 변경사항 :
### 2.1 admin password 지정하고 싶은 경우 :

- values.yaml에서 비밀번호를 직접 지정할 수 있습니다.

```yaml
# configs.secret.argocdServerAdminPassword 값에 bcrypt 해시값을 넣는다.
configs:
  secret:
    ## `htpasswd -nbBC 10 "" $ARGO_PWD | tr -d ':\n' | sed 's/$2y/$2a/'`
    argocdServerAdminPassword: [ argocd_admin_password ]
```

* * *

- bcrypt 해시 생성하여 새 비밀번호를 반영합니다.

```bash
$ htpasswd -nbBC 10 "" '[password]' | tr -d ':\n'
```

* * *

### 2.2 CRD 소유권을 완전히 새로 가져가고 싶은 경우 :

- Argo CD 관련 CRD를 삭제합니다.

```bash
$ kubectl delete crd applications.argoproj.io appprojects.argoproj.io applicationsets.argoproj.io

# 그 다음 정상 설치
$ helm install argocd ./ -n argocd -f values.yaml
```

> Helm 차트가 CRD를 언인스톨 시 자동 제거하지 않는 경우가 많아 이전에 삭제해도 CRD가 남아 충돌이 자주 발생합니다.
{: .prompt-warning}

* * *