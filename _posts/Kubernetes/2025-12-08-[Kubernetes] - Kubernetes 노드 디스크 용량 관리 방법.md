---
layout: post
title: "Kubernetes 노드 디스크 용량 관리 방법"
date: 2025-12-08
categories: [컨테이너, Kubernetes] 
tags: [Kubernetes, Keblet]
image: /assets/img/post-title/kubernetes-wallpaper.jpg
---

> 클러스터를 구성하여 운영하는 과정에서 서비스 배포 및 업데이트가 반복됨에 따라, 컨테이너 이미지 변경·재배포가 빈번하게 발생하고 있습니다.
>
> Kubernetes 환경에서는 이미지가 각 Worker Node 로 pull 되어 로컬 디스크에 저장되며,정책적으로 자동 정리가 되지 않을 경우 미사용(필요 없는) 이미지와 레이어가 지속적으로 누적될 수 있어 아래와 같은 설정 및 정리 방법을 진행합니다.
{: .prompt-info}

## 1. kubelet의 Image Garbage Collection(GC) 설정하기 :
### 1.1 kubelet Image Garbage Collection 정책 설명 :
- kubelet의 노드 디스크 사용률 기준으로 Pod에서 쓰지 않는 이미지를 자동으로 정리합니다.

| 옵션 | 의미 |
|:---:|:---:|
| `imageGCHighThresholdPercent` | 이 %를 넘으면 이미지 GC 시작 |
| `imageGCLowThresholdPercent` | 이 %까지 내려가면 GC 중단 |

* * *

## 1.2 kubelet 설정 파일 수정하기 :

```bash
# Worker Node에서 진행합니다.

$ vi /var/lib/kubelet/config.yaml
#==========아래 내용 추가==========
imageGCHighThresholdPercent: 85 # 노드 디스크 사용률 85% 초과 시 사용하지 않는 이미지 삭제
imageGCLowThresholdPercent: 80 # 노드 디스크 사용률 80% 될 때까지 반복
```

![kubelet의 Image Garbage Collection(GC) 설정](/assets/img/post/kubernetes/kubelet의%20Image%20Garbage%20Collection(GC)%20설정.png)

* * *

## 1.3 kubelet 설정 적용하기 :

```bash
$ systemctl restart kubelet
```

* * *