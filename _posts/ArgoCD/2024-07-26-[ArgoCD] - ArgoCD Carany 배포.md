---
layout: post
title: "ArgoCD Canary 배포"
date: 2024-07-26
categories: ArgoCD
tags: [ArgoCD, Argo-Rollout, Canary]
image: /assets/img/post-title/argocd-wallpaper.jpg
---

##  Argo rollout 설치하기 :
- Argo rollout 설치해야 하므로, 아래 링크를 통해서 설치 진행하면된다.
> * [Argo rollout 설치하기](https://hwangyoonjae.github.io/posts/ArgoCD-ArgoCD-Rollout/ "Argo rollout 설치하기")

* * *

## Canary App 배포방법 :
### rollout 생성하기:

```yaml
# rollout.yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: my-svc
spec:
  replicas: 4
  revisionHistoryLimit: 3
  progressDeadlineSeconds: 600
  strategy:
    canary:
      maxSurge: 25%
      maxUnavailable: 0
      stableService: my-svc
      canaryService: my-svc-canary
      trafficRouting:
        nginx:
          stableIngress: my-svc     # base/ingress.yaml 의 Ingress 이름과 동일
      steps:
        - setWeight: 10             # 새 버전이 10% 트랙픽 받음
        - pause: { duration: 120 }  # 2분 관찰 후 자동 진행
        - setWeight: 30             # 새 버전이 30% 트랙픽 받음
        - pause: { duration: 180 }  # 3분 관찰
        - setWeight: 60             # 새 버전이 60% 트랙픽 받음
        - pause: { duration: 300 }  # 5분 관찰
        # 마지막 스텝이 끝나면 자동으로 100%로 승격됨(별도 promote 필요 없음)
  selector:
    matchLabels:
      app: my-svc
  template:
    metadata:
      labels:
        app: my-svc
    spec:
      containers:
        - name: app
          # 태그는 overlays/kustomization.yaml의 images.newTag에서 관리(Image Updater write-back)
          image: harbor.test.com/util/custom-nginx
          ports:
            - name: http
              containerPort: 80
```

* * *

### Service 생성하기 :
- 새 버전과 기존 버전의 애플리케이션 간의 트래픽을 효율적으로 라우팅하고 관리하기 위해 서비스를 생성한다.

```yaml
# service-stable.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-svc
spec:
  selector:
    app: my-svc
  ports:
    - port: 80
      targetPort: http
```

```yaml
# service-canary.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-svc-canary
spec:
  selector:
    app: my-svc
  ports:
    - port: 80
      targetPort: http
```

> svc-stable: 운영 중인 안정 버전
> 
> svc-canary: 새 버전 테스트용
> 
> 둘로 나눠야 Argo Rollouts가 퍼센트 기반 트래픽 분할 + 모니터링 + 롤백을 제대로 할 수 있음
{: .prompt-info}

* * *

### Ingress 생성하기 : 

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-svc
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
    - host: placeholder.test.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-svc
                port:
                  number: 80
```

* * *

### kustomization 생성하기 :

```yaml
# kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - rollout.yaml
  - svc-stable.yaml
  - svc-canary.yaml
  - ingress.yaml
images:
  - name: harbor.test.com/util/custom-nginx
    newTag: 1.0.19 # Nginx 버전
```

> kustomization.yaml이란?
> 
> Kubernetes 리소스를 모아주고, 공통 설정을 적용하고, 환경별 Overlay까지 지원하는 배포 단위 정의서
{: .prompt-info}

* * *

### Gitlab repo에 파일 커밋하기 :
- 생성한 파일들을 gitlab repo의 커밋하면, webhook을 통해 파일 변경을 전달받은 argocd에서 자동으로 배포하는 것을 확인한다.
![argo rollout Canary 배포 동작 확인](/assets/img/post/ArgoCD/argo%20rollout%20carany%20배포%20동작%20확인.png)

* * *

### Canary 배포한 Pod 및 Service 확인하기 :
- 서버에서 생성한 Pod와 Service 목록을 확인한다.

```bash
$ kubectl get pod -n [namespace명]
$ kubectl get svc -n [namespace명]
```
![argo rollout Canary service 확인](/assets/img/post/ArgoCD/argo%20rollout%20carany%20service%20확인.png)

* * *

### Rollout 상태 확인하기 :

- 현재 생성한 Rollout 목록 조회한다.

```bash
$ kubectl -n [namespace] get rollout
```

![canary rollout 목록 조회](/assets/img/post/ArgoCD/canary%20rollout%20목록%20조회.png)

* * *

- Rollout 이름을 통해서 Blue-Green 배포 상태를 확인한다.

```bash
$ kubectl argo rollouts get rollout [rollout_name] -n [namespace]
```

![canary rollout 상태 확인](/assets/img/post/ArgoCD/canary%20rollout%20상태%20확인.png)

## 배포 확인하기 :
### 배포 매니페스트 환경별 패치 파일 생성하기 :

- 이 Kustomization은 base 리소스를 가져와 dev-int 네임스페이스에 배포하고, Ingress 설정만 환경에 맞게 수정하는 오버레이 설정한다.

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: dev-int
resources:
  - ../../base
patchesStrategicMerge:
  - ingress-patch.yaml
```

```yaml
# ingress-patch.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-svc
spec:
  rules:
    - host: web.dev-test.com # 프로젝트를 분리할 경우 각 폴더별 ingress 파일 생성하여 도메인 주소 변경 필요
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-svc
                port:
                  number: 80
```

* * *

### 웹페이지 확인하기 :

- 아래 그림과 같이 배포되어 정상 접속 되는것을 확인한다.

![argo rollout canary 배포 후 웹 접속](/assets/img/post/ArgoCD/argo%20rollout%20canary%20배포%20후%20웹%20접속.png)

* * *