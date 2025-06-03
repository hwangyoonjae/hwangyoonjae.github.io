---
layout: post
title: "Kubernetes Node Cordon & Drain"
date: 2024-07-31
categories: Kubernetes 
tags: [Kubernetes, Cordon, Drain]
image: /assets/img/post-title/kubernetes-wallpaper.jpg
---

## Cordon :
### Cordon이란? :
- 특정 노드를 비활성화하여 그 노드에 새로운 파드가 스케줄링되는 것을 막는 기능으로, 이미 그 노드에서 실행 중인 파드는 영향을 받지 않으며 계속 실행된다.

> 주로 노드를 유지 보수하거나 점검할 때 사용한다.
{: .prompt-tip}

* * *

### Cordon 설정하기 :
```bash
# cordon 적용
$ kubectl cordon {node name}

# cordon 해제
$ kubectl uncordon {node name}
```
![node schedulingdisabled](/assets/img/post/kubernetes/node%20schedulingdisabled.png)

* * *

## Drain :
### Drain이란? :
- 특정 노드에서 실행 중인 모든 파드를 안전하게 다른 노드로 이동시키며 파드를 강제로 종료하고, 해당 노드에서 파드가 실행되지 않도록 처리한다.

> 노드를 비우고 유지보수나 점검을 위해 파드를 안전하게 다른 노드로 이동시킬 때 사용한다.
{: .prompt-tip}

* * *

### Drain 설정하기 :
```bash
# 드레인 적용
$ kubectl drain {node name} --ignore-daemonsets --force
# or
$ kubectl drain {node name} --ignore-daemonsets --delete-emptydir-data

# 드레인 해제
$ kubectl uncordon {node name}
```
![node schedulingdisabled](/assets/img/post/kubernetes/node%20schedulingdisabled.png)

> Drain 옵션 설정
>
> \--ignore-daemonsets : DaemonSet으로 생성된 파드들을 무시하고 삭제하지 않도록 설정<br>
> \--force : 강제로 파드를 삭제하도록 설정<br>
> \--delete-emptydir-data : emptyDir 볼륨에 저장된 데이터를 삭제하도록 설정
{: .prompt-info}

* * *

## Cordon과 Drain의 차이점 :
- Cordon : 노드에 새로운 파드가 스케줄링되지 않도록 막는 역할을 하며, 이미 실행 중인 파드는 영향을 받지 않는다.
- Drain : 실행 중인 모든 파드를 다른 노드로 옮기고, 노드를 비우는 작업을 수행하며, 서비스 중단을 최소화하면서 노드를 비우는 데 사용한다.

|  |SchedulingDisabled|기존 Pod 삭제|
|:---:|:---:|:---:|
|cordon|O|X|
|drain|O|O|

* * *

## 질문사항 :

> 노드를 단순히 종료할 때는 이러한 과정을 하지 않는 이유는?
>
> Kubernetes의 내재된 기능과 동작 원리 때문이다.
{: .prompt-question}

- 위 질문에 대한 자세한 내용은 아래와 같다.

> Kubernetes의 내재된 기능
>
> **고가용성 및 복원력**: Kubernetes는 클러스터의 고가용성과 복원력을 보장하기 위해 설계되어있어 노드가 예상치 않게 종료되거나 장애가 발생할 경우, Kubernetes는 이를 감지하고 자동으로 파드를 다른 노드에 스케줄링하려고 한다.
> 
> **노드 상태 감시**: Kubernetes의 컨트롤 플레인 컴포넌트는 노드의 상태를 지속적으로 모니터링하며, 노드가 응답하지 않거나 연결이 끊어진 경우, Kubernetes는 해당 노드를 비정상(불량) 상태로 판단하고 거기에 스케줄링된 파드를 새로운 노드로 이동시킨다.
> 
> **ReplicaSet 및 Deployment**: 대부분의 워크로드는 ReplicaSet이나 Deployment를 통해 관리되고, 선언된 파드 수를 유지하기 위해 노드가 다운되었을 때 자동으로 새로운 파드를 다른 노드에 스케줄링한다.
{: .prompt-info}

* * *