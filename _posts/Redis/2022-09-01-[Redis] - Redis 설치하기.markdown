---
title: "[Redis] - Redis 설치하기"
categories:
  - Redis
tags:
  - [Redis, NoSQL]

toc: true
toc_sticky: true

date: 2022-09-01
last_modified_at: 2022-09-01
---

## Redis 설치준비:
### Redis 필요한 gcc-c++ 설치:
```bash
$ yum -y install gcc-c++
```

### Redis 패키지 설치하기:
- 아래 명령어와 같이 설치파일을 다운받는다.
```bash
$ wget http://download.redis.io/releases/redis-7.0.8.tar.gz
```
<span style="color:#FA5858; font-size:12px">※ 필자는 7.0.9버전을 사용했다.</span>

* * *

### 폐쇄망 설치:
- 폐쇄망 서버에 Redis를 설치하는 경우 아래 URL 접속하여 압축 파일을 다운받는다.
> * [Redis 다운로드](http://download.redis.io/releases/ "Redis 다운로드")

* * *

## Redis 설치하기:
```bash
$ tar -zxvf redis-7.0.8.tar.gz
$ cd redis
$ make
$ cd src
$ make install
$ cd ..
$ ./install_server.sh
```

### Redis 서비스 시작하기:
```bash
# redis 서버 시작
$ systemctl start redis

# redis 서버 자동 실행
$ systemctl enable redis

# redis 서버 동작 확인
$ systemctl status redis
```
[![텍스트](/assets/images/DB/Redis%20%EC%A0%95%EC%83%81%EC%8B%A4%ED%96%89%20%ED%99%95%EC%9D%B8.PNG)](/assets/images/DB/Redis%20%EC%A0%95%EC%83%81%EC%8B%A4%ED%96%89%20%ED%99%95%EC%9D%B8.PNG)

* * *

### Redis 원격 액세스 설정하기:
- 기본적으로 Redis는 원격 연결을 허용하지 않는다. Redis가 실행 중인 시스템인 127.0.0.1(로컬 호스트)에서만 Redis 서버에 연결할 수 있다.
```bash
$ vi /etc/redis.conf
```

- redis.conf 파일에서 bind 127.0.0.1로 시작하는 줄을 찾고 127.0.0.1 뒤에 서버 개인 IP 주소를 추가한다.
```
# IF YOU ARE SURE YOU WANT YOUR INSTANCE TO LISTEN TO ALL THE INTERFACES
# JUST COMMENT THE FOLLOWING LINE.
# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
bind 127.0.0.1 0.0.0.0
```
<span style="color:#FA5858; font-size:12px">※ 0.0.0.0 으로 설정해 모든 접근허용 설정이 가능하다.</span>

* * *

### Redis 변경 내용을 저장 후 서비스 재시작하기:
```bash
$ sudo systemctl restart redis
```

* * *

### ss 명령어로 서버가 6379 포트 수신 중인지 확인하기:
```bash
$ ss -an | grep 6379
```
[![텍스트](/assets/images/DB/Redis%206379%ED%8F%AC%ED%8A%B8%20%EC%88%98%EC%8B%A0%20%ED%99%95%EC%9D%B8.PNG)](/assets/images/DB/Redis%206379%ED%8F%AC%ED%8A%B8%20%EC%88%98%EC%8B%A0%20%ED%99%95%EC%9D%B8.PNG)

* * *

### Redis 서버 확인하기:
```bash
$ redis-cli ping
```
[![텍스트](/assets/images/DB/Redis%20%EC%84%9C%EB%B2%84%20%ED%99%95%EC%9D%B8.PNG)](/assets/images/DB/Redis%20%EC%84%9C%EB%B2%84%20%ED%99%95%EC%9D%B8.PNG)

```bash
# Redis 명령창 접속
$ redis-cli
```

* * *