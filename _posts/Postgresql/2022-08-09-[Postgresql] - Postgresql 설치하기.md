---
layout: post
title: "Postgresql 설치하기"
date: 2022-08-09
categories: Postgresql
tags: [DBMS, ORDBMS, PostgreSQL]
image: /assets/img/post-title/postgresql-wallpaper.jpg
---

## PostgreSQL란?:
- 오픈 소스 객체-관계형 데이터베이스 시스템(ORDBMS)으로, Enterprise급 DBMS의 기능과 차세대 DBMS에서나 볼 수 있을 법한 기능들을 제공한다.
- 가장 진보한 오픈소스 데이터베이스 시스템이라고 할 수 있으며, Unix/Linux, macOS, Solaris, Windows 등의 OS를 지원한다.

* * *

### ORDBMS란?: 
- 객체 지향 데이터베이스 시스템과 관계형 데이터베이스 시스템을 기반으로하며 복잡한 객체가 중심 역할을 하는 DBMS입니다.
```bash
# RDBMS와 ORDBMS 차이점
RDBMS : 행과 열이 있는 하나 이상의 관계 또는 테이블의 모음이다.
ORDBMS : 데이터가 객체로 저장된 것처럼 작동한다.
```

* * *

## PostgreSQL 설치하기:
### Yum 통해서 설치하기:
```bash
# PostgreSQL 설치하기 :
$ yum install -y postgresql
$ yum install -y postgresql-server
```

* * *

### 폐쇄망 설치:
- 폐쇄망 서버에 PostgreSQL를 설치하는 경우 아래 URL 접속 시 CentOS 버전에 맞춰 폴더 접속한 후 **/EL-Linux버전-x86_64/**
> * [PostgreSQL RPM 패키지 다운로드](https://download.postgresql.org/pub/repos/yum/reporpms/ "PostgreSQL RPM 패키지 다운로드")

```bash
# rpm으로 설치
$ rpm -Uvh pgdg-redhat-repo-latest.noarch.rpm --nodeps
```

* * *

### 기본 데이터베이스 생성하기:
- initdb 명령어를 통해 기본 데이터베이스를 설치하고, 기본 데이터베이스는 postgres라는 이름으로 생성된다.
```bash
# PostgreSQL 초기화
$ postgresql-setup initdb
```
[![postgres DB 초기화](/assets/img/post/DB/postgres%20DB%20%EC%B4%88%EA%B8%B0%ED%99%94.PNG)](/assets/img/post/DB/postgres%20DB%20%EC%B4%88%EA%B8%B0%ED%99%94.PNG)

* * *

### 서비스 등록 및 실행하기:
```bash
# 서비스 등록
$ systemctl enable postgresql

# 서비스 실행
$ systemctl start postgresql

# 서비스 동작 확인
$ systemctl status postgresql
```
[![텍스트](/assets/img/post/DB/postgres%20%EC%84%9C%EB%B9%84%EC%8A%A4%20%ED%99%95%EC%9D%B8.PNG)](/assets/img/post/DB/postgres%20%EC%84%9C%EB%B9%84%EC%8A%A4%20%ED%99%95%EC%9D%B8.PNG)

* * *

### postgresql 접속하기:
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

### postgresql 초기 비밀번호 변경하기:
```sql
ALTER USER postgres PASSWORD '변경할 비밀번호';
```
[![텍스트](/assets/img/post/DB/postgres%20%EC%B4%88%EA%B8%B0%20%ED%8C%A8%EC%8A%A4%EC%9B%8C%EB%93%9C%20%EB%B3%80%EA%B2%BD.PNG)](/assets/img/post/DB/postgres%20%EC%B4%88%EA%B8%B0%20%ED%8C%A8%EC%8A%A4%EC%9B%8C%EB%93%9C%20%EB%B3%80%EA%B2%BD.PNG)

* * *

### 데이터베이스 생성:
```
postgres=# create database <name> encoding 'utf-8';
```

* * *

### postgresql 원격 접속 허용과 포트 번호 지정하기:
```bash
$ vi /var/lib/pgsql/14/data/postgresql.conf
```
[![텍스트](/assets/img/post/DB/%EC%9B%90%EA%B2%A9%20%EC%A0%91%EC%86%8D%EA%B3%BC%20%ED%8F%AC%ED%8A%B8%EB%B2%88%ED%98%B8%20%EC%84%A4%EC%A0%95.PNG)](/assets/img/post/DB/%EC%9B%90%EA%B2%A9%20%EC%A0%91%EC%86%8D%EA%B3%BC%20%ED%8F%AC%ED%8A%B8%EB%B2%88%ED%98%B8%20%EC%84%A4%EC%A0%95.PNG)

* * *