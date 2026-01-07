---
layout: post
title: "ArgoCD Rollout이란"
date: 2024-07-11
categories: [DevOps, ArgoCD] 
tags: [ArgoCD, Argo-Rollout]
image: /assets/img/post-title/argocd-wallpaper.jpg
---

## 1. ArgoCD Rollout이란? :
- Argo Rollouts와 연동하여 점진적인 배포 전략을 관리하고 자동화하기 위해 사용합니다.
- Argo Rollouts는 Kubernetes 애플리케이션의 점진적 배포를 위한 고급 기능을 제공하며, Canary, Blue-Green, Experiment 등의 배포 전략을 지원합니다.
![argocd rollout 아키텍처](/assets/img/post/ArgoCD/argocd%20rollout%20아키텍처.png)

* * *

## 2. ArgoCD Rollout의 필요성 및 장점 :
1. **점진적 배포**:
    - 애플리케이션을 단계적으로 배포함으로써 전체 시스템에 미치는 영향을 최소화합니다.
    - 예를 들어, Canary 배포는 새로운 버전의 애플리케이션을 일부 트래픽에만 노출시켜 검증한 후 전체 트래픽으로 확장합니다.

2. **자동화된 배포 관리**:
    - ArgoCD와 Argo Rollouts를 연동하여 CI/CD 파이프라인의 일환으로 자동화된 배포 전략을 구현할 수 있습니다.
    - GitOps 방식으로 모든 배포 설정을 Git 저장소에서 관리하고 추적할 수 있습니다.

3. **실시간 모니터링 및 롤백**:
    - 배포 과정에서 실시간 모니터링을 통해 문제가 발생하면 자동으로 롤백할 수 있습니다.
    - 배포 상태를 지속적으로 확인하고 필요시 수동 롤백도 가능하다.

4. **A/B 테스트 및 실험**:
    - 새로운 기능을 특정 사용자 그룹에게만 배포하여 테스트할 수 있습니다.
    - Experiment 전략을 통해 여러 버전을 동시에 배포하고 비교할 수 있습니다.

* * *

## 3. ArgoCD와 Argo Rollouts 설치하기 :
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

* * *

## 4. Argo CD 배포 방식 :
### 4.1 롤링 업데이트 (Rolling Update) : 
- 새로운 버전의 애플리케이션을 점진적으로 배포합니다. 기존의 파드가 새 버전의 파드로 점진적으로 교체된다.
- 배포 중 다운타임이 없으며, 사용자는 점진적으로 새로운 버전을 접하게된다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app-container
        image: my-app-image:latest
        ports:
        - containerPort: 80
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
```

* * *

### 4.2 블루-그린 배포 (Blue-Green Deployment) :
- 두 개의 독립적인 환경(블루와 그린)을 사용하여 배포를 진행하고, 새 버전이 준비되면 그린 환경에 배포하여 준비가 완료되면 블루에서 그린으로 트래픽을 전환합니다.
- 배포 중 다운타임이 없고, 새로운 버전이 정상 작동하는지 확인 후 트래픽을 전환할 수 있습니다.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: my-app-rollout
spec:
  replicas: 3
  strategy:
    blueGreen:
      activeService: my-app-active
      previewService: my-app-preview
      autoPromotionEnabled: true
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app-container
        image: my-app-image:latest
        ports:
        - containerPort: 80
```

* * *

### 4.3 카나리 배포 (Canary Deployment) :
- 새로운 버전의 애플리케이션을 단계적으로 배포하여 처음에는 소수의 인스턴스에만 배포하고, 문제가 없으면 점진적으로 더 많은 인스턴스에 배포합니다.
- 새로운 버전의 안정성을 점진적으로 확인할 수 있습니다.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: my-app-rollout
spec:
  replicas: 3
  strategy:
    canary:
      steps:
      - setWeight: 20
      - pause: { duration: 10m }
      - setWeight: 50
      - pause: { duration: 10m }
      - setWeight: 100
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app-container
        image: my-app-image:latest
        ports:
        - containerPort: 80
```

> argo-rollout 배포 기능 사용 시 strategy의 **Blue-Green** 또는 **Canary** 중 선택하여 입력하면된다.
{: .prompt-info}

* * *