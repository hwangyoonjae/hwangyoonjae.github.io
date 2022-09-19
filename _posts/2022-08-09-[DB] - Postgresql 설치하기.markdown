---
title: "[DB] - Postgresql 설치하기"
layout: post
date: 2022-08-09
image: /assets/images/Post/postgre.png
headerImage: true
tag:
- DBMS
- ORDBMS
- PostgreSQL
category: blog
author: hwangyoonjae
description: Markdown summary with different options
---

## PostgreSQL란? :
- 오픈소스 데이터베이스이며, 관계형 데이터베이스 시스템(RDBMS)의 일종이다.
- 가장 진보한 오픈소스 데이터베이스 시스템이라고 할 수 있으며, Unix/Linux, macOS, Solaris, Windows 등의 OS를 지원한다.

* * *

## PostgreSQL 설치하기 :
### Yum 통해서 설치하기 :
```bash
# 레퍼지토리 RPM 설치하기 :
$ yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm

# PostgreSQL 설치하기 :
$ yum install -y postgresql14-server postgresql14-contrib
```
<span style="color:#FA5858; font-size:12px">※ 필자는 14버전을 사용했다.</span>
* * *

### 기본 데이터베이스 생성하기 :
- initdb 명령어를 통해 기본 데이터베이스를 설치하고, 기본 데이터베이스는 postgres라는 이름으로 생성된다.
```bash
# PostgreSQL 초기화
$ /usr/pgsql-14/bin/postgresql-14-setup initdb
```

* * *

### 서비스 등록 및 실행 :
```bash
# 서비스 등록
$ systemctl enable postgresql-14

# 서비스 실행
$ systemctl start postgresql-14

# 서비스 동작 확인
$ systemctl status postgresql-13
```

* * *

### postgresql 접속:
```bash
$ sudo -u postgres psql
또는
# 전용 계정으로 전환
$ su - postgres
 
# psql 실행
$ psql
 
# 초기 DB 확인
$ \l
```

* * *

### 데이터베이스 생성:
```
postgres=# create database <name> encoding 'utf-8';
```

* * *