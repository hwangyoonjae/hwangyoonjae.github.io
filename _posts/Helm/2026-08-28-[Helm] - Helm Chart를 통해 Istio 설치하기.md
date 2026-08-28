---
layout: post
title: "Helm Chart를 통해 Istio 설치하기"
date: 2026-08-28
categories: [컨테이너, Helm]
tags: [Helm, Istio]
image: /assets/img/post-title/helm-wallpaper.jpg
---


## 1. Istio 설치하기 :
### 1.1 Istio Helm Chart 다운로드 :

```bash
$ helm repo add istio https://istio-release.storage.googleapis.com/charts
$ helm repo update
$ helm search repo istio
```

![Istio helm chart 목록 조회](/assets/img/post/helm/Istio%20helm%20chart%20목록%20조회.png)

* * *

- 폐쇄망에서 진행하는 경우 아래와 같이 진행하면 됩니다.

```bash
# 압축 파일 다운로드
$ helm pull istio/base --version 1.30.4
$ helm pull istio/istiod --version 1.30.4
$ helm pull istio/gateway --version 1.30.4

# 압축 파일 풀기
$ tar -zxvf base-1.30.4.tgz
$ tar -zxvf istiod-1.30.4.tgz
$ tar -zxvf gateway-1.30.4.tgz
```

> Istio는 역할별로 base, istiod, gateway Helm Chart가 분리되어 있어 각각 설치해야 합니다.
> 
> base는 CRD, istiod는 Control Plane, gateway는 외부 트래픽 진입점을 담당합니다.
{: :prompt-info}

* * *

### 1.2 Istio Container Image 다운로드 : 

```bash
$ docker pull registry.istio.io/release/pilot:1.30.4
$ docker pull registry.istio.io/release/proxyv2:1.30.4

$ docker tag registry.istio.io/release/pilot:1.30.4    harbor.test.com/istio/pilot:1.30.4
$ docker tag registry.istio.io/release/proxyv2:1.30.4  harbor.test.com/istio/proxyv2:1.30.4

$ docker push harbor.test.com/istio/pilot:1.30.4
$ docker push harbor.test.com/istio/proxyv2:1.30.4 
```

* * *

### 1.3 values.yaml 수정하기 :

- 위 과정에서 다운받은 컨테이너 이미지로 수정 및 배포 관련 설정사항을 수정합니다.

```yaml
# istiod/values.yaml
  global:
    hub: registry.istio.io/release
    # Default tag for Istio images.
    tag: 1.30.4
```

* * *

### 1.4 Istio Helm Chart 설치하기 :

```bash
# 네임스페이스 생성
$ kubectl create namespace istio-system

# istio/base 설치 진행
$ helm upgrade --install istio-base ./base -n istio-system

# istio/istiod 설치 진행
$ helm upgrade --install istiod ./istiod -n istio-system

# istio/gateway 설치 진행
$ helm upgrade --install istio-ingress ./gateway -n istio-system
```

![Istio Helm Chart 설치 완료 화면](/assets/img/post/helm/Istio%20Helm%20Chart%20설치%20완료%20화면.png)

* * *

- Helm Chart 설치 후, 파드 목록을 조회하여 정상 설치 되었는지 확인합니다.

```bash
# pod 목록 조회
$ kubectl get po -n istio-system
```

![Istio helm chart 배포 후 확인](/assets/img/post/helm/Istio%20helm%20chart%20배포%20후%20확인.png)

* * *