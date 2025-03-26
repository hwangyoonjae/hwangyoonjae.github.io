---
layout: post
title: "[Kubernetes] - Calico 통신 이해하기"
date: 2025-03-24
categories: Kubernetes
tags: [Kubernetes, Calico]
image: /assets/img/post-title/kubernetes-wallpaper.jpg
---

## Calico의 구조 :
- Calico CNI를 설치하면 Calico-node POD가 배포되고, Felix, BIRD, Confd라는 3가지 프로세스를 통해 Calico가 동작한다.

  ![Calico 통신 구조](/assets/img/post/kubernetes/Calico%20통신%20구조.png)

  - **Felix (필릭스)** : 인터페이스 관리, 라우팅 정보 관리, ACL 관리, 상태 체크
  - **BIRD (버드)**: BGP Peer 에 라우팅 정보 전파 및 수신, BGP RR(Route Reflector)
  - **Confd** : calico global 설정과 BGP 설정 변경 시(트리거) BIRD 에 적용해줌
  - **Datastore plugin** : calico 설정 정보를 저장하는 곳 - k8s API datastore(kdd) 혹은 etcd 중 선택
  - **Calico IPAM plugin** : 클러스터 내에서 파드에 할당할 IP 대역
  - **calico-kube-controllers** : calico 동작 관련 감시(watch)
  - **calicoctl** : calico 오브젝트를 CRUD 할 수 있다, 즉 datastore 접근 가능

  > Calico에서 노드간 라우팅 정보 공유는 어떤 통신을 통해 일어나는가?
  >
  > 각 노드간 통신을 하려면 각각의 라우팅 정보를 공유해야하기에 BGP 프로토콜을 사용한다.
  {: .prompt-question}

---

## Calico 네트워크 모드 :
- Kubernetes 클러스터에서 노드 간 Pod 네트워크를 연결하기 위해 여러 가지 네트워크 모드를 지원한다.

  |모드|설명|기본값|
  |------|------|------|
  |IP-in-IP (IPIP)|노드 간 통신 시 Pod IP를 IP-in-IP 터널로 캡슐화하여 전달|기본값|
  |VXLAN|IPIP 대신 VXLAN을 사용하여 오버레이 네트워크를 구성|IPIP 대신 설정 필요|
  |Direct (BGP)|BGP를 사용하여 노드 간 Pod IP를 직접 라우팅|IPIP 대신 설정 필요|

---

### IP-in-IP (IPIP) 모드 :
- 클러스터 외부 네트워크와의 충돌을 방지하고, Pod 간 통신을 안정적으로 유지하기 위해 사용된다.

![Calico-ipip 설정 확인](/assets/img/post/kubernetes/Calico-ipip%20설정%20확인.png)

> Direct (BGP) 모드가 기본이 아닌 이유
> 
> Direct (BGP) 모드는 IP 터널 없이 노드 간 직접 라우팅을 사용하는 방식인데, 이를 사용하려면 클러스터 네트워크에 BGP 라우터가 필요하다.
> 대부분의 Kubernetes 환경에서는 BGP 설정 없이도 네트워크가 동작해야 하기 때문에 기본값이 IPIP 모드로 설정되어 있다.
{: .prompt-tip}

---

