---
layout: post
title: "[Cubrid] - Cubrid 이중화 구성하기"
date: 2023-04-05
categories: Cubrid
tags: [DB, Cubrid, Master, Slave]
image: /assets/img/post-title/cubrid-wallpaper.jpg
---

## CUBRID Replication 실습하기:
- 고객사 이중화 구축한걸 바탕으로 작성하였다.

* * *

## CUBRID 설치 준비하기:
### CUBRID 설치파일 다운로드:
- 아래 URL 참고하여 CUBRID 설치파일을 다운받는다.
> * [CUBRID 다운받기](https://www.cubrid.com/downloads "CUBRID 다운받기")

* * *

## CUBRID 설치하기:
- FTP 통해서 CUBRID 설치파일을 서버에 복사 후 파일을 실행한다.
```bash
$ sh CUBRID-10.2-latest-Linux.x86_64.sh
```

- 실행 후 아래와 같이 문구 나오면 q를 누르고
[![텍스트](/assets/img/post/DB/CUBRID%20%EC%B4%88%EA%B8%B0%20%EC%84%A4%EC%B9%98%20%EC%8B%9C%20%ED%99%94%EB%A9%B4.PNG)](/assets/img/post/DB/CUBRID%20%EC%B4%88%EA%B8%B0%20%EC%84%A4%EC%B9%98%20%EC%8B%9C%20%ED%99%94%EB%A9%B4.PNG)

- 아래 그림과 같이 진행한다.
[![텍스트](/assets/img/post/DB/CUBRID%20%EC%84%A4%EC%B9%98%ED%99%94%EB%A9%B4.PNG)](/assets/img/post/DB/CUBRID%20%EC%84%A4%EC%B9%98%ED%99%94%EB%A9%B4.PNG)

* * *

## CUBRID 서비스 시작하기:
- CUBRID 설치한 계정 경로(/home/계정)으로 들어가 명령어를 입력하여 서비스 시작한다.
```bash
$ . .cubrid.sh
$ cubrid service start
```

* * *

## CUBRID Replication 구성하기:
### CUBRID HA란?:
- CUBRID의 HA(High Availability) 기능은 shared-nothing 구조이며, 액티브 서버(Active Server)에서 스탠바이 서버(Standby Server)로 데이터를 동기화하기 위해 다음 두 단계를 수행한다.
```
1. 트랜잭션 로그 다중화: 액티브 서버(Active Server)에서 생성되는 트랜잭션 로그를 실시간으로 다른 노드에 복제한다.
2. 트랜잭션 로그 반영: 실시간으로 복제된 트랜잭션 로그를 분석하여 스탠바이 서버에 데이터를 반영한다.
```
<span style="color:#FA5858; font-size:12px">※ HA란?: 하드웨어, 소프트웨어, 네트워크 등에 장애가 발생해도 지속적인 서비스를 제공하는 기능</span>

* * *

### 노드(Node)?:
- 노드(Node)는 CUBRID HA를 구성하는 논리적인 단위이다.

* * *

### 노드(Node) 구분:
- **마스터 노드(Master Node)** : 복제의 대상이 되는 노드로, 액티브 서버를 사용한 읽기, 쓰기 등 모든 서비스를 제공한다.
- **슬레이브 노드(Slave Node)** : 마스터 노드와 동일한 내용을 갖는 노드로, 마스터 노드의 변경이 자동으로 반영된다. 스탠바이 서버를 사용한 읽기 서비스를 제공하며 마스터 노드 장애 시 failover가 일어난다.
- **레플리카 노드(Replica Node)** : 마스터 노드와 동일한 내용을 갖는 노드로, 마스터 노드의 변경이 자동으로 반영된다. 스탠바이 서버를 사용한 읽기 서비스를 제공하며 마스터 노드 장애 시 failover가 일어나지 않는다.<br>
<span style="color:#FA5858; font-size:12px">※ failover란?: 마스터 노드에 장애가 발생하여 서비스를 제공할 수 없는 상태가 되면 우선순위가 가장 높은 슬레이브 노드가 자동으로 마스터 노드가 되는 것</span>

* * *

## CUBRID Master Server 설정하기:
### HOST명 변경하기:
```bash
$ vi /etc/hosts
```
[![텍스트](/assets/img/post/DB/CUBRID%20%EC%9D%B4%EC%A4%91%ED%99%94%20host%20%EC%84%A4%EC%A0%95%20.PNG)](/assets/img/post/DB/CUBRID%20%EC%9D%B4%EC%A4%91%ED%99%94%20host%20%EC%84%A4%EC%A0%95%20.PNG)

* * *

### 방화벽 열기:
```bash
$ firewall-cmd --permanent --add-port=7/tcp --zone=public
$ firewall-cmd --permanent --add-port=1523/tcp --zone=public
$ firewall-cmd --permanent --add-port=59901/tcp --zone=public
```

* * *

### 데이터베이스 생성:
```bash
$ cd CUBRID설치경로/databases
$ mkdir masterdb
$ cubrid createdb -F "masterdb" --log-volume-size=1024M masterdb ko_KR.utf8
```
[![텍스트](/assets/img/post/DB/CUBRID%20DB%20%EC%83%9D%EC%84%B1.PNG)](/assets/img/post/DB/CUBRID%20DB%20%EC%83%9D%EC%84%B1.PNG)

* * *

### cubrid.conf 내용 수정하기:
- 아래 그림과 같이 cubrid.conf 內 내용을 추가한다.
```bash
$ cd CUBRID설치경로/conf
$ cp cubrid.conf cubrid.conf.bak
$ vi cubrid.conf
```
[![텍스트](/assets/img/post/DB/CUBRID%20cubrid.conf%20%EC%88%98%EC%A0%95.PNG)](/assets/img/post/DB/CUBRID%20cubrid.conf%20%EC%88%98%EC%A0%95.PNG)

* * *

### cubrid_ha.conf 내용 수정하기:
- 아래 그림과 같이 주석처리(#) 되어있는 부분을 지운다.
```bash
$ cd CUBRID설치경로/conf
$ cp cp cubrid_ha.conf cubrid_ha.conf.bak
$ vi cubrid_ha.conf
```
[![텍스트](/assets/img/post/DB/CUBRID%20cubrid_ha.conf%20%EC%88%98%EC%A0%95.PNG)](/assets/img/post/DB/CUBRID%20cubrid_ha.conf%20%EC%88%98%EC%A0%95.PNG)

* * *

### databases.txt 내용 수정하기:
```bash
$ cd CUBRID설치경로/databases
$ vi databases.txt
```
[![텍스트](/assets/img/post/DB/CUBRID%20databases.txt%20%EC%88%98%EC%A0%95.PNG)](/assets/img/post/DB/CUBRID%20databases.txt%20%EC%88%98%EC%A0%95.PNG)

* * *

## CUBRID Slave Server 설정하기:
- CUBRID Master Server 구성한 방식 그대로 Slave 서버도 세팅한다.
- 단, HOSTNAME만 **nodeB**로 설정한다.

* * *

## CUBRID Replication 시작하기:
- CUBRID 설치한 계정 경로(/home/계정)으로 들어가 명령어를 입력하여 서비스 시작한다.
- Master 노드(nodeA)를 먼저 시작하고, Slave 노드(nodeB)를 시작한다.
```bash
$ . .cubrid.sh
$ cubrid heartbeat start
```

* * *

### CUBRID Replication 상태 확인하기:
```bash
$ cubrid heartbeat status
```
[![텍스트](/assets/img/post/DB/CUBRID%20%EC%9D%B4%EC%A4%91%ED%99%94%20%ED%99%95%EC%9D%B8.PNG)](/assets/img/post/DB/CUBRID%20%EC%9D%B4%EC%A4%91%ED%99%94%20%ED%99%95%EC%9D%B8.PNG)
<span style="color:#FA5858; font-size:12px">※ 구축화면과 위 그림의 나와있는 DB명, DB경로는 다를 수 있습니다.</span>

* * *