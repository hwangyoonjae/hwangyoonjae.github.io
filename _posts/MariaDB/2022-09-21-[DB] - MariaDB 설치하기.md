---
layout: post
title: "MariaDB 설치하기"
date: 2022-09-21
categories: [DB, MariaDB]
tags: [RDBMS, MariaDB, MySQL]
image: /assets/img/post-title/mariadb-wallpaper.jpg
---

## MariaDB란?:
- 오픈 소스의 관계형 데이터베이스 관리 시스템(RDBMS)이다.

### RDBMS(Relational Database Management System)란?:
- 관계형 데이터베이스를 생성하고 수정하고 관리할 수 있는 소프트웨어이다.
- RDBMS의 테이블은 서로 연관되어 있어 일반 DBMS보다 효율적으로 데이터를 저장, 구성 및 관리할 수 있다.

* * *

## MariaDB 설치하기:
### MariaDB yum 저장소 추가하기:
```bash
$ vi /etc/yum.repos.d/MariaDB.repo

# 아래 내용 입력
[mariadb]
name=MariaDB
baseurl=http://yum.mariadb.org/10.8/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
```

* * *

### Yum 통해서 설치하기:
```bash
$ yum install MariaDB
```

* * *

### MariaDB 설치 및 버전 확인하기:
```bash
# 설치확인
$ rpm -qa | grep -i mariadb
```
[![텍스트](/assets/img/post/DB/mariadb%20%EC%84%A4%EC%B9%98%ED%99%95%EC%9D%B8.PNG)](/assets/img/post/DB/mariadb%20%EC%84%A4%EC%B9%98%ED%99%95%EC%9D%B8.PNG)

```bash
# 버전확인
$ mariadb --version
```
[![텍스트](/assets/img/post/DB/mariadb%20%EB%B2%84%EC%A0%84%20%ED%99%95%EC%9D%B8%ED%95%98%EA%B8%B0.PNG)](/assets/img/post/DB/mariadb%20%EB%B2%84%EC%A0%84%20%ED%99%95%EC%9D%B8%ED%95%98%EA%B8%B0.PNG)

* * *

### MariaDB 실행 및 자동실행 등록하기:
```bash
# 서비스 시작
$ systemctl start mariadb

# 서비스 자동실행 등록
$ systemctl enable mariadb
```
[![텍스트](/assets/img/post/DB/mariadb%20%EC%8B%A4%ED%96%89%ED%99%95%EC%9D%B8%20%EB%B0%8F%20%EC%9E%90%EB%8F%99%EC%8B%A4%ED%96%89%20%EB%93%B1%EB%A1%9D.PNG)](/assets/img/post/DB/mariadb%20%EC%8B%A4%ED%96%89%ED%99%95%EC%9D%B8%20%EB%B0%8F%20%EC%9E%90%EB%8F%99%EC%8B%A4%ED%96%89%20%EB%93%B1%EB%A1%9D.PNG)

* * *

### MariaDB 계정 비밀번호 변경하기:
```
$ /usr/bin/mysqladmin -u root password
```
[![텍스트](/assets/img/post/DB/mariadb%20root%EA%B3%84%EC%A0%95%20%ED%8C%A8%EC%8A%A4%EC%9B%8C%EB%93%9C%20%EB%B3%80%EA%B2%BD%ED%95%98%EA%B8%B0.PNG)](/assets/img/post/DB/mariadb%20root%EA%B3%84%EC%A0%95%20%ED%8C%A8%EC%8A%A4%EC%9B%8C%EB%93%9C%20%EB%B3%80%EA%B2%BD%ED%95%98%EA%B8%B0.PNG)

* * *

### MariaDB 접속하기:
```bash
$ mysql -u root -p
```
[![텍스트](/assets/img/post/DB/mariadb%20%EC%A0%91%EC%86%8D%ED%99%94%EB%A9%B4.PNG)](/assets/img/post/DB/mariadb%20%EC%A0%91%EC%86%8D%ED%99%94%EB%A9%B4.PNG)

* * *