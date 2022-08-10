---
title: "[DB] - Postgresql 설치하기"
layout: post
date: 2022-08-09
image: /assets/images/Post/postgre.png
headerImage: true
tag:
- WEB
- WAS
- 동적페이지
- 정적페이지
category: blog
author: hwangyoonjae
description: Markdown summary with different options
---

## PostgreSQL 설치하기 :

### Yum 통해서 설치하기 :
```javascript
# 레퍼지토리 RPM 설치하기 :
$ yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm

# PostgreSQL 설치하기 :
$ yum install -y postgresql14-server postgresql14-contrib
```
<span style="color:#FA5858; font-size:12px">※ 필자는 14버전을 사용했다.</span>
* * *

### 기본 데이터베이스 생성하기 :
- initdb 명령어를 통해 기본 데이터베이스를 설치하고, 기본 데이터베이스는 postgres라는 이름으로 생성된다.
```javascript
$ /usr/pgsql-14/bin/postgresql-14-setup initdb
```

* * *

### 서비스 등록 및 실행 :
```javascript
$ sudo systemctl enable postgresql-14
$ sudo systemctl start postgresql-14
```

* * *

### postgresql 접속:
```javascript
$ sudo -u postgres psql
```

* * *

### 데이터베이스 생성:
```javascript
postgres=# create database <name> encoding 'utf-8';
```

* * *