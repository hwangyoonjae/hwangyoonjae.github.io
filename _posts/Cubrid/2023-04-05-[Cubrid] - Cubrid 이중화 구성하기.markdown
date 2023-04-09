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
- FTP 통해서 CUBRID 설치파일을 서버에 복사 후 파일을 실행한다.
```bash
$ sh CUBRID-10.2-latest-Linux.x86_64.sh
```

- 실행 후 아래와 같이 문구 나오면 q를 누르고
[![텍스트](/assets/images/DB/CUBRID%20%EC%B4%88%EA%B8%B0%20%EC%84%A4%EC%B9%98%20%EC%8B%9C%20%ED%99%94%EB%A9%B4.PNG)](/assets/images/DB/CUBRID%20%EC%B4%88%EA%B8%B0%20%EC%84%A4%EC%B9%98%20%EC%8B%9C%20%ED%99%94%EB%A9%B4.PNG)

- 아래 그림과 같이 진행한다.
[![텍스트](/assets/images/DB/CUBRID%20%EC%84%A4%EC%B9%98%ED%99%94%EB%A9%B4.PNG)](/assets/images/DB/CUBRID%20%EC%84%A4%EC%B9%98%ED%99%94%EB%A9%B4.PNG)

* * *

## CUBRID 서비스 시작하기:
- CUBRID 설치한 계정 경로(/home/계정)으로 들어가 명령어를 입력하여 서비스 시작한다.
```bash
$ . .cubrid.sh
$ cubrid service start
```

* * *