---
layout: post
title: "Github 블로그 만들기"
date: 2022-06-04
categories: [Blog, Jekyll]
tags: [markdown, components, extra]
image: /assets/img/post-title/jekyll-wallpaper.jpg
---

## 1. 계기 :
공부하면서 코딩을 하다 보면, 그 순간에는 이해가 되지만, 지나고 보면 까먹는 경우가 많다 보니 정리를 하기로 했다. <br>
원래는 PPT로 모르거나 헷갈리거나 기본 개념 부분들을 정리하려고 했지만, 처음에만 조금씩 정리했지 안하는 경우가 많았다. <br>
코딩에 대해서는 회사업무나 개인 사정으로 늦더라도 커밋을 항상 하면서 진행했는데...
요새 회사업무에 출장가고, 학교 시험이나 과제도 하다보니깐 몸이 쉬고 싶다고해야하나...?? <br>
알아야 할 부분들이 많은데 커밋은 해야 하고, 모르는 게 많으니 커밋할게 없었다. <br>
그래서 나는 깃허브 블로그를 통해서 내가 직접 만들고, 이해한 부분들에 대해서 블로그에 정리 하기로 시작했다.

* * *

## 1. 블로그 생성:
### 1.1 깃허브 블로그 사용할 레퍼지토리를 생성하기 :
![레퍼지토리생성](/assets/img/post/Github/%EB%A0%88%ED%8D%BC%EC%A7%80%ED%86%A0%EB%A6%AC%EC%83%9D%EC%84%B1.PNG)
![레퍼지토리이름정하기](/assets/img/post/Github/%EB%A0%88%ED%8D%BC%EC%A7%80%ED%86%A0%EB%A6%AC%EC%9D%B4%EB%A6%84%EC%A0%95%ED%95%98%EA%B8%B0.PNG)

> 반드시 레퍼지토리명은 위 그림과 같이 **username.github.io** 이런식으로 만들어주세요.
{: .prompt-info}

* * *

### 1.2 생성한 레퍼지토리 클론하기 : 
- 주소를 복사하고 터미널을 열어서 clone하고 싶은 경로로 이동한 다음 아래 명령어로 clone을 시작한다.

![클론하기](/assets/img/post/Github/%ED%81%B4%EB%A1%A0%ED%95%98%EA%B8%B0.PNG)

```bash
git clone "레퍼지토리 복사한 주소"
```

* * *

### 1.3 폴더 열기 :
- 복사 완료 후, 정상적으로 clone 되었는지 확인한다.

![텍스트](/assets/img/post/Github/%ED%81%B4%EB%A1%A0%ED%99%95%EC%9D%B8.PNG)

* * *

## 2. 테마 적용하기 :
### 2.1 지킬 테마 사용하기 :
- 블로그 생성을 직접 만들어서 해도되지만, 지킬이라는 서비스 플랫폼을 사용하여 본인이 적용하고 싶은 테마를 선택한다.

> * [지킬사이트 바로가기](http://jekyllthemes.org/ "지킬테마")

* * *

### 2.2 지킬테마 다운로드 :
- 본인이 선택한 지킬 테마를 다운로드 받는다.

![지킬테마 다운로드](/assets/img/post/Github/%EC%A7%80%ED%82%AC%ED%85%8C%EB%A7%88%20%EB%8B%A4%EC%9A%B4.PNG)

* * *

### 2.3 내 github.io 폴더에 붙혀넣기 :
- 다운받은 지킬파일들을 내 github.io 폴더에 붙혀넣기

![지킬테마적용](/assets/img/post/Github/%EC%A7%80%ED%82%AC%ED%85%8C%EB%A7%88%EC%A0%81%EC%9A%A9.png)

* * *

### 2.4 깃허브 블로그 접속하기 :
- 그렇게 완성한 나의 블로그이다. <br>이제 구조는 다 만들어졌으므로 템플릿 안 포스트 폴더에 블로그 글을 올리고 commit하면 자동으로 업데이트된다.

> * [완성된 블로그 접속하기](https://hwangyoonjae.github.io/ "완성된 블로그 접속하기")

[![텍스트](/assets/img/post/Github/%EB%B8%94%EB%A1%9C%EA%B7%B8%EC%99%84%EC%84%B1.PNG)](/assets/img/post/Github/%EB%B8%94%EB%A1%9C%EA%B7%B8%EC%99%84%EC%84%B1.PNG)

> 현재 블로그는 변경되어 위 그림하고는 다릅니다.
{: .prompt-warning}

* * *