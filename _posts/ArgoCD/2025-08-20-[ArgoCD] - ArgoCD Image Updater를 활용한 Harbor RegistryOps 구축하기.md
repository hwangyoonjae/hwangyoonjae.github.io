---
layout: post
title: "ArgoCD Image Updater를 활용한 Harbor RegistryOps 구축하기"
date: 2025-08-20
categories: ArgoCD
tags: [ArgoCD, ImageUpdater]
image: /assets/img/post-title/argocd-wallpaper.jpg
---

## ArgoCD Image Updater 설치파일 다운로드:

```bash
$ wget https://raw.githubusercontent.com/argoproj-labs/argocd-image-updater/stable/manifests/install.yaml
```

![argo-image-updater manifest 다운로드](/assets/img/post/ArgoCD/argo-image-updater%20manifest%20다운로드.png)

---

## ArgoCD Image Updater 인증 설정하기:
### TLS 인증서 적용하기:

```bash
$ kubectl -n [namespace] create secret generic [secret명] --from-file=ca.crt=[인증서 경로]
```

> 필자는 아래 그림과 같이 사설 인증서를 생성하여 테스트 하였습니다.
{: .prompt-warning}

![argo-image-updater TLS 인증서 secret 생성](/assets/img/post/ArgoCD/argo-image-updater%20TLS%20인증서%20secret%20생성.png)

---

### Harbor 인증 Secret 생성하기:

```bash
$ kubectl create secret docker-registry [secret명] \
    --docker-server=[Harbor_URL] \
    --docker-username=[id] \
    --docker-password='[password]' \
    --docker-email=[email] \
    -n [namespace]
```

![argo-image-updater harbor 인증 secret 생성](/assets/img/post/ArgoCD/argo-image-updater%20harbor%20인증%20secret%20생성.png)

---

### registries.conf ConfigMap 생성하기:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  labels:
    app.kubernetes.io/name: argocd-image-updater-config
    app.kubernetes.io/part-of: argocd-image-updater
  name: argocd-image-updater-config
  namespace: argocd
# 아래 내용 추가
data:
  registries.conf: |
    registries:
      - name: harbor
        api_url: https://[Harbor_URL]
        prefix: [Harbor 주소]
        insecure: false # TLS 적용
        credentials: pullsecret:[namespace]/[secret명]
```

---

## ArgoCD Image Updater Deployment 수정하기:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/name: argocd-image-updater
    app.kubernetes.io/part-of: argocd-image-updater
  name: argocd-image-updater
spec:

```

## ArgoCD Application Annotations 추가하기:

```bash
$ kubectl edit application [application명] -n [namespace]
```

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  # 아래 내용 추가
  annotations:
    argocd-image-updater.argoproj.io/image-list: nginx=[Harbor_Project_URL]
    argocd-image-updater.argoproj.io/update-strategy.nginx: digest
  creationTimestamp: "2025-08-20T05:10:16Z"
  generation: 25
  name: hug-test
  namespace: argocd
  resourceVersion: "7896800"
  uid: 9e3174cb-4353-4b8e-9a5d-501885b5ca1c
```

> 아래와 같이 ArgoCD 웹 페이지에서도 수정 가능합니다.
{: .prompt-info}

![argocd application annotations 웹에서 추가](/assets/img/post/ArgoCD/argocd%20application%20annotations%20웹에서%20추가.png)

---