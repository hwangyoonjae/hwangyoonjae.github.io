---
layout: post
title: "[Jenkins] - Jenkins 설치하기"
date: 2022-09-09
categories: Jenkins
tags: [Jenkins, Java, CI]
image: /assets/post/jenkins-wallpaper.jpg
---

## 젠킨스(Jenkins)란?:
- Java로 제작된 오픈소스 CI(Continous Integration) 툴이다.
- Build나 Deloy, Test 프로세스를 상시적으로 실시하는 자동화 서비스이다.

#### CI(Continous Integration)란?:
- 팀의 구성원들이 작업한 내용을 정기적으로 통합하는 것이다.

* * *

## jdk 설치 및 환경변수 설정하기:
### jdk 설치하기:
- jenkins는 java로 작성된 프로그램으로 jdk8 또는 jdk11을 이용하여 동작하기에 jdk를 설치해야 한다.
  ```bash
  # java-1.8.0-openjdk-devel.x86_64 설치
  $ yum install java-1.8.0-openjdk-devel.x86_64

  # java-11-openjdk-devel.x86_64 설치
  $ yum install java-11-openjdk-devel.x86_64
  ```

### jdk 설치 확인:
```bash
$ java -version
```
[![텍스트](/assets/images/Jenkins/%EC%9E%90%EB%B0%94%20%EC%A0%95%EC%83%81%20%EC%84%A4%EC%B9%98%20%ED%99%95%EC%9D%B8.PNG)](/assets/images/Jenkins/%EC%9E%90%EB%B0%94%20%EC%A0%95%EC%83%81%20%EC%84%A4%EC%B9%98%20%ED%99%95%EC%9D%B8.PNG)<br>
<span style="color:#FA5858; font-size:12px">※ 필자는 jdk8 설치 시 젠킨스 실행이 안되어 jdk11을 설치하여 진행하였다.</span>

### JAVA_HOME 설정:
- 자바 설치가 완료 되었다면 설치 경로는 /usr/lib/jvm/[자바버전]의 경로를 확인한다.
  ```bash
  # javac 명령어의 위치 확인
  $ which javac

  # JAVA_HOME 경로 얻기
  $ readlink -f [javac 명령어 위치]
  ```
  [![텍스트](/assets/images/Jenkins/%EC%9E%90%EB%B0%94%20%ED%99%98%EA%B2%BD%EB%B3%80%EC%88%98%20%EC%84%A4%EC%A0%95.PNG)](/assets/images/Jenkins/%EC%9E%90%EB%B0%94%20%ED%99%98%EA%B2%BD%EB%B3%80%EC%88%98%20%EC%84%A4%EC%A0%95.PNG)

- JAVA_HOME 환경변수로 등록한다.
  ```bash
  $ vi /etc/profile

  # 맨 하단에 내용 추기
  $ export JAVA_HOME=[위에서 얻은 경로에서 /bin/javac 제외한 부분 입력]
  ```
  [![텍스트](/assets/images/Jenkins/%EC%9E%90%EB%B0%94%20%ED%99%98%EA%B2%BD%EB%B3%80%EC%88%98%20%EA%B2%BD%EB%A1%9C%20%EB%93%B1%EB%A1%9D.PNG)](/assets/images/Jenkins/%EC%9E%90%EB%B0%94%20%ED%99%98%EA%B2%BD%EB%B3%80%EC%88%98%20%EA%B2%BD%EB%A1%9C%20%EB%93%B1%EB%A1%9D.PNG)
  
  ```bash
  $ source /etc/profile

  $ echo $JAVA_HOME
  ```
  [![텍스트](/assets/images/Jenkins/%EC%9E%90%EB%B0%94%20%ED%99%98%EA%B2%BD%EB%B3%80%EC%88%98%20%EA%B2%BD%EB%A1%9C%20%ED%99%95%EC%9D%B8.PNG)](/assets/images/Jenkins/%EC%9E%90%EB%B0%94%20%ED%99%98%EA%B2%BD%EB%B3%80%EC%88%98%20%EA%B2%BD%EB%A1%9C%20%ED%99%95%EC%9D%B8.PNG)

### 기존 자바 변경 방법:
- 혹시나 자바 버전이 다를 경우 아래와 같이 자바 버전을 변경한다.
```bash
$ update-alternatives --config java
```
[![텍스트](/assets/images/Jenkins/%EC%9E%90%EB%B0%94%20%EB%B2%84%EC%A0%84%20%EB%B3%80%EA%B2%BD%20%EB%B0%A9%EB%B2%95.PNG)](/assets/images/Jenkins/%EC%9E%90%EB%B0%94%20%EB%B2%84%EC%A0%84%20%EB%B3%80%EA%B2%BD%20%EB%B0%A9%EB%B2%95.PNG)
<span style="color:#FA5858; font-size:12px">※ 필자는 jdk11을 설치하였기에 3번을 선택하여 진행했다.</span>

* * *

## 젠킨스(Jenkins) 설치하기:
- yum 레포지터리에 젠킨스 레드햇 안정화 버전 레포지터리를 추가한다.
  ```bash
  $ wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
  ```

- rpm에 젠킨스를 추가한다.
  ```bash
  $ rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
  ```

- 젠킨스를 설치한다.
  ```bash
  # epel 설치
  $ yum install epel-release

  # 젠킨스 설치
  $ yum install jenkins
  ```
  <span style="color:#FA5858; font-size:12px">※ epel이란? : Extra Packages for Enterprise Linux 의 약자로 기업용 리눅스 환경을 위한 추가 패키지를 의미한다.</span>

- 젠킨스 설치가 잘 되었는지 확인한다.
  ```bash
  $ rpm -qa | grep jenkins
  ```

- vi 편집기를 이용하여 포트를 변경하고, 해당 포트에 방화벽을 설정한다.
  ```bash
  # 젠킨스 포트 변경
  $ vi /usr/lib/systemd/system/jenkins.service
  ```
  [![텍스트](/assets/images/Jenkins/%EC%A0%A0%ED%82%A8%EC%8A%A4%20%ED%8F%AC%ED%8A%B8%20%EB%B3%80%EA%B2%BD%20%ED%99%94%EB%A9%B4.PNG)](/assets/images/Jenkins/%EC%A0%A0%ED%82%A8%EC%8A%A4%20%ED%8F%AC%ED%8A%B8%20%EB%B3%80%EA%B2%BD%20%ED%99%94%EB%A9%B4.PNG)
  
  ```bash
  # 포트 방화벽 설정
  $ firewall-cmd --permanent --zone=public --add-port=9090/tcp
  $ firewall-cmd --reload
  ```

* * *

## 젠킨스(Jenkins) 사용하기:
- 젠킨스를 시작하기 위해 다음 명령어를 입력한다.
  ```bash
  $ systemctl start jenkins
  또는
  $ service jenkins start
  ```

- 호스트에서 브라우저를 열고 http://IP주소:포트번호를 입력해 접속하여 화면에 표시된 경로를 통해 초기 비밀번호를 확인한다.
  [![텍스트](/assets/images/Jenkins/%EC%A0%A0%ED%82%A8%EC%8A%A4%20%EB%B9%84%EB%B0%80%EB%B2%88%ED%98%B8%20%EC%9E%85%EB%A0%A5%20%ED%99%94%EB%A9%B4.PNG)](/assets/images/Jenkins/%EC%A0%A0%ED%82%A8%EC%8A%A4%20%EB%B9%84%EB%B0%80%EB%B2%88%ED%98%B8%20%EC%9E%85%EB%A0%A5%20%ED%99%94%EB%A9%B4.PNG)

- 아래 명령어를 입력하여 초기 비밀번호를 확인 후 입력한다.
  ```bash
  $ cat /var/lib/jenkins/secrets/initialAdminPassword
  ```
  [![텍스트](/assets/images/Jenkins/%EC%B4%88%EA%B8%B0%20%EB%B9%84%EB%B0%80%EB%B2%88%ED%98%B8%20%ED%99%95%EC%9D%B8.PNG)](/assets/images/Jenkins/%EC%B4%88%EA%B8%B0%20%EB%B9%84%EB%B0%80%EB%B2%88%ED%98%B8%20%ED%99%95%EC%9D%B8.PNG)

- Install suggested plugins를 클릭해 기초 플러그인을 설치한다.
  [![텍스트](/assets/images/Jenkins/%EC%A0%A0%ED%82%A8%EC%8A%A4%20%ED%94%8C%EB%9F%AC%EA%B7%B8%EC%9D%B8%20%EC%84%A4%EC%B9%98.PNG)](/assets/images/Jenkins/%EC%A0%A0%ED%82%A8%EC%8A%A4%20%ED%94%8C%EB%9F%AC%EA%B7%B8%EC%9D%B8%20%EC%84%A4%EC%B9%98.PNG)

- 계정 정보를 입력하고 Save and Continue를 누른다.
  [![텍스트](/assets/images/Jenkins/%EA%B3%84%EC%A0%95%EC%A0%95%EB%B3%B4%20%EC%9E%85%EB%A0%A5%20%ED%99%94%EB%A9%B4.PNG)](/assets/images/Jenkins/%EA%B3%84%EC%A0%95%EC%A0%95%EB%B3%B4%20%EC%9E%85%EB%A0%A5%20%ED%99%94%EB%A9%B4.PNG)

- URL을 입력하고 Save and Finish를 누른다.
  [![텍스트](/assets/images/Jenkins/%EC%84%9C%EB%B2%84%20%EC%A3%BC%EC%86%8C%20%EC%A7%80%EC%A0%95.PNG)](/assets/images/Jenkins/%EC%84%9C%EB%B2%84%20%EC%A3%BC%EC%86%8C%20%EC%A7%80%EC%A0%95.PNG)

- 그다음 Start using Jenkins를 누른다.
  [![텍스트](/assets/images/Jenkins/%EC%A0%A0%ED%82%A8%EC%8A%A4%20%EC%8B%9C%EC%9E%91%20%ED%99%94%EB%A9%B4.PNG)](/assets/images/Jenkins/%EC%A0%A0%ED%82%A8%EC%8A%A4%20%EC%8B%9C%EC%9E%91%20%ED%99%94%EB%A9%B4.PNG)

- 다음과 같은 화면이 나타나면 기본 설정이 완료된거다.
  [![텍스트](/assets/images/Jenkins/%EC%A0%A0%ED%82%A8%EC%8A%A4%20%EB%A9%94%EC%9D%B8%ED%99%94%EB%A9%B4.PNG)](/assets/images/Jenkins/%EC%A0%A0%ED%82%A8%EC%8A%A4%20%EB%A9%94%EC%9D%B8%ED%99%94%EB%A9%B4.PNG)

* * *