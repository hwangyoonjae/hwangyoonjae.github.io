---
layout: post
title: "[MariaDB] - MariaDB 이중화 구성하기"
date: 2022-10-28
categories: MariaDB
tags: [DB 이중화, Master, Slave]
image: /assets/img/post-title/mariadb-wallpaper.jpg
---

## MariaDB Replication 절차:
- MariaDB에서 기본적으로 제공하는 Replication 기능으로 Master-Slave 구조로 되어있다.
- Master 서버의 Binary로그를 Slave서버가 Relay로그에 저장해서 복제하는 방식이다.
- MasterDB에 이벤트가 발생하면 SlaveDB와 복제를 위해 생성한 Binaary log에 DB업데이트와 동시에 기록한다.
- SlaveDB는 자신이 MasterDB의 몇 번째 위치의 데이터를 마지막으로 가져왔는지 기록했다가 MasterDB의 Binaary Log에 새로운 기록이 업데이트 되면 가져오고 가져온 위치를 기억한 후 SlaveDB는 전달받은 Binaary log를 Relay Logdp 기록하여 순차적으로 DB에 저장한다.
[![텍스트](/assets/img/post/DB/MariaDB%20Replication.PNG)](/assets/img/post/DB/MariaDB%20Replication.PNG)

* * *

## MariaDB Replication 실습하기:
- Maria DB가 설치된 후 진행되는 실습이다.

* * *

## Master Server 구성:
### Master Server 설정하기:
- 아래와 같은 내용을 추가한다.

```bash
$ vi /etc/my.cnf.d/server.cnf

[mysqld]
# binary log의 파일명
log-bin = mysql-bin
# master와 slave의 값 구별
server-id = 1
# STATEMENT, ROW, MIXED 중 선택히여 포맷 지정
binlog_format = row
# 보관기간 설정
expire_logs_days = 2
```

* * *

### Master Server MariaDB 재시작 및 접속하기:
```bash
# MariaDB 재시작
$ service mariadb restart
또는
$ systemctl restart mariadb
```

```sql
> mysql -u root -p
```

* * *

### Slave Server 접속할 계정 생성하기:
```sql
> grant replication slave on *.* 'user'@'%' identified by 'password';
```

* * *

### Master Server binay log 확인하기:
```sql
> show master status;
```
[![텍스트](/assets/img/post/DB/master%20%EC%84%9C%EB%B2%84%20%EC%83%81%ED%83%9C%20%ED%99%95%EC%9D%B8.PNG)](/assets/img/post/DB/master%20%EC%84%9C%EB%B2%84%20%EC%83%81%ED%83%9C%20%ED%99%95%EC%9D%B8.PNG)

* * *

## Slave Server 구성:
### Slave Server 설정하기:
- 아래와 같은 내용을 추가한다.

```bash
$ vi /etc/my.cnf.d/server.cnf

[mysqld]
# binary log의 파일명
log-bin = mysql-bin
# master와 slave의 값 구별
server-id = 2
# STATEMENT, ROW, MIXED 중 선택히여 포맷 지정
binlog_format = row
```

* * *

### Slave Server MariaDB 재시작 및 접속하기:
```bash
# MariaDB 재시작
$ service mariadb restart
또는
$ systemctl restart mariadb
```

```sql
> mysql -u root -p
```

* * *

### Slave Server 연동 설정하기:
```sql
> CHANGE MASTER TO 
  MASTER_HOST='Master Server IP주소', 
  MASTER_USER='user', 
  MASTER_PASSWORD='password', 
  MASTER_PORT=3306, 
  MASTER_LOG_FILE='Master Server File명', 
  MASTER_LOG_POS='Master Server Position 번호';
```

* * *

### Slave Server 실행 및 상태 확인하기:
```sql
> start slave;

> show slave status;
```
[![mariadb slave server 동작확인 화면](/assets/img/post/DB/mariadb%20slave%20server%20%EB%8F%99%EC%9E%91%ED%99%95%EC%9D%B8%20%ED%99%94%EB%A9%B4.PNG)](/assets/img/post/DB/mariadb%20slave%20server%20%EB%8F%99%EC%9E%91%ED%99%95%EC%9D%B8%20%ED%99%94%EB%A9%B4.PNG)

## MasterDB / SlaveDB 동기화 확인하기:
- 먼저 MasterDB에 데이터 베이스를 생성한다.
```sql
> create database testdb_replication character set UTF8;
```
[![mariadb masterdb에 db생성](/assets/img/post/DB/mariadb%20masterdb%EC%97%90%20db%EC%83%9D%EC%84%B1.PNG)](/assets/img/post/DB/mariadb%20masterdb%EC%97%90%20db%EC%83%9D%EC%84%B1.PNG)

- MasterDB에 테이블을 생성한다.
```sql
> use testdb_replication;
> create table test(
    -> test_idx int primary key auto_increment,
    -> test_name varchar(10) not null
    -> );
```
[![mariadb masterdb에 테이블 생성](/assets/img/post/DB/mariadb%20masterdb%EC%97%90%20%ED%85%8C%EC%9D%B4%EB%B8%94%20%EC%83%9D%EC%84%B1.PNG)](/assets/img/post/DB/mariadb%20masterdb%EC%97%90%20%ED%85%8C%EC%9D%B4%EB%B8%94%20%EC%83%9D%EC%84%B1.PNG)

- 생성한 테이블에 데이터를 삽입하고 조회한다.
```sql
> insert into test(test_name) values("yoonjae");
> select * from test;
```
[![mariadb masterdb에 데이터 삽입과 조회](/assets/img/post/DB/mariadb%20masterdb%EC%97%90%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20%EC%82%BD%EC%9E%85%EA%B3%BC%20%EC%A1%B0%ED%9A%8C.PNG)](/assets/img/post/DB/mariadb%20masterdb%EC%97%90%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20%EC%82%BD%EC%9E%85%EA%B3%BC%20%EC%A1%B0%ED%9A%8C.PNG)

* * *

- MasterDB에서 진행한 작업이 SlaveDB로 Sync되는지 확인한다.
```sql
> show databases;
> use testdb_replication
> select * from test;
```
[![mariadb slavedb sync 확인](/assets/img/post/DB/mariadb%20slavedb%20sync%20%ED%99%95%EC%9D%B8.PNG)](/assets/img/post/DB/mariadb%20slavedb%20sync%20%ED%99%95%EC%9D%B8.PNG)

* * *