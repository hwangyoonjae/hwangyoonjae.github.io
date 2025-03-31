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
- Pod 간 트래픽을 IPIP 터널을 통해 전달하여 Kubernetes 클러스터 내에서 간단한 네트워크 구성을 가능하게 한다.

![ipip 모드 통신 구조](/assets/img/post/kubernetes/ipip%20모드%20통신%20구조.png)

> Direct (BGP) 모드가 기본이 아닌 이유
> 
> Direct (BGP) 모드는 IP 터널 없이 노드 간 직접 라우팅을 사용하는 방식인데, 이를 사용하려면 클러스터 네트워크에 BGP 라우터가 필요하다.
> 대부분의 Kubernetes 환경에서는 BGP 설정 없이도 네트워크가 동작해야 하기 때문에 기본값이 IPIP 모드로 설정되어 있다.
{: .prompt-tip}

---

### Direct (BGP) 모드 :
- BGP를 활용하여 각 노드가 직접 라우팅 정보를 교환하며, 터널링 없이 최상의 네트워크 성능을 제공한다.

![direct 모드 통신구조](/assets/img/post/kubernetes/direct%20모드%20통신%20구조.png)

### BGP 연동 :
- Kubernetes 클러스터 내부 네트워크와 IDC 내부망 네트워크 간 직접 라우팅도 가능하다.

![direct 모드 bgp 연동](/assets/img/post/kubernetes/direct%20모드%20bgp%20연동.png)

> Direct모드에서 BGP를 활용하는 이유?
> 
> Pod 간의 네트워크 트래픽을 터널링(IPIP, VXLAN) 없이 직접 전달하는 방식이다. 이 방식을 사용하면 네트워크 성능이 향상되지만, 각 노드가 Pod 네트워크에 대한 라우팅 정보를 알아야 한다는 문제가 생긴다.
> 
> 이 문제를 해결하기 위해 BGP(Border Gateway Protocol) 를 활용하여 노드 간 라우팅 정보를 자동으로 교환하고, Pod 네트워크 간 최적의 경로를 설정한다.
{: .prompt-info}

---

### VXLAN 모드 :
- L2 오버레이 네트워크 기술인 VXLAN을 사용하여 Pod 간 통신을 수행하며, BGP 없이도 사용할 수 있다.

![vxlan 모드 통신구조](/assets/img/post/kubernetes/vlxan%20모드%20통신%20구조.png)
