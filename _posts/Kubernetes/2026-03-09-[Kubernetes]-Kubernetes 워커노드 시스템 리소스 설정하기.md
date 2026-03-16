---
layout: post
title: "Kubernetes 워커노드 시스템 리소스 설정하기"
date: 2026-03-09
categories: [Kubernetes]
tags: [Kubernetes, 리소스]
image: /assets/img/post-title/kubernetes-wallpaper.jpg
---

## 1. 워커노드의 시스템 리소스를 설정해야 하는 이유 :

> Kubernetes 워커노드는 파드뿐만 아니라 kubelet, container runtime, 네트워크 플러그인, 운영체제 프로세스 등 다양한 시스템 구성요소가 함께 동작합니다.
> 
> 만약 노드의 CPU와 메모리를 모두 파드가 사용하도록 두면 시스템 프로세스가 사용할 리소스가 부족해져 노드가 불안정해지거나 NotReady 상태가 될 수 있습니다.
>
> 따라서 kubeReserved와 systemReserved 설정을 통해 시스템이 사용할 리소스를 미리 예약하여 클러스터의 안정적인 운영을 보장해야 합니다.
>
{: .prompt-warning}

* * *

## 2. 워커노드의 시스템 리소스 설정 방법 :

- 워커노드 시스템 리소스는 kubelet 설정 파일에서 지정할 수 있습니다.

```bash
$ vi /var/lib/kubelet/config.yaml
```

```yaml
clusterDomain: cluster.local
# 아래 내용 추가
kubeReserved:
  cpu: "500m"
  memory: "1Gi"
  ephemeral-storage: "1Gi"
systemReserved:
  cpu: "500m"
  memory: "1Gi"
  ephemeral-storage: "1Gi"
# 위 내용 추가
containerRuntimeEndpoint: ""
```

* * *

- 설정 항목 설명은 아래와 같습니다.

| 설정 항목 | kubeReserved | systemReserved |
|---|---|---|
| 목적 | Kubernetes 구성요소가 사용할 리소스 예약 | 운영체제 및 시스템 프로세스 리소스 예약 |
| 대상 프로세스 | kubelet, container runtime, kube-proxy 등 | OS kernel, systemd, sshd 등 시스템 프로세스 |
| 주요 설정 항목 | cpu, memory, ephemeral-storage | cpu, memory, ephemeral-storage |
| 적용 효과 | Kubernetes 구성요소가 안정적으로 동작하도록 보장 | 운영체제가 사용할 최소 리소스를 확보 |
| 공통 특징 | 노드 전체 리소스에서 제외되어 Pod가 사용할 수 없는 리소스로 예약됨 | 노드 전체 리소스에서 제외되어 Pod가 사용할 수 없는 리소스로 예약됨 |

* * *


- kubelet 서비스를 재시작합니다.

```bash
$ systemctl restart kubelet
```

* * * 

## 3. 워커노드의 시스템 리소스 적용 확인하기 :

- 설정이 정상 적용되었는지 확인하는 방법은 다음과 같습니다.

```bash
# kubectl describe node <node-name>
kubectl describe node worker01
```

![워커노드의 시스템 리소스 적용 확인](/assets/img/post/kubernetes/워커노드의%20시스템%20리소스%20적용%20확인.png)

* * *