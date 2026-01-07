---
layout: post
title: "Helm Chart를 통해 ArgoCD Image Updater 설치하기"
date: 2025-12-18
categories: [컨테이너, Helm] 
tags: [Helm, Argocd]
image: /assets/img/post-title/helm-wallpaper.jpg
---

## 1. ArgoCD Image Updater 설치하기 :
### 1.1 ArgoCD Image Updater Helm Chart 다운로드 :

```bash
$ helm repo add argo https://argoproj.github.io/argo-helm
$ helm repo update
$ helm search repo argo/argocd-image-updater --versions | head
```

![argocd image updater helm chart 목록 조회](/assets/img/post/helm/argocd%20image%20updater%20helm%20chart%20목록%20조회.png)

* * *

- 폐쇄망에서 진행하는 경우 아래와 같이 진행하면됩니다.

```bash
$ helm pull argo/argocd-image-updater --version 1.0.2 --destination .
```

* * *

### 1.2 ArgoCD Image Updater Container Image 다운로드 :

```bash
$ docker pull quay.io/argoprojlabs/argocd-image-updater:v1.0.1

$ docker tag quay.io/argoprojlabs/argocd-image-updater:v1.0.1 [HARBOR_DOMAIN]/argocd/argocd-image-updater:v1.0.1
$ docker push [HARBOR_DOMAIN]/argocd/argocd-image-updater:v1.0.1
```

* * *

### 1.3 values.yaml 수정하기 :

- 위 과정에서 다운받은 컨테이너 이미지로 수정합니다.

```yaml
replicaCount: 1
image:
  # -- Default image repository
  repository: [HARBOR_DOMAIN]/argocd/argocd-image-updater # 수정
  # -- Default image pull policy
  pullPolicy: Always
  # -- Overrides the image tag whose default is the chart appVersion
  tag: "v1.0.1" # 수정
```

* * *

### 1.4 ArgoCD Image Updater Helm Chart 설치하기 :


```bash
# argocd가 설치된 상태에서 진행하는 경우 생성 필요없습니다.
$ kubectl create namespace argocd

# 설치 진행
$ helm upgrade --install argocd-image-updater ./ -n argocd
```

* * *

- Helm Chart 설치 후, 파드 목록을 조회하여 정상 설치 되었는지 확인합니다.

```bash
# pod 목록 조회
$ kubectl get po -n argocd
```

![argocd image updater helm chart 배포 후 확인](/assets/img/post/helm/argocd%20image%20updater%20helm%20chart%20배포%20후%20확인.png)

* * *