---
layout: post
title: "Helm Chart를 통해 Headlamp 설치하기"
date: 2026-05-27
categories: [컨테이너, Helm] 
tags: [Helm, Dashboard, Monitoring]
image: /assets/img/post-title/helm-wallpaper.jpg
---

## 1. Headlamp란? :
- Kubernetes 클러스터를 웹 브라우저에서 쉽게 관리할 수 있도록 제공하는 Kubernetes 전용 Web UI 대시보드입니다.

* * *

## 2. 설치하기 :
### 2.1 Headlamp Helm Chart 다운로드 :

```bash
$ helm repo add headlamp https://kubernetes-sigs.github.io/headlamp/
$ helm repo update
$ helm search repo headlamp --versions | head
```

![headlamp helm chart 목록 조회](/assets/img/post/helm/headlamp%20helm%20chart%20목록%20조회.png)

* * *

- 폐쇄망에서 진행하는 경우 아래와 같이 진행하면 됩니다.

```bash
$ helm pull headlamp/headlamp --version 0.42.0 --destination .
```

* * *

### 2.2 Headlamp Container Image 다운로드 : 

```bash
$ docker pull ghcr.io/headlamp-k8s/headlamp:v0.42.0

$ docker tag ghcr.io/headlamp-k8s/headlamp:v0.42.0 harbor.test.com/headlamp-k8s/headlamp:v0.42.0

$ docker push harbor.test.com/headlamp-k8s/headlamp:v0.42.0
```

* * *

### 2.3 values.yaml 수정하기 :

- 위 과정에서 다운받은 컨테이너 이미지로 수정 및 배포 관련 설정사항을 수정합니다.

```yaml
image:
  # -- Container image registry
  registry: harbor.test.com
  # -- Container image name
  repository: headlamp-k8s/headlamp
  # -- Image pull policy. One of Always, Never, IfNotPresent
  pullPolicy: IfNotPresent
  # -- Container image tag, If "" uses appVersion in Chart.yaml
  tag: "v0.42.0"
```

* * *

### 2.4 Headlamp Helm Chart 설치하기 :

```bash
# argocd가 설치된 상태에서 진행하는 경우 생성 필요없습니다.
$ kubectl create namespace headlamp-system

# 설치 진행
$ helm upgrade --install headlamp headlamp/headlamp -n headlamp-system
```

![Headlamp Helm Chart 설치 완료 화면](/assets/img/post/helm/Headlamp%20Helm%20Chart%20설치%20완료%20화면.png)

> Headlamp 설치 완료 후 Helm에서 안내하는 port-forward 방식은 로컬 테스트를 위한 임시 접속 방법입니다.
> 
> 본 구축 환경에서는 Ingress Controller를 통해 외부 접근을 구성할 예정이므로, port-forward 및 POD_NAME/CONTAINER_PORT 환경 변수 설정 작업은 수행하지 않고 Ingress 기반으로 서비스를 노출하였습니다.
{: .prompt-warning}

- 아래 YAML 내용과 같이 ingress를 생성합니다.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: headlamp
  namespace: headlamp-system
spec:
  ingressClassName: nginx
  rules:
    - host: headlamp.test.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: headlamp
                port:
                  number: 80
```

* * *

- ingress를 생성합니다.

```bash
$ kubectl apply -f ingress.yaml -n headlamp-system
```


* * *

- Helm Chart 설치 후, 파드 목록을 조회하여 정상 설치 되었는지 확인합니다.

```bash
# pod 목록 조회
$ kubectl get po -n headlamp-system
```

![headlamp helm chart 배포 후 확인](/assets/img/post/helm/velero%20helm%20chart%20배포%20후%20확인.png)

* * *

## 3. 대시보드 접속하기 :
### 3.1 인증 토큰 생성하기 :
- Headlamp 접속 후 로그인 화면에서 Kubernetes 인증 토큰 입력이 필요하며, 아래 명령어를 통해 사용할 ServiceAccount Token을 생성할 수 있습니다.

![headlamp 대시보드 화면](/assets/img/post/helm/headlamp%20대시보드%20화면.png)

```bash
$ kubectl create token headlamp --namespace headlamp-system
```

![headlamp serviceaccount 생성](/assets/img/post/helm/headlamp%20serviceaccount%20생성.png)

* * *

### 3.2 대시보드 인증하기 : 
- 위에서 작업한 Ingress 주소를 브라우저에 입력하여, 생성된 토큰을 복사하여 입력합니다.

![headlamp 로그인 후 화면](/assets/img/post/helm/headlamp%20로그인%20후%20화면.png)

* * *