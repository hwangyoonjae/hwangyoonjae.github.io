---
title: "[Redis] - Redis 설치하기"
categories:
  - Redis
tags:
  - [Redis, NoSQL]

toc: true
toc_sticky: true

date: 2022-09-01
last_modified_at: 2023-04-18
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
```bash
$ tar -zxvf redis-7.0.9.tar.gz
$ cd redis
$ make
$ cd src
$ make install
$ cd ..
$ ./install_server.sh
```
[![./install_server.sh 실행화면](/assets/images/DB/Redis%20install_server.sh%20%EC%8B%A4%ED%96%89%ED%99%94%EB%A9%B4.PNG)](/assets/images/DB/Redis%20install_server.sh%20%EC%8B%A4%ED%96%89%ED%99%94%EB%A9%B4.PNG)

* * *

### ./install_server.sh 오류:
- 아래 그림과 같은 경우
[![./install_server.sh 오류](/assets/images/DB/Redis%20install_server.sh%20%EC%98%A4%EB%A5%98.PNG)](/assets/images/DB/Redis%20install_server.sh%20%EC%98%A4%EB%A5%98.PNG)

- 다음과 같이 해당 내용에 "#"를 추가하여 주석처리한다.
[![./install_server.sh 오류 조치방법](/assets/images/DB/Redis%20install_server.sh%20%EC%98%A4%EB%A5%98%20%EC%A1%B0%EC%B9%98%EB%B0%A9%EB%B2%95.PNG)](/assets/images/DB/Redis%20install_server.sh%20%EC%98%A4%EB%A5%98%20%EC%A1%B0%EC%B9%98%EB%B0%A9%EB%B2%95.PNG)

* * *

### ss 명령어로 서버가 6379 포트 수신 중인지 확인하기:
```bash
$ ps -ef | grep redis
```
[![텍스트](/assets/images/DB/Redis%20%EC%8B%A4%ED%96%89%20%ED%99%95%EC%9D%B8.PNG)](/assets/images/DB/Redis%20%EC%8B%A4%ED%96%89%20%ED%99%95%EC%9D%B8.PNG)

* * *

### Redis 서버 확인하기:
```bash
$ redis-cli ping
```
[![텍스트](/assets/images/DB/Redis%20%EC%84%9C%EB%B2%84%20%ED%99%95%EC%9D%B8.PNG)](/assets/images/DB/Redis%20%EC%84%9C%EB%B2%84%20%ED%99%95%EC%9D%B8.PNG)

* * *