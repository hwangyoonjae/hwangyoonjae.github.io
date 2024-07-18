---
layout: post
title: "[Redis] - Redis란"
date: 2022-08-11
categories: Redis
tags: [DB, NoSQL]
image: /assets/img/post-title/redis-wallpaper.jpg
---


## Redis란? :
- **Remote Dictionary Server**의 약자로, Key-Value 구조의 비정형 데이터를 저장하고 관리하기 위한 오픈소스 기반의 비 관계형 데이터베이스 관리 시스템(DBMS)이다.
- 별도의 쿼리 없이 Key를 통해 빠르게 결과를 가져올 수 있고, 디스크에 데이터를 쓰는 구조가 아닌 메모리에서 데이터를 처리하기 때문에 작업 속도가 상당히 빠르다.

* * *

### Redis 구조
- 주로 Cache 서버를 구현할 때 많이 쓴다.
[![텍스트](/assets/img/post/Redis/Redis%20%EA%B5%AC%EC%A1%B0.PNG)](/assets/img/post/Redis/Redis%20%EA%B5%AC%EC%A1%B0.PNG)

<span style="color:#FA5858; font-size:12px">※ 캐시란? : 한번 읽어온 데이터를 임의의 공간에 저장하여 다음에 읽을 때는 빠르게 결과 값을 받을 수 있도록 도와주는 공간</span>
* * *

### Redis의 영속성
- Redis는 영속성을 보장하기 위해 데이터를 디스크에 저장할 수 있다.
- 서버가 내려가더라고 디스크에 저장된 데이터를 읽어서 메모리에 로딩한다.


#### [데이터를 디스크에 저장하는 방식]
- **RDB(Snapshotting) 방식**
  - 순간적으로 메모리에 있는 내용 전체를 디스크에 옮겨 담는 방식
- **AOF(Append On File) 방식**
  - Redis의 모든 write/update 연산 자체를 모두 log 파일에 기록하는 형태

* * *

### Redis 주의할점
- 인메모리 데이터 저장소의 특성상, 서버에 장애가 발생했을 경우 데이터 유실이 발생할 수 있기 때문에 서버 장애 시 운영에 대한 플랜이 필요하다.
- 메모리 관리가 중요하다.
- 싱글 스레드의 특성상, 한 번에 하나의 명령만 처리할 수 있어 처리하는데 시간이 오래 걸리는 요청, 명령은 피해야한다.

* * *