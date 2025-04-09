---
layout: post
title: "[ArgoCD] - ArgoCD Blue-Green 배포"
date: 2024-07-25
categories: ArgoCD
tags: [ArgoCD, Argo-Rollout, Blue-Green]
image: /assets/img/post-title/argocd-wallpaper.jpg
---

##  Argo rollout 설치하기 :
- Argo rollout 설치해야 하므로, 아래 링크를 통해서 설치 진행하면된다.
> * [Argo rollout 설치하기](https://hwangyoonjae.github.io/posts/ArgoCD-ArgoCD-Rollout/ "Argo rollout 설치하기")

* * *

## rollout App 배포방법 :

```yaml
# rollout.yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: rollout-bluegreen
  namespace: argo-rollouts
spec:
  replicas: 2
  revisionHistoryLimit: 2 # 보관할 이전 버전의 최대 수
  selector:
    matchLabels:
      app: rollout-bluegreen
  template:
    metadata:
      labels:
        app: rollout-bluegreen
    spec:
      containers:
      - name: rollouts-demo
        image: harbor.com/argo-rollout/argoproj/rollouts-demo:blue
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
  strategy:
    blueGreen: # 블루-그린 배포 전략을 사용할 것임을 지정
      activeService: rollout-bluegreen-active # 현재 활성화된 애플리케이션 버전에 대한 트래픽을 라우팅하는 서비스
      previewService: rollout-bluegreen-preview # 새로운 애플리케이션 버전을 미리보기할 때 사용할 서비스
      autoPromotionEnabled: false
```

* * *

## Service 생성하기 :
- BlueGreen 전략에는 activeService와 previewService라는 두 가지 서비스가 필요하여 두 설정 모두 Kubernetes 서비스 리소스를 참조한다.

```yaml
# service-active.yaml
kind: Service
apiVersion: v1
metadata:
  name: rollout-bluegreen-active
  namespace: argo-rollouts
spec:
  type: NodePort
  selector:
    app: rollout-bluegreen
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
    nodePort: 30081
```

```yaml
# service-preview.yaml
kind: Service
apiVersion: v1
metadata:
  name: rollout-bluegreen-preview
  namespace: argo-rollouts
spec:
  type: NodePort
  selector:
    app: rollout-bluegreen
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
    nodePort: 30082
```

```bash
$ kubectl apply -f blue-green-service-active.yaml
$ kubectl apply -f blue-green-service-preview.yaml
```

* * *

## Gitlab repo에 파일 커밋하기 :
- 생성한 파일들을 girlab repo의 커밋하면, webhook을 통해 파일 변경을 전달받은 argocd에서 자동으로 배포하는 것을 확인한다.
![argo rollout blue-green 배포 동작 확인](/assets/img/post/ArgoCD/argo%20rollout%20blue-green%20배포%20동작%20확인.png)

* * *

## Blue-Green 배포한 Pod 및 Service 확인하기 :
- 서버에서 생성한 Pod와 Service 목록을 확인한다.

```bash
$ kubectl get pod -n argo-rollouts
$ kubectl get svc -n argo-rollouts
```
![argo rollout blue-green service 확인](/assets/img/post/ArgoCD/argo%20rollout%20service%20확인.png)

* * *

## 수동 승격 방법 :
- Argo Rollouts의 블루-그린 전략을 사용하여 autoPromotionEnabled를 false로 설정한 경우, 수동으로 새 버전의 애플리케이션을 활성화하는 과정으로 수동 승격을 수행하는 방법은 아래와 같다.

```bash
$ kubectl argo rollouts promote {rollout명} -n {namespace명}
```

![blue green 수동 승격 후 화면](/assets/img/post/ArgoCD/blue%20green%20수동%20승격%20후%20화면.png)

* * *