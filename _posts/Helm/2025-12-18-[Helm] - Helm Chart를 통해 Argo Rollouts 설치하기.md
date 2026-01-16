---
layout: post
title: "Helm Chart를 통해 Argo Rollouts 설치하기"
date: 2025-12-18
categories: [컨테이너, Helm] 
tags: [Helm, Argocd, Rollout]
image: /assets/img/post-title/helm-wallpaper.jpg
---

## 1. Argo Rollouts 설치하기 :
### 1.1 Argo Rollouts Helm Chart 다운로드 :

```bash
$ helm repo add argo https://argoproj.github.io/argo-helm
$ helm repo update
$ helm search repo argo/argo-rollouts --versions | head
```

![argo rollouts helm chart 목록 조회](/assets/img/post/helm/argo%20rollouts%20helm%20chart%20목록%20조회.png)

* * *

- 폐쇄망에서 진행하는 경우 아래와 같이 진행하면됩니다.

```bash
$ helm pull argo/argo-rollouts --version 2.40.5 --destination .
```

* * *

### 1.2 Argo Rollouts Container Image 다운로드 :

```bash
$ docker pull quay.io/argoproj/argo-rollouts:v1.8.3

$ docker tag quay.io/argoproj/argo-rollouts:v1.8.3 harbor.test.com/argocd/argo-rollouts:v1.8.3
$ docker push harbor.test.com/argocd/argo-rollouts:v1.8.3
```

* * *

### 1.3 values.yaml 수정하기 :

- 위 과정에서 다운받은 컨테이너 이미지로 수정합니다.

```yaml
controller:
  image:
    # -- Registry to use
    registry: harbor.test.com
    # --  Repository to use
    repository: argocd/kubectl-argo-rollouts
    # -- Overrides the image tag (default is the chart appVersion)
    tag: "v1.8.3"
    # -- Image pull policy
    pullPolicy: IfNotPresent

dashboard:
  image:
    # -- Registry to use
    registry: harbor.test.com
    # --  Repository to use
    repository: argocd/kubectl-argo-rollouts
    # -- Overrides the image tag (default is the chart appVersion)
    tag: "v1.8.3"
    # -- Image pull policy
    pullPolicy: IfNotPresent
```

* * *

### 1.4 Argo Rollouts Helm Chart 설치하기 :


```bash
# argocd가 설치된 상태에서 진행하는 경우 생성 필요없습니다.
$ kubectl create namespace argocd

# 설치 진행
$ helm upgrade --install argo-rollouts ./ -n argocd
```

* * *

- Helm Chart 설치 후, 파드 목록을 조회하여 정상 설치 되었는지 확인합니다.

```bash
# pod 목록 조회
$ kubectl get po -n argocd
```

![argo rollouts helm chart 배포 후 확인](/assets/img/post/helm/argocd%20image%20updater%20helm%20chart%20배포%20후%20확인.png)

* * *