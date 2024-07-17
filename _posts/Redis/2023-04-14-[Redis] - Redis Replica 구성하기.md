---
layout: post
title: "[Redis] - Redis Replica 구성하기"
date: 2023-04-14
categories: Redis
tags: [Redis, Replica, 복제]
image: /assets/post/redis-wallpaper.jpg
---

## Redis Replica 구성하고 싶었던 계기:
- Redis 설치는 할 줄 알지만 실제 서비스를 운영하고 있는 곳에서는 구성만이 아니라 동일한 프로세스를 복제하여 한곳에서 이슈가 발생하면 백업본으로 구성한 프로세스로 연결하도록 하는 방식을 알고 싶어서다.

* * *

## Redis 설치하기:
- Redis 설치해야하므로 경우 아래 링크 참조하면된다.
> * [Redis 설치방법](https://hwangyoonjae.github.io/redis/Redis-Redis-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0/ "Redis 설치방법")

* * *

## Redis Master 구성하기:
- Redis 설치한 경로에서 환경설정 파일을 연다.
```bash
$ vi /etc/redis/6379.conf
```

- 아래 내용을 찾아 주석해제하고 수정한다.
```bash
bind 0.0.0.0
requirepass "Redis에서 사용할 암호"
masterauth "Redis에서 사용할 암호"
repl-ping-slave-period 10
repl-timeout 60
```

- Redis 종료 시 암호를 요구하기에 스크립트 內 암호를 추가한다.
```bash
$ vi /etc/init.d/redis_6379
```
```bash
stop)
        if [ ! -f $PIDFILE ]
        then
            echo "$PIDFILE does not exist, process is not running"
        else
            PID=$(cat $PIDFILE)
            echo "Stopping ..."
            $CLIEXEC -p $REDISPORT -a mypassword shutdown  # shutdown 앞에 "-a [Redis 암호]"를 입력한다.
            while [ -x /proc/${PID} ]
            do
                echo "Waiting for Redis to shutdown ..."
                sleep 1
            done
            echo "Redis stopped"
        fi
        ;;
```

* * *

## Redis Slave 구성하기:
- Master와 동일하게 환경설정 파일 수정과 스크립트 內 암호를 추가한다.
- 추가로 Redis Slave 환경설정 파일에는 Redis Master의 IP:PORT를 입력한다.
```bash
replicaof "IP" "PORT"
```

* * *

## Redis replication 확인하기:
- Redis Master/Slave Redis Daemon 재시작한다.
```bash
# 재시작
$ /etc/init.d/redis_6379 stop
$ /etc/init.d/redis_6379 start
# 로그 확인
$ tail -f /var/log/redis_6379.log
```
[![Redis Master 재시작](/assets/images/Redis/Redis%20Master%20%EC%9E%AC%EC%8B%9C%EC%9E%91%20%EA%B2%B0%EA%B3%BC.PNG)](/assets/images/Redis/Redis%20Master%20%EC%9E%AC%EC%8B%9C%EC%9E%91%20%EA%B2%B0%EA%B3%BC.PNG)

* * *

## Redis replication 테스트하기:
- Redis Master에 데이터를 입력한다.
```bash
$ cd /redis-7.0.9/src
$ redis-cli -a "Redis에서 설정한 암호"
```
[![Redis Master 데이터 입력](/assets/images/Redis/Redis%20Master%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20%EC%9E%85%EB%A0%A5.PNG)](/assets/images/Redis/Redis%20Master%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20%EC%9E%85%EB%A0%A5.PNG)

* * *

- Redis Master에 입력한 데이터를 Redis Slave에서 확인한다.
```bash
$ cd /redis-7.0.9/src
$ redis-cli -a "Redis에서 설정한 암호"
```
[![Redis Slave 데이터 출력](/assets/images/Redis/Redis%20Slave%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20%EC%B6%9C%EB%A0%A5.PNG)](/assets/images/Redis/Redis%20Slave%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20%EC%B6%9C%EB%A0%A5.PNG)

* * *