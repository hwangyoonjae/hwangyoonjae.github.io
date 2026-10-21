---
layout: post
title: "Helm Chart를 통해 Kasten 설치하기"
date: 2026-10-21
categories: [컨테이너, Helm]
tags: [Helm, Kasten]
image: /assets/img/post-title/helm-wallpaper.jpg
---


## 1. KASTEN 설치하기 :
### 1.1 KASTEN Helm Chart 다운로드 :

```bash
$ helm repo add kasten https://charts.kasten.io/
$ helm repo update
$ helm search repo kasten
```

![kasten helm chart 목록 조회](/assets/img/post/helm/kasten%20helm%20chart%20목록%20조회.png)

* * *

- 폐쇄망에서 진행하는 경우 아래와 같이 진행하면 됩니다.

```bash
# 압축 파일 다운로드
$ helm pull kasten/k10 --version 9.0.8

# 압축 파일 풀기
$ tar -zxvf k10-9.0.8.tgz
```

* * *

### 1.2 KASTEN Container Image 다운로드 : 

```bash
$ docker pull gcr.io/kasten-images/upgrade:9.0.8
$ docker pull gcr.io/kasten-images/executor:9.0.8
$ docker pull gcr.io/kasten-images/catalog:9.0.8
$ docker pull gcr.io/kasten-images/vbrintegrationapi:9.0.8
$ docker pull gcr.io/kasten-images/state:9.0.8
$ docker pull gcr.io/kasten-images/repositories:9.0.8
$ docker pull gcr.io/kasten-images/prometheus:9.0.8
$ docker pull gcr.io/kasten-images/metering:9.0.8
$ docker pull gcr.io/kasten-images/logging:9.0.8
$ docker pull gcr.io/kasten-images/kanister-tools:9.0.8
$ docker pull gcr.io/kasten-images/kanister:9.0.8
$ docker pull gcr.io/kasten-images/jobs:9.0.8
$ docker pull gcr.io/kasten-images/gateway:9.0.8
$ docker pull gcr.io/kasten-images/garbagecollector:9.0.8
$ docker pull gcr.io/kasten-images/frontend:9.0.8
$ docker pull gcr.io/kasten-images/events:9.0.8
$ docker pull gcr.io/kasten-images/dashboardbff:9.0.8
$ docker pull gcr.io/kasten-images/crypto:9.0.8
$ docker pull gcr.io/kasten-images/controllermanager:9.0.8
$ docker pull gcr.io/kasten-images/configmap-reload:9.0.8
$ docker pull gcr.io/kasten-images/bloblifecyclemanager:9.0.8
$ docker pull gcr.io/kasten-images/auth:9.0.8
$ docker pull gcr.io/kasten-images/aggregatedapis:9.0.8

$ docker tag gcr.io/kasten-images/upgrade:9.0.8               harbor.test.com/kasten-images/upgrade:9.0.8
$ docker tag gcr.io/kasten-images/executor:9.0.8              harbor.test.com/kasten-images/executor:9.0.8
$ docker tag gcr.io/kasten-images/catalog:9.0.8               harbor.test.com/kasten-images/catalog:9.0.8
$ docker tag gcr.io/kasten-images/vbrintegrationapi:9.0.8     harbor.test.com/kasten-images/vbrintegrationapi:9.0.8
$ docker tag gcr.io/kasten-images/state:9.0.8                 harbor.test.com/kasten-images/state:9.0.8
$ docker tag gcr.io/kasten-images/repositories:9.0.8          harbor.test.com/kasten-images/repositories:9.0.8
$ docker tag gcr.io/kasten-images/prometheus:9.0.8            harbor.test.com/kasten-images/prometheus:9.0.8
$ docker tag gcr.io/kasten-images/metering:9.0.8              harbor.test.com/kasten-images/metering:9.0.8
$ docker tag gcr.io/kasten-images/logging:9.0.8               harbor.test.com/kasten-images/logging:9.0.8
$ docker tag gcr.io/kasten-images/kanister-tools:9.0.8        harbor.test.com/kasten-images/kanister-tools:9.0.8
$ docker tag gcr.io/kasten-images/kanister:9.0.8              harbor.test.com/kasten-images/kanister:9.0.8
$ docker tag gcr.io/kasten-images/jobs:9.0.8                  harbor.test.com/kasten-images/jobs:9.0.8
$ docker tag gcr.io/kasten-images/gateway:9.0.8               harbor.test.com/kasten-images/gateway:9.0.8
$ docker tag gcr.io/kasten-images/garbagecollector:9.0.8      harbor.test.com/kasten-images/garbagecollector:9.0.8
$ docker tag gcr.io/kasten-images/frontend:9.0.8              harbor.test.com/kasten-images/frontend:9.0.8
$ docker tag gcr.io/kasten-images/events:9.0.8                harbor.test.com/kasten-images/events:9.0.8
$ docker tag gcr.io/kasten-images/dashboardbff:9.0.8          harbor.test.com/kasten-images/dashboardbff:9.0.8
$ docker tag gcr.io/kasten-images/crypto:9.0.8                harbor.test.com/kasten-images/crypto:9.0.8
$ docker tag gcr.io/kasten-images/controllermanager:9.0.8     harbor.test.com/kasten-images/controllermanager:9.0.8
$ docker tag gcr.io/kasten-images/configmap-reload:9.0.8      harbor.test.com/kasten-images/configmap-reload:9.0.8
$ docker tag gcr.io/kasten-images/bloblifecyclemanager:9.0.8  harbor.test.com/kasten-images/bloblifecyclemanager:9.0.8
$ docker tag gcr.io/kasten-images/auth:9.0.8                  harbor.test.com/kasten-images/auth:9.0.8
$ docker tag gcr.io/kasten-images/aggregatedapis:9.0.8        harbor.test.com/kasten-images/aggregatedapis:9.0.8

$ docker push harbor.test.com/kasten-images/upgrade:9.0.8
$ docker push harbor.test.com/kasten-images/executor:9.0.8
$ docker push harbor.test.com/kasten-images/catalog:9.0.8
$ docker push harbor.test.com/kasten-images/vbrintegrationapi:9.0.8
$ docker push harbor.test.com/kasten-images/state:9.0.8
$ docker push harbor.test.com/kasten-images/repositories:9.0.8
$ docker push harbor.test.com/kasten-images/prometheus:9.0.8
$ docker push harbor.test.com/kasten-images/metering:9.0.8
$ docker push harbor.test.com/kasten-images/logging:9.0.8
$ docker push harbor.test.com/kasten-images/kanister-tools:9.0.8
$ docker push harbor.test.com/kasten-images/kanister:9.0.8
$ docker push harbor.test.com/kasten-images/jobs:9.0.8
$ docker push harbor.test.com/kasten-images/gateway:9.0.8
$ docker push harbor.test.com/kasten-images/garbagecollector:9.0.8
$ docker push harbor.test.com/kasten-images/frontend:9.0.8
$ docker push harbor.test.com/kasten-images/events:9.0.8
$ docker push harbor.test.com/kasten-images/dashboardbff:9.0.8
$ docker push harbor.test.com/kasten-images/crypto:9.0.8
$ docker push harbor.test.com/kasten-images/controllermanager:9.0.8
$ docker push harbor.test.com/kasten-images/configmap-reload:9.0.8
$ docker push harbor.test.com/kasten-images/bloblifecyclemanager:9.0.8
$ docker push harbor.test.com/kasten-images/auth:9.0.8
$ docker push harbor.test.com/kasten-images/aggregatedapis:9.0.8
```

### 1.3 values.yaml 수정하기 :

- 위 과정에서 다운받은 컨테이너 이미지로 수정 및 배포 관련 설정사항을 수정합니다.

```yaml
# k10/values.yaml
global:
  # These are the default values for picking k10 images. They can be overridden
  # to specify a particular registy and tag.
  image:
    registry: harbor.test.com/kasten-images
    tag: ""
    pullPolicy: Always

```

* * *

### 1.4 KASTEN Helm Chart 설치하기 :

```bash
# 네임스페이스 생성
$ kubectl create namespace kasten-io

# kasten/base 설치 진행
$ helm upgrade --install k10 ./ -n kasten-io
```

![kasten Helm Chart 설치 완료 화면](/assets/img/post/helm/kasten%20Helm%20Chart%20설치%20완료%20화면.png)

* * *

- Helm Chart 설치 후, 파드 목록을 조회하여 정상 설치 되었는지 확인합니다.

```bash
# pod 목록 조회
$ kubectl get po -n kasten-io
```

![kasten helm chart 배포 후 확인](/assets/img/post/helm/kasten%20helm%20chart%20배포%20후%20확인.png)

* * *
