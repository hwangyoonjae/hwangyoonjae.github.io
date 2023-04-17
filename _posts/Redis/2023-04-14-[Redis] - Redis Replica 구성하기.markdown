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

- Redis 설치해야하는 경우 아래 링크 참조하면된다.
> * [Redis 설치방법](https://hwangyoonjae.github.io/redis/Redis-Redis-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0/ "Redis 설치하는 방법")

## Redis Master 구성하기:
- Redis 설치한 경로에서 환경설정 파일을 연다.
```bash
$ cd /etc/redis
$ vi 6379.conf
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
- Redis Master Redis Daemon 재시작한다.
```bash
# 재시작
$ /etc/init.d/redis_6379 stop
$ /etc/init.d/redis_6379 start

# 로그 확인
$ tail -f /var/log/redis_6379.log
```
[![Redis Master 재시작](/assets/images/DB/Redis%20Master%20%EC%9E%AC%EC%8B%9C%EC%9E%91%20%EA%B2%B0%EA%B3%BC.PNG)](/assets/images/DB/Redis%20Master%20%EC%9E%AC%EC%8B%9C%EC%9E%91%20%EA%B2%B0%EA%B3%BC.PNG)

- Redis Slave Redis Daemon 재시작한다.
```bash
# 재시작
$ /etc/init.d/redis_6380 stop
$ /etc/init.d/redis_6380 start

# 로그 확인
$ tail -f /var/log/redis_6380.log
```
[![Redis Slave 재시작](/assets/images/DB/Redis%20Slave%20%EC%9E%AC%EC%8B%9C%EC%9E%91%20%EA%B2%B0%EA%B3%BC.PNG)](/assets/images/DB/Redis%20Slave%20%EC%9E%AC%EC%8B%9C%EC%9E%91%20%EA%B2%B0%EA%B3%BC.PNG)