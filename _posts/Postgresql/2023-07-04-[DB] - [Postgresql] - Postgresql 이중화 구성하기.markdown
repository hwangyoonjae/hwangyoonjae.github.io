---
title: "[Postgresql] - Postgresql 이중화 구성하기"
categories:
  - Postgresql
tags:
  - [DB 이중화, Master, Slave]

toc: true
toc_sticky: true

date: 2023-07-04
last_modified_at: 2023-07-04
---

## Postgresql Replication 실습하기:
- Postgresql가 설치된 후 진행되는 실습으로 설치가 필요한 경우에는 아래 게시글 통해서 설치한다.
> * [Postgresql 설치하기](https://hwangyoonjae.github.io/postgresql/DB-Postgresql-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0/ "Postgresql 설치하기")

* * *

## Master Server 구성:
### Master Server 설정하기:
- 아카이브 디렉토리 생성한다.
```bash
$ mkdir -p /var/lib/pgsql/archive
$ chown -R postgres:postgres /var/lib/pgsql/archive
```

* * *

- Mater -> Slave로 DB 복제할 계정을 생성한다.
```bash
$ su - postgres
$ psql
$ create user [user명] replication login encrypted password 'password' connection limit -1;
$ \du
```
[![postgres DB 복제할 계정 생성 화면](/assets/images/DB/postgres%20DB%20%EB%B3%B5%EC%A0%9C%ED%95%A0%20%EA%B3%84%EC%A0%95%20%EC%83%9D%EC%84%B1%20%ED%99%94%EB%A9%B4.PNG)](/assets/images/DB/postgres%20DB%20%EB%B3%B5%EC%A0%9C%ED%95%A0%20%EA%B3%84%EC%A0%95%20%EC%83%9D%EC%84%B1%20%ED%99%94%EB%A9%B4.PNG)

* * *

- Postgresql 환경 설정을 위해 postgresql.conf 파일을 수정한다.
```bash
$ su – postgres
$ vi data/postgresql.conf
```
```bash
listen_addresses = '*'
port = 5432
wal_level = hot_standby
wal_log_hints = on
archive_mode = on
archive_command = 'test ! -f /var/lib/pgsql/archive/%f && cp %p /var/lib/pgsql/archive/%f'
max_wal_senders = 3
wal_keep_segments = 64
hot_standby = on
logging_collector = on
```

* * *

- 클라이언트의 주소와 역할 이름을 지정하고 모든 데이터베이스에 연결을 허용할지 여부를 설정하는데 사용하기 위해 pg_hba.conf 파일을 수정한다.
```bash
$ su – postgres
$ vi /var/lib/pgsql/data/pg_hba.conf
```
```bash
# 설정 방법
host   DATABASE  USER  ADDRESS  METHOD  [OPTIONS]
```
[![pg_hba.conf 파일 수정](/assets/images/DB/pg_hba.conf%20%ED%8C%8C%EC%9D%BC%20%EC%88%98%EC%A0%95.PNG)](/assets/images/DB/pg_hba.conf%20%ED%8C%8C%EC%9D%BC%20%EC%88%98%EC%A0%95.PNG)

* * *

- 위 설정 후 DB 재기동한다.
```bash
$ systemctl restart postgresql
```

* * *

### Slave Server 설정하기:
- Master -> Slave로 DB 복제를 위해 폴더명을 변경한다.
```bash
$ mv /var/lib/pgsql/10/data /var/lib/pgsql/data_bak
$ mkdir /var/lib/pgsql/data
$ chown -R postgres:postgres /var/lib/pgsql/data
$ chmod -R 0700 /var/lib/pgsql/data
$ su - postgres
$ export LANG=C
$ cd /usr/bin
$ ./pg_basebackup -h [MasterDB IP주소] -D /var/lib/pgsql/data -U [복제계정] -v -P -X stream
```
[![postgres Master 서버의 데이터 복제 화면](/assets/images/DB/postgres%20Master%20%EC%84%9C%EB%B2%84%EC%9D%98%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20%EB%B3%B5%EC%A0%9C%20%ED%99%94%EB%A9%B4.PNG)](/assets/images/DB/postgres%20Master%20%EC%84%9C%EB%B2%84%EC%9D%98%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20%EB%B3%B5%EC%A0%9C%20%ED%99%94%EB%A9%B4.PNG)

* * *

- Postgresql 환경 설정을 위해 postgresql.conf 파일을 수정한다.
```bash
$ su – postgres
$ vi data/postgresql.conf
```
```bash
listen_addresses = '*'
port = 5432
wal_level = hot_standby
wal_log_hints = on
archive_mode = on
archive_command = 'test ! -f /var/lib/pgsql/archive/%f && cp %p /var/lib/pgsql/archive/%f'
max_wal_senders = 3
wal_keep_segments = 64
hot_standby = on
logging_collector = on
```

* * *

- 이중화 복제 설정을 위해 recovery.conf 파일을 수정한다.
```bash
standby_mode = 'on'
primary_conninfo = 'host=[MaterDB IP주소] port=5432 user=[계정] password=1q2w3e4r!'
restore_command = 'cp /var/lib/pgsql/archive/%f %p'
recovery_target_timeline = 'latest'
trigger_file = '/tmp/postgresql.trigger.5432'
```

* * *

- 위 설정 후 DB 재기동한다.
```bash
$ systemctl restart postgresql
```

* * *