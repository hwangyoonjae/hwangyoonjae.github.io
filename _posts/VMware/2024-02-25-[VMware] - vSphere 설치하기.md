---
layout: post
title: "vSphere 설치하기"
date: 2024-02-25
categories: VMware
tags: [VMware, vSphere]
image: /assets/img/post-title/vmware-wallpaper.jpg
---

## vSphere 설치하기:
### vSphere 설치 파일 다운로드:
- 아래 주소 접속하여 vSphere설치 파일 다운로드 후 ISO 파일을 다운로드한다.
> * [ESXI ISO 파일 다운로드](https://customerconnect.vmware.com/downloads/#all_products "ESXI ISO 파일 다운로드")

* * *

### vSphere 부팅 USB 만들기:
- 해당 ISO 파일을 통하여 부팅 USB를 만든다.
- 필자는 Rufus Tool을 통하여 부팅 USB를 만들었다.
> * [Rufus Tool 사용하기](https://rufus.ie/ko/ "Rufus Tool 사용하기")

* * *

## vSphere 설치하기:
- 부팅 USB를 서버에 연결하여 설치 진행한다.  
[![vSphere 부팅화면](/assets/img/post/VMware/vSphere%20부팅화면.png)](/assets/img/post/VMware/vSphere%20부팅화면.png)

- 설치환영 메세지 창으로, ***Enter*** 클릭한다.  
[![vSphere 설치환영화면](/assets/img/post/VMware/vSphere%20설치환영화면.png)](/assets/img/post/VMware/vSphere%20설치환영화면.png)

- 라이센스 동의 창으로 ***F11*** 클릭한다.  
[![vSphere 라이센스 동의](/assets/img/post/VMware/vSphere%20라이센스%20동의.png)](/assets/img/post/VMware/vSphere%20라이센스%20동의.png)

- 가상 디스크 선택 창으로 ***Enter*** 클릭한다.  
[![vSphere 가상디스크](/assets/img/post/VMware/vSphere%20가상디스크.png)](/assets/img/post/VMware/vSphere%20가상디스크.png)

- Keyboard Layout 창으로 ***KR 없으니 US Default Enter*** 클릭한다.  
[![vSphere  키보드 레이아웃](/assets/img/post/VMware/vSphere%20%20키보드%20레이아웃.png)](/assets/img/post/VMware/vSphere%20%20키보드%20레이아웃.png)

- Root Password 설정 ***소문자,숫자,특수기호 혼합 7글자 이상*** 입력한다.  
[![vSphere root 패스워드](/assets/img/post/VMware/vSphere%20root%20패스워드.png)](/assets/img/post/VMware/vSphere%20root%20패스워드.png)

- 설치 진행을 위해 ***F11*** 키를 클릭한다.
[![vSphere 설치 진행](/assets/img/post/VMware/vSphere%20설치%20진행.png)](/assets/img/post/VMware/vSphere%20설치%20진행.png)

- 설치 완료 후 재부팅 진행 ***Enter*** 클릭한다.  
[![vSphere 설치완료](/assets/img/post/VMware/vSphere%20설치완료.png)](/assets/img/post/VMware/vSphere%20설치완료.png)

* * *

## vSphere 네트워크 설정하기:
- 네트워크 세팅 ***F2번*** 클릭한다.  
[![vSphere 네트워크 설정](/assets/img/post/VMware/vSphere%20네트워크%20설정.png)](/assets/img/post/VMware/vSphere%20네트워크%20설정.png)

- 관리자 root 패스워드 입력한다.  
[![vSphere 관리자 root 패스워드](/assets/img/post/VMware/vSphere%20관리자%20root%20패스워드.png)](/assets/img/post/VMware/vSphere%20관리자%20root%20패스워드.png)

- Network Management 선택한다.  
[![vSphere 네트워크 관리 선택](/assets/img/post/VMware/vSphere%20네트워크%20관리%20선택.png)](/assets/img/post/VMware/vSphere%20네트워크%20관리%20선택.png)

- IPV4 선택한다.  
[![vSphere 네트워크 IP 설정](/assets/img/post/VMware/vSphere%20네트워크%20IP%20설정.png)](/assets/img/post/VMware/vSphere%20네트워크%20IP%20설정.png)

- IPv4 네트워크 고정IP 지정한다.  
[![vSphere 네트워크 고정IP 설정](/assets/img/post/VMware/vSphere%20네트워크%20고정IP%20설정.png)](/assets/img/post/VMware/vSphere%20네트워크%20고정IP%20설정.png)

- DNS 구성 선택한다.  
[![DNS 구성 선택](/assets/img/post/VMware/DNS%20구성%20선택.png)](/assets/img/post/VMware/DNS%20구성%20선택.png)

- DNS 구성한다.  
[![vSphere DNS 구성](/assets/img/post/VMware/vSphere%20DNS%20구성.png)](/assets/img/post/VMware/vSphere%20DNS%20구성.png)

- 위 설정 저장을 위해 ***Y*** 를 입력한다.  
[![vSphere 설정 저장](/assets/img/post/VMware/설정%20저장.png)](/assets/img/post/VMware/설정%20저장.png)

* * *
