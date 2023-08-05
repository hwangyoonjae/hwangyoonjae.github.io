---
title: "[Jenkins] - Jenkins 빌드부터 배포까지 프로젝트 구축하기"
categories:
  - Jenkins
tags:
  - [Jenkins, CI, CD, Github]

toc: true
toc_sticky: true

date: 2023-07-31
last_modified_at: 2023-07-31
---

## Jenkins 빌드부터 배포까지 프로젝트 구축하기:
### Gradle 버전 세팅하기:
- ***Jenkins관리 > Tools***를 클릭한다.
[![Jenkins Tool 설정](/assets/images/Jenkins/Jenkins%20Tool%20설정.PNG)](/assets/images/Jenkins/Jenkins%20Tool%20설정.PNG)

- Gradle 버전의 맞춰 설정한다.
[![Gradle 설치](/assets/images/Jenkins/Gradle%20설치.PNG)](/assets/images/Jenkins/Gradle%20설치.PNG)
<span style="color:#FA5858; font-size:12px">※ 필자는 Gradle 7.5버전으로 설정하였다.</span>

* * *

### Gradle 프로젝트 생성하기:
- Gradle로 구성한 소스를 깃허브를 통해 빌드하기 위해 프로젝트를 생성한다.
[![Backend Project 생성](/assets/images/Jenkins/Backend%20Project%20생성.PNG)](/assets/images/Jenkins/Backend%20Project%20생성.PNG)

- Jenkins Github 연동이 필요한 경우에는 아래 게시글 참고하면 된다.
> * [Jenkins Github 연동하기](https://hwangyoonjae.github.io/jenkins/Jenkins-Jenkins-Github-%EC%97%B0%EB%8F%99%ED%95%98%EA%B8%B0/ "Jenkins Github 연동하기")

* * *

### Gradle Build 세팅하기:
- ***생성한 프로젝트 > 구성***을 클릭한다.
[![Project 구성 선택](/assets/images/Jenkins/Project%20구성%20선택.PNG)](/assets/images/Jenkins/Project%20구성%20선택.PNG)

- Build step의 Filter는 ***invoke Gradle script***를 선택한다.
[![Gradle 빌드 환경 세팅](/assets/images/Jenkins/Gradle%20빌드%20환경%20세팅.PNG)](/assets/images/Jenkins/Gradle%20빌드%20환경%20세팅.PNG)

- ***gradle*** (Global Tool Configuration에서 생성했던 걸로)를 선택한다.
[![Gradle 빌드 Step](/assets/images/Jenkins/Gradle%20빌드%20Step.PNG)](/assets/images/Jenkins/Gradle%20빌드%20Step.PNG)

* * *

### Gradle Build 진행하기:
- 지금까지 세팅한거를 바탕으로 빌드를 진행한다.
[![Gladle 빌드 시작](/assets/images/Jenkins/Gladle%20빌드%20시작.PNG)](/assets/images/Jenkins/Gladle%20빌드%20시작.PNG)

- 빌드가 성공된 것을 확인한다.
[![Gradle 빌드 성공](/assets/images/Jenkins/Gradle%20빌드%20성공.PNG)](/assets/images/Jenkins/Gradle%20빌드%20성공.PNG)

* * *

## 빌드한 파일 확인하기:
- Jenkins에서 빌드한 파일을 docker container 안에서 확인한다.
```bash
$ docker exec -it "컨테이너명" /bin/bash
$ cd /var/jenkins_home/workspace/Backend/build/libs/
$ ls -al
```
[![Gradle 빌드 파일](/assets/images/Jenkins/Gradle%20빌드%20파일.PNG)](/assets/images/Jenkins/Gradle%20빌드%20파일.PNG)

* * *