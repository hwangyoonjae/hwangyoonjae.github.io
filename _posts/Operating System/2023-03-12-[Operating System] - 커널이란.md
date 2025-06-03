---
layout: post
title: "커널이란"
date: 2023-03-12
categories: OperatingSystem
tags: [Kernel]
image: /assets/img/post-title/os-wallpaper.jpg
---

## 컴퓨터의 구분:
- 컴퓨터는 **물리적으로 존재하지 않는 소프트웨어**와 **물리적으로 만질 수 있는 하드웨어(키보드, 모니터, 컴퓨터 본체와 본체 안에 있는 CPU, 메모리 등)**으로 나뉜다.
[![텍스트](/assets/img/post/Operating%20System/%EC%BB%B4%ED%93%A8%ED%84%B0%20%EA%B5%AC%EB%B6%84%20%EA%B5%AC%EC%A1%B0.PNG)](/assets/img/post/Operating%20System/%EC%BB%B4%ED%93%A8%ED%84%B0%20%EA%B5%AC%EB%B6%84%20%EA%B5%AC%EC%A1%B0.PNG)

- 소프트웨어 같은 경우에는 운영체제와 응용 프로그램으로 나뉘고, 운영체제는 커널과 시스템 프로그램으로 구분된다.

* * *

## 운영체제(Operating System)란?:
- 컴퓨터 시스템의 자원들을 효율적으로 관리하며, 사용자가 컴퓨터를 편리하고, 효과적으로 사용할 수 있도록 환경을 제공하는 여러 프로그램의 모임이다.

* * *

## 운영체제(Operating System) 연산들:
### 멀티프로그래밍:
- 메모리에 여러개의 응용 프로그램을 적재하는 것이다.
- 사용하는 이유로는 메모리에 여러개의 프로그램을 올려두어서 CPU가 하나의 프로그램이 대기중일때 다른 프로그램을 수행시켜 효율을 극대화하기 위해서다.

### 운영체제 연산의 2가지 모드:
- 운영체제 연산의 2가지 모드로는 **유저모드와 커널모드**가 있다.
> * [유저모드와 커널모드 자세히게 알아보기](https://hwangyoonjae.github.io/operating%20system/Operating-System-%EC%9C%A0%EC%A0%80%EB%AA%A8%EB%93%9C(User-Mode)-VS-%EC%BB%A4%EB%84%90%EB%AA%A8%EB%93%9C(Kernel-Mode)/ "유저모드와 커널모드 자세히게 알아보기")

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
[![커널계층구조](/assets/img/post/Linux/%EC%BB%A4%EB%84%90%EB%AA%A8%EB%93%9C%20%EA%B3%84%EC%B8%B5%EA%B5%AC%EC%A1%B0.PNG)](/assets/img/post/Linux/%EC%BB%A4%EB%84%90%EB%AA%A8%EB%93%9C%20%EA%B3%84%EC%B8%B5%EA%B5%AC%EC%A1%B0.PNG)<br>
<span style="color:#FA5858; font-size:12px">※ 커널 계층 구조는 위 그림과 같다.</span>

* * *

## 커널(Kernel)이 자원을 관리하는 이유:
- 사용자가 물리적인 하드웨어에 접근하고 사용할 수 있도록 매개하기 위해서이다.
- 커널은 사용자가 하드웨어에 접근하고 통신하기 위한 중간 다리 역할을 수행한다.

* * *