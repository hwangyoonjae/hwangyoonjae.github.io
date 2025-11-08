---
layout: post
title: "Oracle DB 설치하기"
date: 2023-01-11
categories: [DB, Oracle]
tags: [Oracle, MS-SQL]
image: /assets/img/post-title/oracle-wallpaper.jpg
---

## 1. Oralce이란? :
- 미국의 기업에서 만든 데이터 베이스 관리 시스템이며, 관계형 데이터베이스의 한 종류이다.

* * *

## 2. Oracle DB 설치 준비하기 :
### 2.1 Oracle 19c 파일을 다운로드 :
- 아래 URL 참고하여 oracle 설치파일과 설치 시 추가로 필요한 rpm파일까지 다운받는다.
> * [oracle 19c 다운받기](https://www.oracle.com/database/technologies/oracle-database-software-downloads.html "oracle 19c 다운받기")
![테스트](/assets/img/post/DB/oracle%20%EC%84%A4%EC%B9%98%ED%8C%8C%EC%9D%BC%20%EB%8B%A4%EC%9A%B4%EB%A1%9C%EB%93%9C.PNG)
> * [oracle-database-preinstall-19c 다운받기](https://yum.oracle.com/repo/OracleLinux/OL7/latest/x86_64/getPackage/oracle-database-preinstall-19c-1.0-1.el7.x86_64.rpm "oracle-database-preinstall-19c 다운받기")

* * *

### 2.2 Oracle 사용할 계정 및 폴더 생성하기 :
- Oracle 사용할 계정을 생성한다.

```bash
# 사용자 계정 생성
$ groupadd -g 1900 dba
$ useradd -g dba -u 1900 ora19c
$ passwd ora19c
```
![텍스트](/assets/img/post/DB/oracle%20user%20%EC%83%9D%EC%84%B1.PNG)

* * *

- Oracle 사용할 폴더를 생성한다.

```bash
# 폴더 생성하기
$ mkdir -p /oracle/ora19c/19c
$ mkdir -p /oracle/oraInventory
$ chown -R ora19c.dba /oracle/ora19c
$ chown -R ora19c.dba /oracle/oraInventory/
$ chgrp -R dba /oracle/
$ chmod -R 755 /oracle/
```
![텍스트](/assets/img/post/DB/oracle%20%ED%8F%B4%EB%8D%94%20%EC%83%9D%EC%84%B1.PNG)

* * *

### 2.3 도메인의 IP 설정하기 :
- IP에 두 도메인을 연결하여 설정한다.

```bash
$ vi /etc/hosts
```
```bash
# 아래 내용 추가
서버IP  testdb.itclass.co.kr  testdb
```
![텍스트](/assets/img/post/DB/oracle%20hostname%20%EC%A7%80%EC%A0%95.PNG)

* * *

### 2.4 ora19c 계정 환경변수 설정하기 :
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

### 2.5 Oracle 19c 설치하기 :
- 위에서 진행한 oracle 19c 설치파일을 옮겨서 설치한다.

```bash
$ yum install binutils combat-libcap1 combat-libstdc++-33 gcc gcc-c++ glibc glibc-devel ksh libgcc libstdc++ libstdc++-devel libaio libaio-devel make sysstat
$ cd /oracle/ora19c
$ rpm -Uvh oracle-database-preinstall-19c-1.0-1.el7.x86_64.rpm
$ rpm -Uvh oracle-database-ee-19c-1.0-1.x86_64.rpm
$ /etc/init.d/oracledb_ORCLCDB-19c configure
```
![텍스트](/assets/img/post/DB/oracle%20%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4%20%EC%83%9D%EC%84%B1.PNG)

* * *

- 오라클사 홈페이지에서 다운로드 받은 압축 파일을 해제 후 인스톨러 실행한다.

```bash
# ora19c 계정 사용
$ cd /oracle/ora19c/19c
$ unzip LINUX.X64_193000_db_home.zip
$ export DISPLAY=:0
$ xhost +
$ ./runInstaller
```
![텍스트](/assets/img/post/DB/oracle%20%EC%84%A4%EC%B9%98%ED%8C%8C%EC%9D%BC%20%EC%8B%A4%ED%96%89%ED%99%94%EB%A9%B4.PNG)

* * *

- 구성 옵션 선택에서 단일 인스턴스 데이터베이스 생성 및 구성을 선택한다.
![텍스트](/assets/img/post/DB/oracle%20%EA%B5%AC%EC%84%B1%20%EC%98%B5%EC%85%98%20%EC%84%A0%ED%83%9D%ED%99%94%EB%A9%B4.PNG)

* * *

- 다음 버튼 클릭 후 **INS-32042** 에러가 발생할 수 있다.

```html
[INS-32042] 사용자(ora19c)가 중앙 인벤토리 그룹(oinstall)의 멤버인지 확인하십시오.
```

* * *

- /etc/oraInst.loc 파일 열어 inst_group을 변경한다.

```bash
vi /etc/oraInst.loc
# 아래 내용 변경
inventory_loc=/opt/oracle/oraInventory
inst_group=dba
```

* * *

- 시스템 클래스에서 데스크톱 클래스를 선택한다.
![텍스트](/assets/img/post/DB/oracle%20%EC%8B%9C%EC%8A%A4%ED%85%9C%20%ED%81%B4%EB%9E%98%EC%8A%A4%20%EC%84%A0%ED%83%9D%ED%99%94%EB%A9%B4.PNG)

* * *

- 일반 설치 구성에서 데이터베이스 설치를 설정한다.

![텍스트](/assets/img/post/DB/oracle%20%EC%9D%BC%EB%B0%98%20%EC%84%A4%EC%B9%98%20%EA%B5%AC%EC%84%B1%ED%99%94%EB%A9%B4.PNG)

* * *

- 자동으로 구성 스크립트 실행을 선택하고 루트 사용자 인증서 비밀번호를 입력한다.

![텍스트](/assets/img/post/DB/oracle%20%EB%A3%A8%ED%8A%B8%20%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8%20%EC%8B%A4%ED%96%89%20%EA%B5%AC%EC%84%B1%ED%99%94%EB%A9%B4.PNG)

* * *

- 모든 설정 완료 후 설치를 진행한다.

![텍스트](/assets/img/post/DB/oracle%20%EC%A0%9C%ED%92%88%20%EC%84%A4%EC%B9%98%ED%99%94%EB%A9%B4.PNG)

* * *

### 2.6 Oracle 19c 실행하기 :
- oracle 19c에 로그인한다.

```bash
$ sqlplus / as sysdba
```
![텍스트](/assets/img/post/DB/oracle%20%EC%8B%A4%ED%96%89%ED%99%94%EB%A9%B4.PNG)

* * *

- 오라클 서비스 실행은 아래와 같다.

```bash
# 오라클 시작
SQL> startup
```
```bash
# 오라클 종료
SQL> shutdown immediate
```

* * *