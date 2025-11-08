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

- curl 또는 wget으로 직접 다운로드한다.

```bash
$ waget https://github.com/argoproj/argo-helm/releases/download/argo-cd-[version]/argo-cd-[버전].tgz
```

> 필자는 현 시점 최신버전인 **9.0.5**를 사용했다.
{: .prompt-tip}

* * *

### 1.2 Helm을 통해 ArgoCD 설치하기 :

- values.yaml 파일에 값을 수정한다.

```bash
$ vi values.yaml
```
```yaml
# values.yaml
namespaceOverride: [ argocd_namespace ]

global:
  domain: [ argocd_domain ]

  image:
    repository: [ argocd_image_registry ]/[ argocd_namespace ]/argocd
    tag: [ argocd_image_tag ]

server:
  ingress:
    enabled: true
    annotations:
      nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
      nginx.ingress.kubernetes.io/ssl-passthrough: "true"

    ingressClassName: "nginx"

    hostname: [ argocd_domain ]

    path: /
    pathType: Prefix

    tls:
      - secretName: "argocd-server-tls"
        hosts:
          - [ argocd_domain ]

  resources:
    requests:
      cpu: "[ argocd_cpu_request ]"
      memory: "[ argocd_memory_request ]"
    limits:
      cpu: "[ argocd_cpu_limit ]"
      memory: "[ argocd_memory_limit ]"

redis:
  image:
    repository: [ argocd_image_registry ]/[ argocd_namespace ]/redis
    tag: [ argocd_image_version ]
    imagePullPolicy: [ argocd_image_pull_policy ]

configs:
  params:
    server.insecure: true


dex:
  enabled: true # true -> false로 수정
```

* * *

- argocd 설치한다.

```bash
$ helm install argocd ./ -n argocd -f values.yaml
```

![helm chart를 통한 argocd 배포](/assets/img/post/helm/helm%20chart를%20통한%20argocd%20배포.png)

* * *

- 초기 비밀번호는 아래 명령어를 통해서 확인하여 로그인한다.

```bash
$ kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

![argocd 초기 비밀번호 확인](/assets/img/post/helm/argocd%20초기%20비밀번호%20확인.png)

* * *

## 2. ArgoCD 설치 과정에서 변경사항 :
### 2.1 admin password 지정하고 싶은 경우 :

- values.yaml에서 비밀번호를 직접 지정할 수 있다.

```yaml
# configs.secret.argocdServerAdminPassword 값에 bcrypt 해시값을 넣는다.
configs:
  secret:
    ## `htpasswd -nbBC 10 "" $ARGO_PWD | tr -d ':\n' | sed 's/$2y/$2a/'`
    argocdServerAdminPassword: [ argocd_admin_password ]
```

* * *

- bcrypt 해시 생성하여 새 비밀번호를 반영한다.

```bash
$ htpasswd -nbBC 10 "" '[password]' | tr -d ':\n'
```

* * *

### 2.2 CRD 소유권을 완전히 새로 가져가고 싶은 경우 :

- Argo CD 관련 CRD를 삭제한다.

```bash
$ kubectl delete crd applications.argoproj.io appprojects.argoproj.io applicationsets.argoproj.io

# 그 다음 정상 설치
$ helm install argocd ./ -n argocd -f values.yaml
```

> Helm 차트가 CRD를 언인스톨 시 자동 제거하지 않는 경우가 많아 이전에 삭제해도 CRD가 남아 충돌이 자주 발생한다.
{: .prompt-warning}

* * *