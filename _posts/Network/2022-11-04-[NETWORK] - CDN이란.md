---
layout: post
title: "[Network] - CDN이란"
date: 2022-11-04
categories: Network
tags: [CDN, GSLB]
image: /assets/post/network-wallpaper.jpg
---

## CDN(Content Delivery Network)이란?:
- Content Delivery Network의 약자로, 지리적 제약 없이 전 세계 사용자에게 빠르고 안전하게 콘텐츠를 전송할 수 있도록 하는 콘텐츠 전송 기술이다.

* * *

## CDN(Content Delivery Network)장점:
```html
- 웹사이트 로딩 속도 개선
- 인터넷 회선 비용 절감
- 컨텐츠 제공의 안정성
- 웹사이트 보안 개선
```

* * *

## CDN(Content Delivery Network)원리:
- 서버를 분산시켜 캐싱해두고 사용자의 컨텐츠 요청이 들어오면 사용자와 가장 가까운 위치에 존재하는 서버로 매핑시켜 요청된 콘텐츠의 캐싱된 내용을 내어주는 방식으로 빠르게 데이터를 전송할 수 있게한다.
[![텍스트](/assets/images/Network/CDN%20%EC%9B%90%EB%A6%AC.PNG)](/assets/images/Network/CDN%20%EC%9B%90%EB%A6%AC.PNG)

* * *

## CDN(Content Delivery Network) 필요기술:
### 로드밸런싱(Load Balance):
- 사용자에게 콘텐츠 전송 요청(Delivery Request)을 받았을 때, 최적의 네트워크 환경을 찾아 연결하는 기술로 GSLB(Global Server Load Balancing)이라고 한다.
- 물리적으로 가장 가깝거나 여유 트래픽이 남아 있는 곳으로 접속을 유도하는 기술이다.
- GSLB(Global Server Load Balancing)는 서버상태를 주기적으로 health check(건강 상태)를 수행한다.

### CDN의 트래픽을 감지:
- 통계자료를 고객에게 제공하고, 트래픽을 분산한다.

* * *

## CDN(Content Delivery Network) 캐싱 방식:
### Static Caching:
- Contents들을 운영자가 미리 Cache Server에 복사하기 때문에 사용자가 Cache Server에 Contents를 요청 시 무조건 Cache Server에서 확인 가능하다.

### Dynamic Caching:
- 사용자가 Contents 요청 시 Contents가 없을 때 Origin Server로 부터 다운로드 받아 전달한다.

* * *