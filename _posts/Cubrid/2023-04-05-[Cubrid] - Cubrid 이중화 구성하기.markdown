---
title: "[Cubrid] - Cubrid 이중화 구성하기"
categories:
  - Cubrid
tags:
  - [DB, Cubrid, Master, Slave]

toc: true
toc_sticky: true

date: 2023-04-05
last_modified_at: 2023-04-05
---

## CUBRID 이중화 실습하기:
- 고객사 이중화 구축한걸 바탕으로 작성하였다.

* * *

## CUBRID 설치 준비하기:
### CUBRID 설치파일 다운로드:
- 아래 URL 참고하여 CUBRID 설치파일을 다운받는다.
> * [CUBRID 다운받기](https://www.cubrid.com/downloads "CUBRID 다운받기")

* * *

## CUBRID 설치하기:
- FTP 통해서 CUBRID 설치파일을 서버에 복사한다.

```bash
$ sh CUBRID-10.2-latest-Linux.x86_64.sh
```
* * *

## CUBRID 서비스 시작하기:

```bash
$ cubrid service start
```
* * *