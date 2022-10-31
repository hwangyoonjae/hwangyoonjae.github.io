---
title: "[DB] - MariaDB 이중화 구성하기"
layout: post
date: 2022-10-28
image: /assets/images/Post/mariadb.png
headerImage: true
tag:
- DB 이중화
category: blog
author: hwangyoonjae
description: Markdown summary with different options
---

## MariaDB Replication 절차:
- MariaDB에서 기본적으로 제공하는 Replication 기능으로 Master-Slave 구조로 되어있다.
- Master 서버의 Binary로그를 Slave서버가 Relay로그에 저장해서 복제하는 방식이다.
[![텍스트](/assets/images/DB/MariaDB%20Replication.PNG)](/assets/images/DB/MariaDB%20Replication.PNG)
- MasterDB에 이벤트가 발생하면 SlaveDB와 복제를 위해 생성한 Binaary log에 DB업데이트와 동시에 기록한다.
- SlaveDB는 자신이 MasterDB의 몇 번째 위치의 데이터를 마지막으로 가져왔는지 기록했다가 MasterDB의 Binaary Log에 새로운 기록이 업데이트 되면 가져오고 가져온 위치를 기억한 후 SlaveDB는 전달받은 Binaary log를 Relay Logdp 기록하여 순차적으로 DB에 저장한다.

* * *

## MariaDB Replication 실습하기:
- Maria DB가 설치된 후 진행되는 실습이다.

* * *

## Master Server 구성하기:
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
[![텍스트](/assets/images/DB/master%20%EC%84%9C%EB%B2%84%20%EC%83%81%ED%83%9C%20%ED%99%95%EC%9D%B8.PNG)](/assets/images/DB/master%20%EC%84%9C%EB%B2%84%20%EC%83%81%ED%83%9C%20%ED%99%95%EC%9D%B8.PNG)

* * *

## Slave Server 구성하기:
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