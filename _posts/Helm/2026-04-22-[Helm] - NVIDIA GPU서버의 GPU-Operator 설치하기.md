---
layout: post
title: "NVIDIA GPU서버의 GPU-Operator 설치하기"
date: 2026-04-22
categories: [Helm]
tags: [Helm, GPU]
image: /assets/img/post-title/kubernetes-wallpaper.jpg
---

## 설치하기 :
### 1.1 gpu-operator Helm Chart 다운로드 :

```bash
$ helm repo add nvidia https://helm.ngc.nvidia.com/nvidia
$ helm repo update
$ helm search repo gpu-operator --versions | head
```

![gpu-operator helm chart 목록 조회](/assets/img/post/helm/gpu-operator%20helm%20등록.png)

* * *

- 폐쇄망에서 진행하는 경우 아래와 같이 진행하면 됩니다.

```bash
$ helm pull nvidia/gpu-operator --version 25.10.1 --destination .
```

* * *

### 1.2 gpu-operator Container Image 다운로드 : 

```bash
$ docker pull registry.k8s.io/nfd/node-feature-discovery:v0.18.2 
$ docker pull nvcr.io/nvidia/gpu-operator:v25.10.1
$ docker pull nvcr.io/nvidia/cuda:13.0.1-base-ubi9
$ docker pull nvcr.io/nvidia/cloud-native/k8s-driver-manager:v0.9.1
$ docker pull nvcr.io/nvidia/k8s/container-toolkit:v1.18.1
$ docker pull nvcr.io/nvidia/k8s-device-plugin:v0.18.1
$ docker pull nvcr.io/nvidia/cloud-native/dcgm:4.4.2-1-ubuntu22.04
$ docker pull nvcr.io/nvidia/k8s/dcgm-exporter:4.4.2-4.7.0-distroless
$ docker pull nvcr.io/nvidia/cloud-native/k8s-mig-manager:v0.13.1
$ docker pull nvcr.io/nvidia/cloud-native/vgpu-device-manager:v0.4.1
$ docker pull nvcr.io/nvidia/cloud-native/k8s-kata-manager:v0.2.3
$ docker pull nvcr.io/nvidia/kubevirt-gpu-device-plugin:v1.4.0
$ docker pull nvcr.io/nvidia/cloud-native/k8s-cc-manager:v0.1.1
$ docker pull nvcr.io/nvidia/k8s/cuda-sample:vectoradd-cuda12.5.0

$ docker tag registry.k8s.io/nfd/node-feature-discovery:v0.18.2      harbor.test.com/library/nfd/node-feature-discovery:v0.18.2
$ docker tag nvcr.io/nvidia/gpu-operator:v25.10.1                    harbor.test.com/library/nvidia/gpu-operator:v25.10.1
$ docker tag nvcr.io/nvidia/cuda:13.0.1-base-ubi9                    harbor.test.com/library/nvidia/cuda:13.0.1-base-ubi9
$ docker tag nvcr.io/nvidia/cloud-native/k8s-driver-manager:v0.9.1   harbor.test.com/library/nvidia/cloud-native/k8s-driver-manager:v0.9.1
$ docker tag nvcr.io/nvidia/k8s/container-toolkit:v1.18.1            harbor.test.com/library/nvidia/k8s/container-toolkit:v1.18.1
$ docker tag nvcr.io/nvidia/k8s-device-plugin:v0.18.1                harbor.test.com/library/nvidia/k8s-device-plugin:v0.18.1
$ docker tag nvcr.io/nvidia/cloud-native/dcgm:4.4.2-1-ubuntu22.04    harbor.test.com/library/nvidia/cloud-native/dcgm:4.4.2-1-ubuntu22.04
$ docker tag nvcr.io/nvidia/k8s/dcgm-exporter:4.4.2-4.7.0-distroless harbor.test.com/library/nvidia/k8s/dcgm-exporter:4.4.2-4.7.0-distroless
$ docker tag nvcr.io/nvidia/cloud-native/k8s-mig-manager:v0.13.1     harbor.test.com/library/nvidia/cloud-native/k8s-mig-manager:v0.13.1
$ docker tag nvcr.io/nvidia/cloud-native/vgpu-device-manager:v0.4.1  harbor.test.com/library/nvidia/cloud-native/vgpu-device-manager:v0.4.1
$ docker tag nvcr.io/nvidia/cloud-native/k8s-kata-manager:v0.2.3     harbor.test.com/library/nvidia/cloud-native/k8s-kata-manager:v0.2.3
$ docker tag nvcr.io/nvidia/kubevirt-gpu-device-plugin:v1.4.0        harbor.test.com/library/nvidia/kubevirt-gpu-device-plugin:v1.4.0
$ docker tag nvcr.io/nvidia/cloud-native/k8s-cc-manager:v0.1.1       harbor.test.com/library/nvidia/cloud-native/k8s-cc-manager:v0.1.1
$ docker tag nvcr.io/nvidia/k8s/cuda-sample:vectoradd-cuda12.5.0     harbor.test.com/library/nvidia/k8s/cuda-sample:vectoradd-cuda12.5.0

$ docker push harbor.test.com/library/nfd/node-feature-discovery:v0.18.2
$ docker push harbor.test.com/library/nvidia/gpu-operator:v25.10.1
$ docker push harbor.test.com/library/nvidia/cuda:13.0.1-base-ubi9
$ docker push harbor.test.com/library/nvidia/cloud-native/k8s-driver-manager:v0.9.1
$ docker push harbor.test.com/library/nvidia/k8s/container-toolkit:v1.18.1
$ docker push harbor.test.com/library/nvidia/k8s-device-plugin:v0.18.1
$ docker push harbor.test.com/library/nvidia/cloud-native/dcgm:4.4.2-1-ubuntu22.04
$ docker push harbor.test.com/library/nvidia/k8s/dcgm-exporter:4.4.2-4.7.0-distroless
$ docker push harbor.test.com/library/nvidia/cloud-native/k8s-mig-manager:v0.13.1
$ docker push harbor.test.com/library/nvidia/cloud-native/vgpu-device-manager:v0.4.1
$ docker push harbor.test.com/library/nvidia/cloud-native/k8s-kata-manager:v0.2.3
$ docker push harbor.test.com/library/nvidia/kubevirt-gpu-device-plugin:v1.4.0
$ docker push harbor.test.com/library/nvidia/cloud-native/k8s-cc-manager:v0.1.1
$ docker push harbor.test.com/library/nvidia/k8s/cuda-sample:vectoradd-cuda12.5.0
```

* * *

### 1.3 gpu-operator Helm Chart 설치하기 :

```bash
$ helm install gpu-operator ./ \
  -n gpu-operator \
  --create-namespace
```

![gpu-operator 배포](/assets/img/post/helm/gpu-operator%20배포.png)

> GPU Operator 설치 시 GPU 노드 Taint/Tolerations 설정 주의사항
>
> GPU Operator 설치 시 tolerations 설정을 사용하여 GPU 노드에 배포되므로, GPU 워커노드에는 이에 맞는 taint 설정을 함께 구성해야 한다.
>
> tolerations 설정 값은 아래와 같습니다.
{: .prompt-warning}

```yaml
tolerations:
  - key: "nvidia.com/gpu"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
```

* * *

- gpu-operator Helm Chart 배포 후, 파드 목록을 조회하여 정상 설치 되었는지 확인합니다.

```bash
# pod 목록 조회
$ kubectl get po -n gpu-operator
```

![gpu-operator helm chart 배포 완료](/assets/img/post/helm/gpu-operator%20파드%20목록.png)

* * *

## 2. gpu-operator를 이용한 파드 생성하기 :

- GPU 서버에 테스트용 파드를 생성하여 Kubernetes가 GPU를 정상적으로 인식하고 사용할 수 있는지 확인합니다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: cuda-vectoradd-test
spec:
  restartPolicy: Never
  containers:
    - image: harbor.test.com/nvidia/cuda:13.0.1-base-ubi9
      name: cuda
      command: ["/bin/bash", "-c", "nvidia-smi && sleep 3600"]
      resources:
        limits:
          nvidia.com/gpu: 1 # Kubernetes에 GPU 1개 사용을 요청하는 설정
  tolerations:
    - key: "node-type"
      value: "nvidia-gpu"
      effect: "NoSchedule"
      operator: "Equal"
```

![cuda를 이용한 pod 생성](/assets/img/post/helm/cuda를%20이용한%20pod%20생성.png)

* * *

