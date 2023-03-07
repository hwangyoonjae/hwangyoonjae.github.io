---
title: "[Operating System] - User모드 VS Kernel모드"
categories:
  - Operating System
tags:
  - [User Mode, Kernel Mode]

toc: true
toc_sticky: true

date: 2023-03-07
last_modified_at: 2023-03-07
---

## 커널모드(Kernel Mode)란?:
- 운영체제 중 항상 메모리에 올라가 있는 운영체제의 핵심 부분으로써 하드웨어와 응용 프로그램 사이에서 인터페이스를 제공하는 역할을 하며 컴퓨터 자원들을 관리하는 역할을 한다.

* * *

## 커널모드(Kernel Mode) 핵심기능:
- 프로세스 관리 : 프로세스에 CPU를 배분하고 작업에 필요한 제반 환경을 제공한다.
- 메모리 관리 : 프로세스에 작업 공간을 배치하고 실제 메모리보다 큰 가상공간을 제공한다.
- 파일 시스템 관리 : 데이터를 저장하고 접근할 수 있는 인터페이스를 제공한다.
- 입출력 관리 : 필요한 입력과 출력 서비스를 제공한다.
- 프로세스간 통신 관리 : 공동 작업을 위한 각 프로세스 간 통신 환경을 지원한다.

[![텍스트](/assets/images/Linux/%EC%BB%A4%EB%84%90%EB%AA%A8%EB%93%9C%20%EA%B3%84%EC%B8%B5%EA%B5%AC%EC%A1%B0.PNG)](/assets/images/Linux/%EC%BB%A4%EB%84%90%EB%AA%A8%EB%93%9C%20%EA%B3%84%EC%B8%B5%EA%B5%AC%EC%A1%B0.PNG)
<span style="color:#FA5858; font-size:12px">※ 커널 계층 구조는 위 그림과 같다.</span>

* * *