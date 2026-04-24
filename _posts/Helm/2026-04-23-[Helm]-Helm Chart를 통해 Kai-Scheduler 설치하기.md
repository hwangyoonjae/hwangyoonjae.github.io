---
layout: post
title: "Helm Chart를 통해 Kai-Scheduler 설치하기"
date: 2026-04-23
categories: [Helm]
tags: [Helm, GPU]
image: /assets/img/post-title/kubernetes-wallpaper.jpg
---

## 설치하기 :
### 1.1 kai-scheduler Helm Chart 다운로드 :

> KAI Scheduler는 Helm chart를 컨테이너 이미지와 동일한 방식으로 관리하고, 폐쇄망 및 DevOps 환경에서의 효율성과 보안성을 높이기 위해 OCI Registry 방식을 사용합니다.
{: .prompt-tip}

```bash
$ helm pull oci://ghcr.io/nvidia/kai-scheduler/kai-scheduler --version v0.12.10

$ tar -zxvf kai-scheduler-v0.12.10.tgz
```

* * *

### 1.2 kai-scheduler Container Image 다운로드 : 

```bash
$ docker pull ghcr.io/nvidia/kai-scheduler/operator:v0.12.10
$ docker pull ghcr.io/nvidia/kai-scheduler/podgrouper:v0.12.10
$ docker pull ghcr.io/nvidia/kai-scheduler/podgroupcontroller:v0.12.10
$ docker pull ghcr.io/nvidia/kai-scheduler/binder:v0.12.10
$ docker pull ghcr.io/nvidia/kai-scheduler/scheduler:v0.12.10
$ docker pull ghcr.io/nvidia/kai-scheduler/queuecontroller:v0.12.10
$ docker pull ghcr.io/nvidia/kai-scheduler/admission:v0.12.10
$ docker pull ghcr.io/nvidia/kai-scheduler/nodescaleadjuster:v0.12.10
$ docker pull ghcr.io/nvidia/kai-scheduler/crd-upgrader:v0.12.10 

$ docker tag ghcr.io/nvidia/kai-scheduler/operator:v0.12.10           harbor.test.com/library/nvidia/kai-scheduler/operator:v0.12.10
$ docker tag ghcr.io/nvidia/kai-scheduler/podgrouper:v0.12.10         harbor.test.com/library/nvidia/kai-scheduler/podgrouper:v0.12.10
$ docker tag ghcr.io/nvidia/kai-scheduler/podgroupcontroller:v0.12.10 harbor.test.com/library/nvidia/kai-scheduler/podgroupcontroller:v0.12.10
$ docker tag ghcr.io/nvidia/kai-scheduler/binder:v0.12.10             harbor.test.com/library/nvidia/kai-scheduler/binder:v0.12.10
$ docker tag ghcr.io/nvidia/kai-scheduler/scheduler:v0.12.10          harbor.test.com/library/nvidia/kai-scheduler/scheduler:v0.12.10
$ docker tag ghcr.io/nvidia/kai-scheduler/queuecontroller:v0.12.10    harbor.test.com/library/nvidia/kai-scheduler/queuecontroller:v0.12.10
$ docker tag ghcr.io/nvidia/kai-scheduler/admission:v0.12.10          harbor.test.com/library/nvidia/kai-scheduler/admission:v0.12.10
$ docker tag ghcr.io/nvidia/kai-scheduler/nodescaleadjuster:v0.12.10  harbor.test.com/library/nvidia/kai-scheduler/nodescaleadjuster:v0.12.10
$ docker tag ghcr.io/nvidia/kai-scheduler/crd-upgrader:v0.12.10       harbor.test.com/library/nvidia/kai-scheduler/crd-upgrader:v0.12.10

$ docker pugh harbor.test.com/library/nvidia/kai-scheduler/operator:v0.12.10
$ docker pugh harbor.test.com/library/nvidia/kai-scheduler/podgrouper:v0.12.10
$ docker pugh harbor.test.com/library/nvidia/kai-scheduler/podgroupcontroller:v0.12.10
$ docker pugh harbor.test.com/library/nvidia/kai-scheduler/binder:v0.12.10
$ docker pugh harbor.test.com/library/nvidia/kai-scheduler/scheduler:v0.12.10
$ docker pugh harbor.test.com/library/nvidia/kai-scheduler/queuecontroller:v0.12.10
$ docker pugh harbor.test.com/library/nvidia/kai-scheduler/admission:v0.12.10
$ docker pugh harbor.test.com/library/nvidia/kai-scheduler/nodescaleadjuster:v0.12.10
$ docker pugh harbor.test.com/library/nvidia/kai-scheduler/crd-upgrader:v0.12.10
```

* * *

### 1.3 values.yaml 수정하기 :

- 위 과정에서 다운받은 컨테이너 이미지로 수정 및 배포 관련 설정사항을 수정합니다.

```yaml
global:
  registry: harbor.test.com/library/nvidia/kai-scheduler # 수정
  tag: ""
  imagePullPolicy: IfNotPresent
  securityContext: {}
  imagePullSecrets: []
  leaderElection: false
  gpuSharing: false
  clusterAutoscaling: false
  nodeSelector: {}
  affinity: {}
  requireDefaultPodAntiAffinityTerm: false
  tolerations: []
  namespaceLabelSelector: {}
  podLabelSelector: {}
  resourceReservation:
    namespace: kai-resource-reservation
    serviceAccount: kai-resource-reservation
    appLabel: kai-resource-reservation
# 아래는 워커노드에 노드셀렉터 또는 taint 설정 시 추가하시면됩니다.
nodeSelector:
  node-type: "gpu-test"
tolerations:
  - key: "node-type"
    operator: "Equal"
    value: "gpu-test"
    effect: "NoSchedule"
```

![values.yaml 파일의 이미지 경로 및 taint 설정 값 설정](/assets/img/post/helm/values.yaml%20파일의%20이미지%20경로%20및%20taint%20설정%20값%20설정.png)

* * *

### 1.4 Kai-Scheduler Helm Chart 설치하기 :

```bash
$ helm install kai-scheduler ./ \
  -n kai-scheduler \
  --create-namespace
```

![kai-scheduler helm chart 배포](/assets/img/post/helm/kai-scheduler%20helm%20chart%20배포.png)

* * *

- Kai-Scheduler Helm Chart 배포 후, 파드 목록을 조회하여 정상 설치 되었는지 확인합니다.

```bash
# pod 목록 조회
$ kubectl get po -n kai-scheduler
```

[kai-scheduler helm chart 배포 완료](/assets/img/post/helm/kai-scheduler%20helm%20chart%20배포%20완료.png)

* * *

## 2. kai-scheular를 이용한 파드 생성하기 :

- KAI Scheduler를 사용하려면 Pod spec에 schedulerName: kai-scheduler를 추가한다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kai-gpu-test
spec:
  schedulerName: kai-scheduler
  restartPolicy: Never
  containers:
    - name: cuda
      image: harbor.test.com/library/nvidia/cuda:13.0.1-base-ubi9
      command: ["/bin/bash", "-c", "nvidia-smi && sleep 3600"]
      resources:
        limits:
          nvidia.com/gpu: 1
  tolerations:
    - key: "node-type"
      operator: "Equal"
      value: "nvidia-gpu"
      effect: "NoSchedule"
```

![kai-scheular를 이용한 파드 생성](/assets/img/post/helm/kai-scheular를%20이용한%20파드%20생성.png)

* * *

- Pod 생성 후, 상태를 확인합니다.

```bash
$ kubectl apply -f kai-gpu-test.yaml

$ kubectl get po -n default
```

![kai-scheular를 이용한 파드 확인](/assets/img/post/helm/kai-scheular를%20이용한%20파드%20확인.png)

* * *