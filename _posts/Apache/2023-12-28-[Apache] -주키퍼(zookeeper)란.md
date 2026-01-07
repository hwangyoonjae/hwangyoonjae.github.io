---
layout: post
title: "주키퍼(zookeeper)란"
date: 2023-12-28
categories: [Server, Apache]
tags: [Apache, Zookeper, Kafka]
image: /assets/img/post-title/apache-zookeeper-wallpaper.jpg
---

## 1. zookeeper란? :
-  분산 코디네이션 서비스를 제공하는 오픈소스 프로젝트로 직접 어플리케이션 작업을 조율하는 것을 쉽게 개발할 수 있도록 도와주는 도구입니다.

* * *

### 1.1 분산 코디네이션 서비스란? :
- 분산 시스템에서 시스템 간의 정보 공유, 상태 체크, 서버들 간의 동기화를 위한 락 등을 처리해주는 서비스입니다.

* * *

## 2. zookeeper 구성하기 :
- 필자는 서버 3대를 준비하여 구성합니다.

* * *

### 2.1 zookeper 설치 파일 다운로드 :
- 아래 URL 접속하여 zookeeper 설치 파일을 다운 받는다.
> * [zookeper 설치 파일 다운로드](https://zookeeper.apache.org/releases.html "zookeper 설치 파일 다운로드")
- 다운이 완료되었으면 압축을 해제합니다.

```bash
$ cd /usr/local
$ wget http://apache.mirror.cdnetworks.com/zookeeper/stable/zookeeper-3.6.2.tar.gz
# 또는 
$ tar -zxvf apache-zookeeper-3.6.2-bin.tar.gz -C /usr/local
$ ln -s /usr/local/apache-zookeeper-3.6.2-bin/ /usr/local/zookeeper
```
[![zookeeper 심볼릭 링크 설정과 폴더 구성](/assets/img/post/Apache/zookeeper%20심볼릭%20링크%20설정과%20폴더%20구성.png)](/assets/img/post/Apache/zookeeper%20심볼릭%20링크%20설정과%20폴더%20구성.png)

* * *

## 3. zookeeper 설정하기 :
- 지노드의 복사본인 스냅샷과 트랜잭션 로그들이 저장될 별도의 디렉토리를 생성합니다.
- zk01서버는 myid를 1, zk02 서버는 myid를 2, zk03 서버는 myid를 3으로 만든다.

```bash
$ mkdir -p /data
$ echo 1 > /data/myid
```

* * *

- 주키퍼 환경설정 파일인 zoo.cfg 파일을 만든 후, 내용을 수정합니다.

```bash
$ vi /usr/local/zookeeper/conf/zoo.cfg
# zoo.cfg 파일 안에 내용
# 주키퍼가 사용하는 시간에 대한 단위(밀리초)
tickTime = 2000 
# 팔로워가 리더와 초기에 연결하는 시간에 대한 타임아웃 tick의 수
initLimit = 10
# 팔로워가 리더와 동기화 하는 시간에 대한 타임아웃 tick의 수
syncLimit = 5 
# 주키퍼의 트랜잭션 로그와 스냅샷이 저장되는 데이터 저장 경로
dataDir = /data 
# 주키퍼 사용 TCP 포트
clientPort = 2181
# 주키퍼 앙상블 구성을 위한 서버 설정이며, server.myid 형식으로 사용
server.1 = zk01:2888:3888
server.2 = zk02:2888:3888
server.3 = zk03:2888:3888
```
> ※ 간혹 주키퍼 기동 시 server.x에 명시된 자기 자신의 호스트명을 인식하지 못하는 경우가 있을 경우 자기 자신의 호스트명은 0.0.0.0으로 변경해주면 정상 동작합니다.
{: .prompt-warning}

* * *

## 4. zookeeper 실행하기 :
- 위 설정 완료 후, 아래 명령어를 통해서 주키퍼를 실행합니다.
```bash
# 주키퍼 서비스 시작
$ /usr/local/zookeeper/bin/zkServer.sh start
# 주키퍼 서비스 종료
$ /usr/local/zookeeper/bin/zkServer.sh stop
```

* * *

### 4.1 zookeeper 오류 :
- zookeeper 서비스 상태 확인 시 아래와 같이 오류가 발생하였다.

[![zookeeper 설치 오류](/assets/img/post/Apache/zookeeper 설치 오류.png)](/assets/img/post/Apache/zookeeper 설치 오류.png)

> 해결방법: zoo.cfg 파일 안에 데이터 디렉토리 경로 설정이 잘못되어 수정 후 조치했습니다.
{. prompt-danger}

* * *