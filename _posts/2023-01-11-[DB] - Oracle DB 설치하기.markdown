---
title: "[DB] - Oracle DB 설치하기"
layout: post
date: 2023-01-11
image: /assets/images/Post/oracle.png
headerImage: true
tag:
- Oracle
- MS-SQL
category: blog
author: hwangyoonjae
description: Markdown summary with different options
---

## Oralce이란?:
- 미국의 기업에서 만든 데이터 베이스 관리 시스템이며, 관계형 데이터베이스의 한 종류이다.

* * *

## Oracle DB 설치 준비하기:
### Oracle 19c 파일을 다운로드:
- 아래 URL 참고하여 oracle 설치파일과 설치 시 추가로 필요한 rpm파일까지 다운받는다.
> * [oracle 19c 다운받기](https://www.oracle.com/database/technologies/oracle-database-software-downloads.html "oracle 19c 다운받기")
[![테스트](/assets/images/DB/oracle%20%EC%84%A4%EC%B9%98%ED%8C%8C%EC%9D%BC%20%EB%8B%A4%EC%9A%B4%EB%A1%9C%EB%93%9C.PNG)](/assets/images/DB/oracle%20%EC%84%A4%EC%B9%98%ED%8C%8C%EC%9D%BC%20%EB%8B%A4%EC%9A%B4%EB%A1%9C%EB%93%9C.PNG)
> * [oracle-database-preinstall-19c 다운받기](https://yum.oracle.com/repo/OracleLinux/OL7/latest/x86_64/getPackage/oracle-database-preinstall-19c-1.0-1.el7.x86_64.rpm "oracle-database-preinstall-19c 다운받기")

* * *

### oracle 사용할 계정 및 폴더 생성하기:
- oracle 사용할 계정을 생성한다.
```bash
# 사용자 계정 생성
$ groupadd -g 1900 dba
$ useradd -g dba -u 1900 ora19c
$ passwd ora19c
```
[![텍스트](/assets/images/DB/oracle%20user%20%EC%83%9D%EC%84%B1.PNG)](/assets/images/DB/oracle%20user%20%EC%83%9D%EC%84%B1.PNG)

- oracle 사용할 폴더를 생성한다.
```bash
# 폴더 생성하기
$ mkdir -p /oracle/ora19c/19c
$ mkdir -p /oracle/oraInventory
$ chown -R ora19c.dba /oracle/ora19c
$ chown -R ora19c.dba /oracle/oraInventory/
$ chgrp -R dba /oracle/
$ chmod -R 755 /oracle/
```
[![텍스트](/assets/images/DB/oracle%20%ED%8F%B4%EB%8D%94%20%EC%83%9D%EC%84%B1.PNG)](/assets/images/DB/oracle%20%ED%8F%B4%EB%8D%94%20%EC%83%9D%EC%84%B1.PNG)

* * *

### 도메인의 IP 설정하기 :
- IP에 두 도메인을 연결하여 설정한다.
```bash
$ vi /etc/hosts
```
```bash
# 아래 내용 추가
서버IP  DB19.itclass.co.kr  DB19
```

* * *

### ora19c 계정 환경변수 설정하기:
- ora19c 계정으로 오라클 운영관리를 설정한다.
```bash
# ora19c 계정으로 로그인 후 진행
$ vi .bash_profile
```
```bash
# 아래 내용 추가
export ORACLE_OWNER=ora19c
export ORACLE_BASE=/oracle/ora19c
export ORACLE_HOME=/oracle/ora19c/19c
export TNS_ADMIN=$ORACLE_HOME/network/admin
export ORACLE_SID=testdb
export NLS_LANG=AMERICAN_AMERICA.AL32UTF8
export ORACLE_HOSTNAME=testdb.co.kr
export TMP=/tmp
export TMPDIR=$TMP
export PATH=$PATH:$ORACLE_HOME/bin:$ORACLE_HOME:/usr/bin:.
```
```bash
# 수정한 환경변수 적용
$ source .bash_profile
```

* * *

### Oracle 19c 설치하기:
- 위에서 진행한 oracle 19c 설치파일을 옮겨서 설치한다.
```bash
$ yum install binutils combat-libcap1 combat-libstdc++-33 gcc gcc-c++ glibc glibc-devel ksh libgcc libstdc++ libstdc++-devel libaio libaio-devel make sysstat
$ cd /oracle/ora19c
$ rpm -Uvh oracle-database-preinstall-19c-1.0-1.el7.x86_64.rpm
$ rpm -Uvh oracle-database-ee-19c-1.0-1.x86_64.rpm
$ /etc/init.d/oracledb_ORCLCDB-19c configure
```
[![텍스트](/assets/images/DB/oracle%20%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4%20%EC%83%9D%EC%84%B1.PNG)](/assets/images/DB/oracle%20%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4%20%EC%83%9D%EC%84%B1.PNG)

* * *