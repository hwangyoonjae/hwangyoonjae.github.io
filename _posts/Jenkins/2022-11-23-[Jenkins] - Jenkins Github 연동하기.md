---
layout: post
title: "Jenkins Github 연동하기"
date: 2022-11-23
categories: Jenkins
tags: [Jenkins, Github]
image: /assets/img/post-title/jenkins-wallpaper.jpg
---

## Jenkins Github 연동하기:
### Item 생성하기:
- Jenkins 메인페이지에서 **새로운 Item** 클릭하여 생성한다.
[![텍스트](/assets/img/post/Jenkins/Item%20%EC%83%9D%EC%84%B1%ED%99%94%EB%A9%B4.PNG)](/assets/img/post/Jenkins/Item%20%EC%83%9D%EC%84%B1%ED%99%94%EB%A9%B4.PNG)

* * *

### Githuib Repository 연결하기:
- 소스 코드 관리 밑에 부분에 Github 연결정보를 입력한다.
[![텍스트](/assets/img/post/Jenkins/git%20%EC%97%B0%EA%B2%B0%ED%99%94%EB%A9%B4.PNG)](/assets/img/post/Jenkins/git%20%EC%97%B0%EA%B2%B0%ED%99%94%EB%A9%B4.PNG)

* * *

### 프로젝트 생성되었는지 확인하기:
- Item 생성 후 저장 버튼 클릭 시 아래 그림과 같이 나온다.
[![텍스트](/assets/img/post/Jenkins/%EC%A0%A0%ED%82%A8%EC%8A%A4%20%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8%20%ED%99%94%EB%A9%B4.PNG)](/assets/img/post/Jenkins/%EC%A0%A0%ED%82%A8%EC%8A%A4%20%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8%20%ED%99%94%EB%A9%B4.PNG)

* * *

## 작업 간 에러사항:
### 에러 1:
- 에러로그
  ```html
  Failed to connect to repository: Command "git.exe ls-remote -h -- git@..." returned status code 128: stdout: stderr: Host key verification failed. fatal: Could not read from remote repository. Please make sure you have the correct access rights and the repository exists.
  ```
- 조치 방법 : Repository에 대한 권한이 없어서 발생한 증상으로 Credentials Add 버튼 클릭하여 계정을 추가한다.
[![텍스트](/assets/img/post/jenkins/Credentials%20Add%20%EA%B3%84%EC%A0%95%20%EC%B6%94%EA%B0%80.PNG)](/assets/img/post/jenkins/Credentials%20Add%20%EA%B3%84%EC%A0%95%20%EC%B6%94%EA%B0%80.PNG)

### 에러 2:
- 에러로그
  ```html
  Couldn't find any revision to build. Verify the repository and branch configuration for this job.
  ```
- 조치방법 : */master -> */main 으로 변경 진행
[![텍스트](/assets/img/post/Jenkins/github%20%EB%B8%8C%EB%9E%9C%EC%B9%98%20%EC%97%90%EB%9F%AC.PNG)](/assets/img/post/Jenkins/github%20%EB%B8%8C%EB%9E%9C%EC%B9%98%20%EC%97%90%EB%9F%AC.PNG)

