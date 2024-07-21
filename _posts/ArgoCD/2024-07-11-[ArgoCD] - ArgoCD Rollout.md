---
layout: post
title: "[ArgoCD] - ArgoCD Rollout"
date: 2024-07-11
categories: ArgoCD
tags: [ArgoCD, Rollout]
image: /assets/img/post-title/argocd-wallpaper.jpg
---

## ArgoCD Rollout이란? :
- Argo Rollouts와 연동하여 점진적인 배포 전략을 관리하고 자동화하기 위해 사용한다.
- Argo Rollouts는 Kubernetes 애플리케이션의 점진적 배포를 위한 고급 기능을 제공하며, Canary, Blue-Green, Experiment 등의 배포 전략을 지원한다.
![argocd rollout 아키텍처](/assets/img/post/ArgoCD/argocd%20rollout%20아키텍처.png)

* * *

## ArgoCD Rollout의 필요성 및 장점 :
1. **점진적 배포**:
    - 애플리케이션을 단계적으로 배포함으로써 전체 시스템에 미치는 영향을 최소화한다.
    - 예를 들어, Canary 배포는 새로운 버전의 애플리케이션을 일부 트래픽에만 노출시켜 검증한 후 전체 트래픽으로 확장한다.

2. **자동화된 배포 관리**:
    - ArgoCD와 Argo Rollouts를 연동하여 CI/CD 파이프라인의 일환으로 자동화된 배포 전략을 구현할 수 있다.
    - GitOps 방식으로 모든 배포 설정을 Git 저장소에서 관리하고 추적할 수 있다.

3. **실시간 모니터링 및 롤백**:
    - 배포 과정에서 실시간 모니터링을 통해 문제가 발생하면 자동으로 롤백할 수 있다.
    - 배포 상태를 지속적으로 확인하고 필요시 수동 롤백도 가능하다.

4. **A/B 테스트 및 실험**:
    - 새로운 기능을 특정 사용자 그룹에게만 배포하여 테스트할 수 있다.
    - Experiment 전략을 통해 여러 버전을 동시에 배포하고 비교할 수 있다.

* * *

## ArgoCD와 Argo Rollouts 연동 예시 :
- Argo Rollouts CLI 설치

```bash
# Argo Rollouts CLI 바이너리 다운로드
$ curl -LO https://github.com/argoproj/argo-rollouts/releases/latest/download/kubectl-argo-rollouts-linux-amd64

# 실행 권한 부여 및 시스템 경로 추가
$ chmod +x ./kubectl-argo-rollouts-linux-amd64
$ mv ./kubectl-argo-rollouts-linux-amd64 /usr/local/bin/kubectl-argo-rollouts

$ kubectl argo rollouts version
# 위 명령어 실행 시 아래와 같이 출력
kubectl-argo-rollouts: v1.7.1+6a99ea9
  BuildDate: 2024-06-24T22:46:25Z
  GitCommit: 6a99ea9908e8f1e816ccd71e4c35adbbbbdd5f6c
  GitTreeState: clean
  GoVersion: go1.21.11
  Compiler: gc
  Platform: linux/amd64
```

- Argo Rollouts를 Kubernetes 클러스터에 설치

```bash
$ kubectl create namespace argo-rollouts
$ kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml
```

- Rollout 리소스 정의

```yaml
#rollout.yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: rollout-bluegreen
  namespace: argo-rollouts
spec:
  replicas: 2
  revisionHistoryLimit: 2
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
        image: harbor.com/argoproj/rollouts-demo:blue
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
  strategy:
    blueGreen:
      activeService: rollout-bluegreen-active
      previewService: rollout-bluegreen-preview
      autoPromotionEnabled: false
```

- service 정의

```yaml
# service.yaml
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
---
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

## Rollout 상태 확인하기 :
### CLI에서 Argo Rollout 상태 확인
```bash
$ kubectl argo rollouts get rollout rollouts-demo --watch
```
![argocd rolloust 상태 cli 확인](/assets/img/post/ArgoCD/argocd%20rolloust%20상태%20cli%20확인.png)

### Argo Rollouts 도구를 사용하여 예시로 제공되는 애플리케이션 확인
- 이 애플리케이션은 데모용으로 설계되어 있어서 Rollout 전략을 시연하고 설명하기 위해 간단하게 구성되어있다.
![argocd rolloust 상태 gui 확인](/assets/img/post/ArgoCD//argocd%20rolloust%20상태%20gui%20확인.png)
 
* * *