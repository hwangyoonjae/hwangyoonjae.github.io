---
layout: post
title: "[ArgoCD] - ArgoCD Carany 배포"
date: 2024-07-26
categories: ArgoCD
tags: [ArgoCD, Argo-Rollout, Carany]
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
kind: Rollout # Rollout으로 생성
metadata:
  name: canary-rollout
  namespace: argo-rollouts
spec:
  replicas: 8
  revisionHistoryLimit: 2 # 보관할 이전 버전의 최대 수
  selector:
    matchLabels:
      app: canary
  template:
    metadata:
      labels:
        app: canary
    spec:
      containers:
      - name: canary-rollouts-demo
        image: harbor.com/argo-rollout/particule/simplecolorapi:3.0
        imagePullPolicy: Always
        ports:
        - containerPort: 5000
  strategy:
    canary: # Carany 배포 전략을 사용할 것임을 지정
      maxSurge: "25%"    # canary 배포로 생성할 pod의 비율
      maxUnavailable: 0  # 업데이트 될 때 사용할 수 없는 pod의 최대 수
      steps:
      - setWeight: 25    # 카나리로 배포된 서버로 전송해야될 트래픽 비율
      - pause: {}        # AutoPromotion Time
```

* * *

## Service 생성하기 :
- 새 버전과 기존 버전의 애플리케이션 간의 트래픽을 효율적으로 라우팅하고 관리하기 위해 서비스를 생성한다.

```yaml
# service-active.yaml
kind: Service
apiVersion: v1
metadata:
  name: canary-service
  namespace: argo-rollouts
spec:
  selector:
    app: canary
  ports:
  - protocol: TCP
    port: 80
    targetPort: 5000
    nodePort: 30083
  type: NodePort
```

* * *

## Gitlab repo에 파일 커밋하기 :
- 생성한 파일들을 girlab repo의 커밋하면, webhook을 통해 파일 변경을 전달받은 argocd에서 자동으로 배포하는 것을 확인한다.
![argo rollout carany 배포 동작 확인](/assets/img/post/ArgoCD/argo%20rollout%20carany%20배포%20동작%20확인.png)

* * *

## Carany 배포한 Pod 및 Service 확인하기 :
- 서버에서 생성한 Pod와 Service 목록을 확인한다.

```bash
$ kubectl get pod -n argo-rollouts
$ kubectl get svc -n argo-rollouts
```
![argo rollout carany service 확인](/assets/img/post/ArgoCD/argo%20rollout%20carany%20service%20확인.png)

* * *

## 수동 승격 방법 :
- Argo Rollouts의 블루-그린 전략을 사용하여 autoPromotionEnabled를 false로 설정한 경우, 수동으로 새 버전의 애플리케이션을 활성화하는 과정으로 수동 승격을 수행하는 방법은 아래와 같다.

```bash
$ kubectl argo rollouts promote {rollout명} -n {namespace명}
```

![carany 수동 승격 후 화면](/assets/img/post/ArgoCD/carany%20수동%20승격%20후%20화면.png)

* * *