---
layout: post
title: "Redis 데이터 처리하기"
date: 2023-05-17
categories: [DB, Redis]
tags: [Redis]
image: /assets/img/post-title/redis-wallpaper.jpg
---

## 1. Redis 데이터 입력, 수정, 삭제, 조회하기 :
### 1.1 Redis 데이터 입력(저장)하기 :
- 데이터를 저장할 때는 **set** 명령을 사용한다.

```bash
set key value
# 예시
ex) set data “hello”
```
![Redis 데이터 입력화면](/assets/img/post/Redis/Redis%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20%EC%9E%85%EB%A0%A5%20%ED%99%94%EB%A9%B4.PNG)

* * *

### 1.2 Redis 데이터 조회하기 :
- 데이터를 조회할 때는 **get** 명령을 사용한다.

```bash
get key
# 예시
ex) get data
```
![Redis 데이터 조회화면](/assets/img/post/Redis/Redis%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20%EC%A1%B0%ED%9A%8C%20%ED%99%94%EB%A9%B4.PNG)

* * *

### 1.3 Redis 데이터 변경하기 :
- 데이터를 변경할 때는 **rename** 명령을 사용한다.

```bash
rename key new_key
# 예시
ex) rename data data1
```
![Redis 데이터 변경화면](/assets/img/post/Redis/Redis%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20%EB%B3%80%EA%B2%BD%20%ED%99%94%EB%A9%B4.PNG)

* * *