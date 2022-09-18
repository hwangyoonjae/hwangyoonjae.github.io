---
title: "[Linux] - Crontab 설정하기"
layout: post
date: 2022-09-18
image: /assets/images/Post/crontab.png
headerImage: true
tag:
- Crontab
category: blog
author: hwangyoonjae
description: Markdown summary with different options
---

## Crontab(크론탭)에 대해 알고 싶었던 계기:
- 서버 설치 과정에서 연동이나 데이터 수집을 위해 Crontab 설정을 해야 한다.<br>
막상 설정을 하려고 하면 기본적인 명령어는 알아도 담당자나 내가 원하는 대로 설정하지 못하여 인터넷에 검색을 통해 설정하지만,<br>보면서도 헷갈리는 게 많아 작성하게 되었다.

* * *

## Crontab(크론탭)이란?:
- 스케줄링을 관리하는 프로그램으로써 시스템 관리자에게 매우 중요한 유틸이다.
- 반복적인 작업을 정의하여 실행해주는 자동 매크로라고 생각하면된다.

* * *

## Crontab(크론탭) 기본 명령어:
- 예약된 작업 리스트 수정한다.
- 각종 크론탭 명령어를 입력후 콜론(:) 입력 후에 wq를 입력해 크론탭을 갱신한다.
  ```bash
  # crontab 작성
  $ crontab -e
  ```

- 예약된 작업 리스트 출력한다.
- cat 명령어로 파일을 읽어들인 것처럼 표준 출력으로 크론탭 내용이 나온다.
  ```bash
  # crontab 작업 리스트 보기
  $ crontab -l
  ```

- 예약된 작업 리스트 목록 삭제한다.
- 설정 및 수정한 크론탭은 삭제된다.
  ```bash
  # crontab 모든 작업 삭제
  $ crontab -r
  ```

* * *

## Crontab(크론탭) 주기 설정:
- Crontab(크론탭)은 아래와 같이 설정 가능하다.
  ```bash
  *　　　　　　*　　　　　　*　　　　　　*　　　　　　*
  분(0-59)　　시간(0-23)　　일(1-31)　　월(1-12)　　　요일(0-7)(0,7 : 일요일 / 1 : 월요일 / 2 : 화요일...)
  ```

### Crontab(크론탭) 주기 설정 방법:
- "*"표시는 해당 필드의 모든 시간을 의미한다.
- 3,5,7 같이 콤마로 구분하여 여러 시간대를 지정할 수 있다.
- 2-10과 같이 하이픈으로 시간 범위를 지정할 수 있다.
-  2-10/3과 같이 하이픈으로 시간 범위를, 슬래쉬로 시간 간격을 지정할 수 있다.

* * *

## crontab 명령 중복 실행 방지방법:
- flock -n 을 사용하여, shell script가 중복 실행 되지 않도록 수행 할 수 있다.
  ```bash
  $ * * * * * /usr/bin/flock -n [실행 스크립트]
  ```
- 만약 shell script가 1분 이상 수행이 된다면, 중복 실행이 되어 예상치 않은 결과가 나올 수 있다.

* * *