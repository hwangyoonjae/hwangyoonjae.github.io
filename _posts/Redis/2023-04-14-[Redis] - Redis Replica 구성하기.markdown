---
title: "[Redis] - Redis Replica 구성하기"
categories:
  - Redis
tags:
  - [Redis, Replica, 복제]

toc: true
toc_sticky: true

date: 2023-04-14
last_modified_at: 2023-04-14
---

## Redis Replica 구성하고 싶었던 계기:
- Redis 설치는 할 줄 알지만 실제 서비스를 운영하고 있는 곳에서는 구성만이 아니라 동일한 프로세스를 복제하여 한곳에서 이슈가 발생하면 백업본으로 구성한 프로세스로 연결하도록 하는 방식을 알고 싶어서다.

* * *