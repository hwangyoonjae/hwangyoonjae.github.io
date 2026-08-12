---
layout: post
title: "Helm Chart를 통해 KEDA 설치하기"
date: 2026-08-12
categories: [컨테이너, Helm]
tags: [Helm, Ingress, Gateway]
image: /assets/img/post-title/helm-wallpaper.jpg
---

## 1. KEDA란? :

- Kubernetes Event-Driven Autoscaling의 약자로, Kubernetes 워크로드를 CPU/Memory뿐 아니라 외부 이벤트나 메트릭을 기준으로 자동 확장/축소할 수 있게 해주는 오토스케일링 컴포넌트입니다.

* * *

## 2. KEDA 설치하기 :
### 2.1 KEDA Helm Chart 다운로드 :

```bash
$ helm repo add kedacore https://kedacore.github.io/charts
$ helm repo update
$ helm search repo kedacore/keda --versions | head
```

![KEDA helm chart 목록 조회](/assets/img/post/helm/KEDA%20helm%20chart%20목록%20조회.png)

* * *

- 폐쇄망에서 진행하는 경우 아래와 같이 진행하면 됩니다.

```bash
# 압축 파일 다운로드
$ helm pull kedacore/keda \
  --version 2.20.2

# 압축 파일 풀기
$ tar -zxvf keda-2.20.2.tgz
```

* * *

### 2.2 KEDA Container Image 다운로드 : 

```bash
$ docker pull ghcr.io/kedacore/keda:2.20.2
$ docker pull ghcr.io/kedacore/keda-metrics-apiserver:2.20.2
$ docker pull ghcr.io/kedacore/keda-admission-webhooks:2.20.2

$ docker tag ghcr.io/kedacore/keda:2.20.2                     harbor.test.com/kedacore/keda:2.20.2
$ docker tag ghcr.io/kedacore/keda-metrics-apiserver:2.20.2   harbor.test.com/kedacore/keda-metrics-apiserver:2.20.2
$ docker tag ghcr.io/kedacore/keda-admission-webhooks:2.20.2  harbor.test.com/kedacore/keda-admission-webhooks:2.20.2

$ docker push harbor.test.com/kedacore/keda:2.20.2
$ docker push harbor.test.com/kedacore/keda-metrics-apiserver:2.20.2
$ docker push harbor.test.com/kedacore/keda-admission-webhooks:2.20.2
```

* * *

### 2.3 values.yaml 수정하기 :

- 위 과정에서 다운받은 컨테이너 이미지로 수정 및 배포 관련 설정사항을 수정합니다.

```yaml
image:
  keda:
    registry: harbor.test.com
    repository: kedacore/keda
    tag: ""
  metricsApiServer:
    registry: harbor.test.com
    repository: kedacore/keda-metrics-apiserver
    tag: ""
  webhooks:
    registry: harbor.test.com
    repository: kedacore/keda-admission-webhooks
    tag: ""
  pullPolicy: IfNotPresent
```

* * *

### 2.4 KEDA Helm Chart 설치하기 :

```bash
# 네임스페이스 생성
$ kubectl create namespace keda

# 설치 진행
$ helm upgrade --install keda . -n keda
```

![KEDA Helm Chart 설치 완료 화면](/assets/img/post/helm/KEDA%20Helm%20Chart%20설치%20완료%20화면.png)

* * *

- Helm Chart 설치 후, 파드 목록을 조회하여 정상 설치 되었는지 확인합니다.

```bash
# pod 목록 조회
$ kubectl get po -n keda
```

![KEDA helm chart 배포 후 확인](/assets/img/post/helm/KEDA%20helm%20chart%20배포%20후%20확인.png)

* * *

- ScaledObject, ScaledJob 같은 CRD를 사용하기 때문에 정상 설치 되었는지 확인합니다.

```bash
$ kubectl get crd | grep keda
```

![KEDA CRD 설치 확인](/assets/img/post/helm/KEDA%20CRD%20설치%20확인.png)

* * *

- KEDA Metrics Server는 Kubernetes HPA에 External Metrics API를 제공하기 때문에 정상 설치 되었는지 확인합니다.

```bash
$ kubectl get apiservice | grep external.metrics
```

![KEDA External Metrics API 확인](/assets/img/post/helm/KEDA%20External%20Metrics%20API%20확인.png)

* * *

- External Metrics API 직접 호출하여 JSON이 반환되는지 확인합니다.

```bash
$ kubectl get --raw "/apis/external.metrics.k8s.io/v1beta1"
```

![External Metrics API 직접 호출 확인](/assets/img/post/helm/External%20Metrics%20API%20직접%20호출%20확인.png)

* * *

## 3. KEDA ScaledObject 구성하기 :

> ScaledObject를 별도로 생성해야 KEDA가 “어떤 Deployment를, 어떤 이벤트 기준으로, 몇개까지” 스케일링할지 알 수 있습니다.
{: .prompt-info}

### 3.1 테스트 Namespace 및 Deployment 생성하기 :

- 테스트를 위해 네임스페이스를 생성합니다.

```bash
$ kubectl create namespace keda-demo
```

* * *

- 테스트를 위해 Deployment를 생성합니다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: keda-test
  namespace: keda-demo
spec:
  replicas: 0
  selector:
    matchLabels:
      app: keda-test
  template:
    metadata:
      labels:
        app: keda-test
    spec:
      containers:
        - name: nginx
          image: harbor.test.com/library/nginx:1.25.3
          ports:
            - containerPort: 80
```
```bash
# 생성
$ kubectl apply -f deployment.yaml
```

* * * 

### 3.2 ScaledObject 생성하기 :

- ScaledObject YAML 파일을 작성하여 생성합니다.

```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: keda-test
  namespace: keda-demo
spec:
  scaleTargetRef:
    name: keda-test      # 스케일링할 상위 워크로드 이름(보통 Deployment)

  pollingInterval: 5     # Trigger 상태 확인 주기(초)
  cooldownPeriod: 10     # Trigger 비활성 후 0개까지 줄이기 전 대기 시간(초)

  minReplicaCount: 0     # 최소 Pod 수, 0이면 Scale to Zero 가능
  maxReplicaCount: 5     # 최대 Pod 수

  triggers:
    - type: cron                    # 시간 기반 Cron Scaler 사용
      metadata:
        timezone: Asia/Seoul        # Cron 기준 시간대
        start: "0 0 * * *"          # 매일 00:00부터 활성화
        end: "59 23 * * *"          # 매일 23:59에 비활성화
        desiredReplicas: "3"        # 활성 시간 동안 유지할 Replica 수
```
```bash
# 생성
$ kubectl apply -f scaledobject.yaml
```

* * *

- 생성 후, 확인합니다.

```bash
$ kubectl get scaledobject -n keda-demo
```

![KEDA ScaledObject 확인](/assets/img/post/helm/KEDA%20ScaledObject%20확인.png)

* * *

### 3.3 KEDA가 HPA를 만들었는지 확인하기 :

- ScaledObject를 생성하면 KEDA가 HPA를 자동으로 생성합니다.

![KEDA HPA 생성 확인](/assets/img/post/helm/KEDA%20HPA%20생성%20확인.png)

* * *

- 실제 Pod가 증가하는지 확인합니다.

```bash
$ watch -n 1 kubectl get deploy,pod,hpa,scaledobject -n keda-demo
```

![KEDA Pod 증가 확인](/assets/img/post/helm/KEDA%20Pod%20증가%20확인.png)

* * *

## 4. Scale To Zero 테스트하기 :

- Cron 시간을 현재 활성화되지 않는 범위로 변경하여 테스트합니다.

```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: keda-test
  namespace: keda-demo
spec:
  scaleTargetRef:
    name: keda-test

  pollingInterval: 5
  cooldownPeriod: 10

  minReplicaCount: 0
  maxReplicaCount: 5

  # 아래 내용으로 변경
  triggers:
    - type: cron
      metadata:
        timezone: Asia/Seoul
        start: "0 0 1 1 *"
        end: "5 0 1 1 *"
        desiredReplicas: "3"
```
```bash
$ kubectl apply -f scaledobject.yaml
```

* * *

- 실제 Pod가 감소하는지 확인합니다.

```bash
$ watch -n 1 kubectl get deploy,pod,hpa,scaledobject -n keda-demo
```

![KEDA Pod 감소 확인](/assets/img/post/helm/KEDA%20Pod%20감소%20확인.png)

> ACTIVE=False는 현재 Cron Trigger가 비활성 상태라는 뜻입니다. 
> 
> 지금 설정이 비활성 시간대라면 KEDA가 트리거가 없다고 판단해서 minReplicaCount: 0까지 내려간 상태이며, KEDA 공식 문서도 Cron scaler에서 비활성 시간에 minReplicaCount: 0이면 Deployment를 0개까지 줄일 수 있다고 설명합니다.
{: .prompt-tip}

* * *