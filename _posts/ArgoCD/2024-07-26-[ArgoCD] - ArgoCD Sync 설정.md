---
layout: post
title: "[ArgoCD] - ArgoCD Sync 설정"
date: 2024-07-26
categories: ArgoCD
tags: [ArgoCD, Sync]
image: /assets/img/post-title/argocd-wallpaper.jpg
---

## Sync Status :
- git과 쿠버네티스 현재 상태를 비교한 결과를 보여준다.
- 디폴트로 3분마다 또는 사용자가 Refresh을 수행하면 Sync Status가 업데이트된다.

> sync status는 **Synced**, **Out Of Sync** 2종류가 있다.
>
> Synced : git과 클러스터 현재 상태가 같은 경우
> Out Of Sync : git과 클러스터 현재 상태가 다른 경우
{: .prompt-info}

* * *

### Sync Status 확인방법 :
- ArgoCD 메인화면 > SYNC STATUS
![argocd sync status 확인방법](/assets/img/post/ArgoCD/argocd%20sync%20status%20확인방법.png)

* * *

## Sync와 Sync Policy :
### Sync :
- git에 있는 의도된 상태를 쿠버네티스 클러스터에 배포하는 작업으로 동기화라고 불리며, sync의 조건은 git과 쿠버네티스 현재 상태와 차이가 있어야 한다.

### Sync Policy :
- Auto sync와 Manual sync가 있는데 누가 sync을 수행하는가에 따라 수행하는 방법이 다르다.
- git과 쿠버네티스 현재 상태를 비교해서 차이가 있다면, Auto Sync는 argocd가 자동으로 sync 수행한다.
![sync policy 적용](/assets/img/post/ArgoCD/sync%20policy%20적용.png)

* * *

## Health Status :
### 개념 :
- git의 의도된 상태가 이상없이 클러스터에 동기화 되었을 때, 쿠버네티스 리소스가 이상없는지 쿠버네티스 상태를 보여준다.
- sync status의 상세정보를 보여준다.

### Healthy 상태 :
- 동기화 한 쿠버네티스 리소스가 이상없는 상태이다.
![healthy 상태](/assets/img/post/ArgoCD/healthy%20상태.png)

* * *

### Missing 상태 :
- 어떤 리소스가 git과 클러스터 상태가 불일치 한지 보여준다.
![missing 상태](/assets/img/post/ArgoCD/missing%20상태.png)

* * *

### Progressing 상태 :
- Sync작업(git의 의도된 상태를 클러스터로 동기화)이 수행 중이면 Progressing으로 표시한다.
![progressing 상태](/assets/img/post/ArgoCD/progressing%20상태.png)

* * *

### Degraded 상태 :
- Sync작업을 실패했다는 의미로, 쿠버네티스 설정 문제(Ex: serviceaccount 권한 없음, 노드에 스케쥴링 불가 등) 또는 네트워크 등 인프라 문제 등 때문에 동기화를 실패할 수 있다.

* * *

### Suspended 상태 :
- sync작업이 일시중지된 상태로 다른 작업이 끝나야 수행되거나 외부 이벤트가 필요한 경우 발생한다.
![suspended 상태](/assets/img/post/ArgoCD/suspended%20상태.png)

> Blue-Green 배포로 인한 Suspended 상태 발생
>
> argo-rollout 배포 기능 중 하나인 Blue-Green 배포 방식을 사용하는 경우 수동으로 승격을 하지 않는 이상 변경된 파드에 대해서는 배포를 하지 않아 위와 같은 상태로 될 가능성이 있다.
{: .prompt-tip }

* * *

## Prune :
### 개념 :
- 동기화된 리소스 삭제옵션으로 argocd를 통해 쿠버네티스 리소스를 동기화하고 git에서 리소스를 삭제할 때, 해당 리소스를 쿠버네티스에서 삭제할지 유지할지 결정하는 옵션이다.
- prune이 비활성화되어 있으면 argocd로 동기화한 쿠버네티스 리소스는, git에 리소스가 삭제되더라도 쿠버네티스에 삭제되지 않고 유지되고, prune이 활성화되면 git에 삭제되면 쿠버네티스 리소스도 삭제된다.

* * *