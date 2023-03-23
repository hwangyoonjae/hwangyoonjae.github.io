---
title: "[Operating System] - 커널이란"
categories:
  - Operating System
tags:
  - [Kernel]

toc: true
toc_sticky: true

date: 2023-03-12
last_modified_at: 2023-03-12
---

## 컴퓨터의 구분:
- 컴퓨터는 **물리적으로 존재하지 않는 소프트웨어**와 **물리적으로 만질 수 있는 하드웨어(키보드, 모니터, 컴퓨터 본체와 본체 안에 있는 CPU, 메모리 등)**으로 나뉜다.
[![텍스트](/assets/images/Operating%20System/%EC%BB%B4%ED%93%A8%ED%84%B0%20%EA%B5%AC%EB%B6%84%20%EA%B5%AC%EC%A1%B0.PNG)](/assets/images/Operating%20System/%EC%BB%B4%ED%93%A8%ED%84%B0%20%EA%B5%AC%EB%B6%84%20%EA%B5%AC%EC%A1%B0.PNG)

- 소프트웨어 같은 경우에는 운영체제와 응용 프로그램으로 나뉘고, 운영체제는 커널과 시스템 프로그램으로 구분된다.

* * *

## 커널(Kernel)이란?:
- 운영체제 중 항상 메모리에 올라가 있는 **운영체제의 핵심 부분**으로써 시스템의 전반을 관리/감독하고, 하드웨어와 관련된 작업을 직접 수행한다.

* * *

## 커널(Kernel) 핵심기능:
- **프로세스 관리** : 프로세스에 CPU를 배분하고 작업에 필요한 제반 환경을 제공한다.
- **메모리 관리** : 프로세스에 작업 공간을 배치하고 실제 메모리보다 큰 가상공간을 제공한다.
- **파일 시스템** 관리 : 데이터를 저장하고 접근할 수 있는 인터페이스를 제공한다.
- **입출력 관리** : 필요한 입력과 출력 서비스를 제공한다.
- **프로세스간 통신 관리** : 공동 작업을 위한 각 프로세스 간 통신 환경을 지원한다.

[![텍스트](/assets/images/Linux/%EC%BB%A4%EB%84%90%EB%AA%A8%EB%93%9C%20%EA%B3%84%EC%B8%B5%EA%B5%AC%EC%A1%B0.PNG)](/assets/images/Linux/%EC%BB%A4%EB%84%90%EB%AA%A8%EB%93%9C%20%EA%B3%84%EC%B8%B5%EA%B5%AC%EC%A1%B0.PNG)<br>
<span style="color:#FA5858; font-size:12px">※ 커널 계층 구조는 위 그림과 같다.</span>

* * *

## 커널(Kernel)이 자원을 관리하는 이유:
- 사용자가 물리적인 하드웨어에 접근하고 사용할 수 있도록 매개하기 위해서이다.
- 커널은 사용자가 하드웨어에 접근하고 통신하기 위한 중간 다리 역할을 수행한다.

* * *