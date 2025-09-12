---
layout: post
title: "ArgoCD Blue-Green 배포"
date: 2024-07-25
categories: [DevOps, ArgoCD]
tags: [ArgoCD, Argo-Rollout, Blue-Green]
image: /assets/img/post-title/argocd-wallpaper.jpg
---

##  Argo rollout 설치하기 :
- Argo rollout 설치해야 하므로, 아래 링크를 통해서 설치 진행하면된다.
> * [Argo rollout 설치하기](https://hwangyoonjae.github.io/posts/ArgoCD-ArgoCD-Rollout/ "Argo rollout 설치하기")

* * *

## Blue-Green App 배포방법 :
### rollout 생성하기 :
> 아래 yaml 파일들은 Gitlab Repository에 저장합니다.
{: .prompt-warning}

```yaml
# rollout.yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: my-svc-blue-green
spec:
  replicas: 4
  revisionHistoryLimit: 2  # 보관할 이전 버전의 최대 수
  selector:
    matchLabels:
      app: my-svc-blue-green
  template:
    metadata:
      labels:
        app: my-svc-blue-green
    spec:
      containers:
        - name: app
          image: harbor.test.com/secloudit-util/custom-nginx   # v1
          ports:
            - containerPort: 80
  strategy:
    blueGreen:                                   # 블루-그린 배포 전략을 사용할 것임을 지정
      activeService: my-svc-blue-green           # 현재 서비스
      previewService: my-svc-blue-green-preview  # 새 버전 검증 서비스
      autoPromotionEnabled: false                # 수동 승격(권장: 테스트용)
      scaleDownDelaySeconds: 30                  # 서비스 전환 후 구버전 유지 시간
      scaleDownDelayRevisionLimit: 1             # 최근 1개 버전은 지연된 상태로 유지
```

* * *

### Service 생성하기 :
- BlueGreen 전략에는 activeService와 previewService라는 두 가지 서비스가 필요하여 두 설정 모두 Kubernetes 서비스 리소스를 참조한다.

```yaml
# svc-stable.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-svc-blue-green
  labels:
    app: my-svc-blue-green
spec:
  selector:
    app: my-svc-blue-green
  ports:
    - port: 80
      targetPort: 80
```

```yaml
# svc-preview.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-svc-blue-green-preview
  labels:
    app: my-svc-blue-green
spec:
  selector:
    app: my-svc-blue-green
  ports:
    - port: 80
      targetPort: 80
```

* * *

### Ingress 생성하기 : 

```yaml
# ingress-stable.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-svc-blue-green
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
    - host: my-svc-blue-green.test.com           # Stable 호스트
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-svc-blue-green
                port:
                  number: 80
```

```yaml
# ingress-preview.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-svc-blue-green-preview
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
    - host: my-svc-blue-green-preview.test.com   # Preview(새 버전 점검용)
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-svc-blue-green-preview
                port:
                  number: 80
```

> Blue-Green 배포에서 Ingress를 2개 생성하는 이유
>
> “현재 운영 트래픽(Active)” 과 “새 버전 검증(Preview)”을 동시에 다뤄야 하기 때문
{: .prompt-info}

* * *

### kustomization 생성하기 :

```yaml
# kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - rollout.yaml
  - svc-stable.yaml
  - svc-preview.yaml
  - ingress-stable.yaml
  - ingress-preview.yaml
images:
  - name: harbor.test.com/util/custom-nginx
    newTag: 1.0.20 # Nginx 버전
```

### Gitlab repo에 파일 커밋하기 :
- 생성한 파일들을 gitlab repo의 커밋하면, webhook을 통해 파일 변경을 전달받은 argocd에서 자동으로 배포하는 것을 확인한다.
![argo rollout blue-green 배포 동작 확인](/assets/img/post/ArgoCD/argo%20rollout%20blue-green%20배포%20동작%20확인.png)

* * *

### Blue-Green 배포한 Pod 및 Service 확인하기 :
- 서버에서 생성한 Pod와 Service 목록을 확인한다.

```bash
$ kubectl get pod -n argo-rollouts
$ kubectl get svc -n argo-rollouts
```
![argo rollout blue-green service 확인](/assets/img/post/ArgoCD/argo%20rollout%20service%20확인.png)

* * *

### Rollout 상태 확인하기 :

- 현재 생성한 Rollout 목록 조회한다.

```bash
$ kubectl -n [namespace] get rollout
```

![blue-green rollout 목록 조회](/assets/img/post/ArgoCD/blue-green%20rollout%20목록%20조회.png)

* * *

- Rollout 이름을 통해서 Blue-Green 배포 상태를 확인한다.

```bash
$ kubectl argo rollouts get rollout [rollout_name] -n [namespace]
```

![blue-green rollout 상태 확인](/assets/img/post/ArgoCD/blue-green%20rollout%20상태%20확인.png)

* * *

## 수동 승격 방법 :
- Argo Rollouts의 블루-그린 전략을 사용하여 autoPromotionEnabled를 false로 설정한 경우, 수동으로 새 버전의 애플리케이션을 활성화하는 과정으로 수동 승격을 수행하는 방법은 아래와 같다.

```bash
$ kubectl argo rollouts promote {rollout명} -n {namespace명}
```

![blue-green 수동 승격 후 화면](/assets/img/post/ArgoCD/blue-green%20수동%20승격%20후%20화면.png)

* * *

## 배포 확인하기 :
### 배포 매니페스트 환경별 패치 파일 생성하기 :

- 이 Kustomization은 base 리소스를 가져와 dev-int 네임스페이스에 배포하고, Ingress 설정만 환경에 맞게 수정하는 오버레이 설정한다.

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: dev-default
resources:
  - ../../base
patchesStrategicMerge:
  - ingress-patch.yaml
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-svc-blue-green
spec:
  rules:
    - host: web.dev-test.com # 프로젝트를 분리할 경우 각 폴더별 ingress 파일 생성하여 도메인 주소 변경 필요
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-svc-blue-green
                port:
                  number: 80
```

* * *

### 웹페이지 확인하기 :

- 아래 그림과 같이 배포되어 정상 접속 되는것을 확인한다.

![argo rollout blue-green 배포 후 웹 접속](/assets/img/post/ArgoCD/argo%20rollout%20blue-green%20배포%20후%20웹%20접속.png)

* * *