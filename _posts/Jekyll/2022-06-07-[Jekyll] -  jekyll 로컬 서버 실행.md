---
layout: post
title: "Window에서 jekyll 로컬 서버 실행"
date: 2022-06-07
categories: [Blog, Jekyll]
tags: [Window, Jekyll]
image: /assets/img/post-title/jekyll-wallpaper.jpg
---

## 1. Window에서 jekyll 로컬 서버 실행하기:
### 1.1 루비 설치하기 :
> * [루비 설치파일 다운받기](https://rubyinstaller.org/downloads/ "지킬테마")

![ruby 설치프로그램](/assets/img/post/local/ruby%20%EC%84%A4%EC%B9%98%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%A8.PNG)

```bash
# 아래와 같이 화살표 표시된 추천버전을 다운받는다.
=> Ruby+devkit 3.x.x-x(x64)
```
> 버전은 계속 업데이트 되어 다를 수 있습니다.
{: .prompt-warning}

<br><br>
다운받은 Installer를 실행하면 CMD창이 뜬다.
<br>
1번과 3번을 선택하라는 창이 뜨면 **1번을(기본설치)** 누르고 Enter를 누른다.
<br>
설치가 완료되고 Enter를 한번더 누르면 CMD창이 종료된다.
<br>

![ruby 설치화면(1)](/assets/img/post/local/ruby%20%EC%84%A4%EC%B9%98%ED%99%94%EB%A9%B4(1).png)
![ruby 설치화면(2)](/assets/img/post/local/ruby%20%EC%84%A4%EC%B9%98%ED%99%94%EB%A9%B4(2).png)
* * *

### 1.2 bundler 설치하기 :
- 방금 루비 설치한 CMD창 **"Start Command Prompt with Ruby"** 을 열어 깃허브 블로그 디렉토리로 이동하여 아래의 명령어를 입력합니다.

```bash
$ bundler update
$ bundler install
```

> ruby 호환성때문에 bundler 업데이트 한번 진행 후 설치하였습니다.
{: .prompt-tip}

* * *

### 1.3 서버 실행하기 :

- 명령어를 통해 서버를 로컬서버를 실행합니다. (서버를 종료시키려면 ctrl + c를 누르면 된다.)

```bash
$ bundle exec jekyll serve 
```

![jekyll 서버 실행](/assets/img/post/local/jekyll%20%EC%84%9C%EB%B2%84%20%EC%8B%A4%ED%96%89.PNG)

* * *

- 하지만 필자는 아래 그림처럼 오류가 발생하여 실행을 못했습니다....

![jekyll 서버 실행안되는 경우](/assets/img/post/local/jekyll%20%EC%84%9C%EB%B2%84%20%EC%8B%A4%ED%96%89%EC%95%88%EB%90%98%EB%8A%94%20%EA%B2%BD%EC%9A%B0.PNG)

* 조치방법
    + [(클릭)필요한 라이브러리 다운받기 위해 사이트 이동합니다.](https://curl.se/windows/ "라이브러리")
    + 본인 PC 사양에 맞춰 다운로드 진행합니다.
    ![라이브러리 다운클릭](/assets/img/post/local/%EB%9D%BC%EC%9D%B4%EB%B8%8C%EB%9F%AC%EB%A6%AC%20%EB%8B%A4%EC%9A%B4%ED%81%B4%EB%A6%AD.PNG)
    + 압축 해제 후 **libcurl-x64.dll**을 **Ruby31-x64\bin**에 복사합니다.
    + 파일명 **libcurl-x64.dll**에서 **libcurl.dll**로 변경합니다.
    + 서버 재실행합니다.

* * *

### 1.4 Window 로컬 실행하기 :
- **http://127.0.0.1:4000/**로 접속하면 아래와 같이 접속된다.
  
![jekyll 로컬 실행화면](/assets/img/post/local/jekyll%20%EB%A1%9C%EC%BB%AC%20%EC%8B%A4%ED%96%89%ED%99%94%EB%A9%B4.PNG)

* * *