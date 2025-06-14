---
layout: post
title: "유저모드(User Mode) VS 커널모드(Kernel Mode)"
date: 2023-03-07
categories: OperatingSystem
tags: [User Mode, Kernel Mode]
image: /assets/img/post-title/os-wallpaper.jpg
---

## 유저모드(User Mode) VS 커널모드(Kernel Mode):
- 커널에서 중요한 자원을 관리하기 때문에, 사용자가 그 중요한 자원에 접근하지 못하도록 모드를 유저모드(User Mode)와 커널모드(Kernel Mode) 2가지로 나누었다.

* * *

## 커널모드(Kernel Mode)란?:
- 모든 자원(드라이버, 메모리, CPU 등)에 접근하고 명령을 할 수 있다.
- CPU에서 커널코드가 실행되고, 처리가 완료되면 중단됐던 프로그램의 CPU 상태를 복원한다.

### 커널모드(Kernel Mode)를 만든 이유:
- 커널모드가 없으면 우리가 개발한 프로그램들이 하드웨어를 함부로 점유해서 사용하고 다른프로세스 영향을 받게되고, 전체 컴퓨터 시스템이 붕괴될 수 있기 때문에 시스템 전반적인 부분과 하드웨어 관련된 부분들을 커널이 담당하고 우리가 만든 프로그램들은 커널을 통해서 시스템이 안정적으로 돌리면서 시스템 보호를 위해서다.

* * *

## 유저모드(User Mode)란?:
- 사용자가 접근할 수 있는 영역을 제한적으로 두고, 프로그램의 자원에 함부로 침범하지 못하는 모드이다.
- 여기서 코드를 작성하고, 프로세스를 실행하는 등의 행동을 할 수 있다.

* * *

## 유저모드(User Mode)와 커널모드(Kernel Mode)의 전환:
- 프로세스가 실행되는 동안 프로세스는 수없이 유저모드와 커널모드를 왔다갔다 하면서 실행이 된다.

### 유저모드(User Mode)에서 커널모드(Kernel Mode)로 요청:
- 프로그램 실행 중에 **인터럽트**가 발생하거나 **시스템 콜**을 호출하게 되면 커널 모드로 전환한다.

### 커널모드(Kernel Mode)에서 유저모드(User Mode)로 반환:
- **시스템 콜** 요청에 대한 일을 하고 결과 값을 리턴 값으로 전달한다.

[![텍스트](/assets/img/post/Operating%20System/%EC%9C%A0%EC%A0%80%EB%AA%A8%EB%93%9C%20%EC%BB%A4%EB%84%90%EB%AA%A8%EB%93%9C%20%ED%9D%90%EB%A6%84.png)](/assets/img/post/Operating%20System/%EC%9C%A0%EC%A0%80%EB%AA%A8%EB%93%9C%20%EC%BB%A4%EB%84%90%EB%AA%A8%EB%93%9C%20%ED%9D%90%EB%A6%84.png)

* * *

## 유저모드(User Mode)와 커널모드(Kernel Mode)를 나눈 이유:
- 커널에서 중요한 자원, 즉 운영체제를 실행시키기 위한 자원을 관리하기 때문에 일반 사용자가 그 중요한 자원에 접근하지 못하도록 하기 위해서다.

* * *