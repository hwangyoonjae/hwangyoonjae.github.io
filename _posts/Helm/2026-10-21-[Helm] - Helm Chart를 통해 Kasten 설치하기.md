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

---

## 2. Kasten 로그인 인증 설정하기 :
### 2.1 Kasten Token 인증 활성화 여부 확인하기 :

```bash
$ helm get values k10 -n kasten-io -a | grep -A 5 tokenAuth
```
![Kasten Token 인증 활성화 여부 확인](/assets/img/post/helm/Kasten%20Token%20인증%20활성화%20여부%20확인.png)

---

### 2.2 Kasten Token 인증 활성화하기 :

> 기본값은 false이므로 별도 설정을 하지 않았다면 비활성화 상태일 가능성이 높습니다. 
> 
> 이에 현재 설치된 Kasten Chart 버전을 그대로 사용해서 아래와 같이 변경합니다.
{: .prompt-warning}

```bash
# Token 인증 활성화
$ helm upgrade k10 ./ \
  -n kasten-io \
  --reuse-values \
  --set auth.tokenAuth.enabled=true \
  --set auth.secureCookies=false

# 변경 결과 확인
$ helm get values k10 -n kasten-io
```
![Kasten Token 활성화 화면](/assets/img/post/helm/Kasten%20Token%20활성화%20화면.png)

---

### 2.3 Kasten Token 생성하기 :

- Kubernetes에서 ServiceAccount를 생성하고 인증 토큰을 발급한 다음, 해당 토큰으로 로그인합니다.

```bash
# ServiceAccount 생성
$ kubectl create serviceaccount kasten-admin \
  -n kasten-io
```

- ServiceAccount에 관리자 권한을 부여합니다.

```bash
# 관리자 권한 부여
$ kubectl create clusterrolebinding kasten-admin-binding \
  --clusterrole=cluster-admin \
  --serviceaccount=kasten-io:kasten-admin

# 생성 확인
$ kubectl get sa kasten-admin -n kasten-io
$ kubectl get clusterrolebinding kasten-admin-binding
```

![Kasten 서비스 어카운트 생성 완료 화면](/assets/img/post/helm/Kasten%20서비스%20어카운트%20생성%20완료%20화면.png)

> cluster-admin은 클러스터 전체에 대한 최고 관리자 권한입니다. 
> 
> 초기 테스트용으로만 사용하고, 운영 환경에서는 Kasten 백업 관리에 필요한 권한만 부여한 계정을 사용하는 것이 안전합니다.
{: .prompt-warning}

---

### 2.4 Kasten Token 발급받기 :

```bash
$ kubectl create token kasten-admin \
  -n kasten-io \
  --duration=1h
```

![kasten token 생성화면](/assets/img/post/helm/kasten%20token%20생성화면.png)

---

## 3. Kasten 웹 대시보드 접속하기 :
### 3.1 Kasten Ingress 생성하기 :

- 기존에 설치한 Kasten K10의 gateway Service에 Ingress를 연결하면 웹 대시보드에 도메인으로 접속할 수 있습니다.

```bash
$ vi kasten-dashboard-ingress.yaml
```
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: kasten-dashboard
  namespace: kasten-io
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  ingressClassName: nginx

  tls:
    - hosts:
        - kasten.example.com
      secretName: kasten-tls

  rules:
    - host: kasten.example.com
      http:
        paths:
          - path: /k10
            pathType: Prefix
            backend:
              service:
                name: gateway
                port:
                  number: 80
```

> Ingress 생성 후 ***https://kasten.example.com/k10/#/*** 예시 주소처럼 실제 DNS 설정 후 접속할 수 있습니다.
{: .prompt-tip}

---

### 3.2 Kasten 대시보드 로그인하기 :

- ```2.4 Kasten Token 발급받기``` 과정에서 출력된 JWT 토큰 값을 로그인 화면에 입력합니다.

![Kasten Dashboard 로그인하기](/assets/img/post/helm/Kasten%20Dashboard%20로그인하기.png)

---

- 입력 후, 대시보드 초기 화면을 확인합니다.

![Kasten Dashboard 초기화면](/assets/img/post/helm/Kasten%20Dashboard%20초기화면.png)

---