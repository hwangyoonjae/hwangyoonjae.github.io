---
layout: post
title: "Redis 설치하기"
date: 2022-09-01
categories: Redis
tags: [DB, NoSQL]
image: /assets/img/post-title/redis-wallpaper.jpg
---

## Redis 설치준비:
### Redis 필요한 gcc-c++ 설치:
- Redis Compile에 필요한 패키지를 다운받는다.
```bash
$ yum -y install gcc-c++
```

* * *

### Redis 패키지 설치하기:
- 아래 명령어와 같이 설치파일을 다운받는다.
```bash
$ wget http://download.redis.io/releases/redis-7.0.9.tar.gz
```
<span style="color:#FA5858; font-size:12px">※ 필자는 7.0.9버전을 사용했다.</span>

* * *

### 폐쇄망 설치:
- 폐쇄망 서버에 Redis를 설치하는 경우 아래 URL 접속하여 압축 파일을 다운받는다.
> * [Redis 다운로드](http://download.redis.io/releases/ "Redis 다운로드")

* * *

## Redis 설치하기:
- 다운받은 설치파일 압축 풀고 설치를 진행한다.
```bash
$ tar -zxvf redis-7.0.9.tar.gz
$ cd redis
$ make
$ cd src
$ make install
$ cd ..
$ ./install_server.sh
```
[![./install_server.sh 실행화면](/assets/img/post/Redis/Redis%20install_server.sh%20%EC%8B%A4%ED%96%89%ED%99%94%EB%A9%B4.PNG)](/assets/img/post/Redis/Redis%20install_server.sh%20%EC%8B%A4%ED%96%89%ED%99%94%EB%A9%B4.PNG)

* * *

### ./install_server.sh 오류:
- 아래 그림과 같은 경우
[![./install_server.sh 오류](/assets/img/post/Redis/Redis%20install_server.sh%20%EC%98%A4%EB%A5%98.PNG)](/assets/img/post/Redis/Redis%20install_server.sh%20%EC%98%A4%EB%A5%98.PNG)

- 다음과 같이 해당 내용에 "#"를 추가하여 주석처리한다.
[![./install_server.sh 오류 조치방법](/assets/img/post/Redis/Redis%20install_server.sh%20%EC%98%A4%EB%A5%98%20%EC%A1%B0%EC%B9%98%EB%B0%A9%EB%B2%95.PNG)](/assets/img/post/Redis/Redis%20install_server.sh%20%EC%98%A4%EB%A5%98%20%EC%A1%B0%EC%B9%98%EB%B0%A9%EB%B2%95.PNG)

* * *

### Redis 서비스 확인하기:
```bash
$ ps -ef | grep redis
```
[![텍스트](/assets/img/post/Redis/Redis%20%EC%8B%A4%ED%96%89%20%ED%99%95%EC%9D%B8.PNG)](/assets/img/post/Redis/Redis%20%EC%8B%A4%ED%96%89%20%ED%99%95%EC%9D%B8.PNG)

* * *

### Redis 서버 확인하기:
```bash
$ redis-cli ping
```
[![텍스트](/assets/img/post/Redis/Redis%20%EC%84%9C%EB%B2%84%20%ED%99%95%EC%9D%B8.PNG)](/assets/img/post/Redis/Redis%20%EC%84%9C%EB%B2%84%20%ED%99%95%EC%9D%B8.PNG)

* * *