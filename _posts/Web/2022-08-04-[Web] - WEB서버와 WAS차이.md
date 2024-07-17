---
layout: post
title: "[Web] - WEB Server와 WAS 차이"
date: 2022-08-04
categories: Web
tags: [Web, Was]
image: /assets/featured-posts/code1.jpg
---

## WEB Server와 WAS에 대해 알고 싶었던 계기 :
- 서버점검이나 구축을 진행하면서 서버 OS 정보를 정리하는 경우가 있다.<br>
서버에서 사용하는 플랫폼에 대해서는 버전 확인이나 설치 여부를 확인할 수 있지만, WEB과 WAS를 구별해서 정리를 해야하는 경우 헷갈리는게 너무 많았다.<br>
그래서 업무에 있어 한번쯤은 이해를 해야할 거 같아서 블로그를 작성하게 되었다.

* * *

## WEB Server와 WAS(Web Application Server)란? :

### WEB서버(Web Server)란? :
- 클라이언트(사용자)가 웹 브라우저에서 어떠한 페이지 요청을 하면 웹 서버에서 그 요청을 받아 **정적 컨텐츠**를 제공하는 서버

```javascript
정적 컨텐츠는?
- 단순 HTML 문서, CSS, javascript, 이미지, 파일 등 즉시 응답가능한 컨텐츠
```

```javascript
- 클라이언트의 입장: 웹 서버에게 주소(url)을 가지고 통신 규칙(http)에 맞게 요청하면, 알맞은 내용(html)을 응답 받는다.

- 서버 입장: 클라이언트의 요청을 기다리고, 웹 요청(http)에 대한 데이터를 만들어서 응답, 이때 데이터는 웹에서 처리할 수 있는 html, css, 이미지 등 정적인 데이터로 한정한다.
```

### WEB Server 기능 :
- HTTP 프로토콜을 기반으로 하여 클라이언트(웹 브라우저 또는 웹 크롤러)의 요청을 서비스 하는 기능을 담당한다.<br>
<span style="color:#FA5858; font-size:12px">※ 대표적인 웹 서버 : Apache</span>

[![텍스트](/assets/images/Linux/%EC%9B%B9%EC%84%9C%EB%B2%84%20%EC%9A%94%EC%B2%AD%EB%B0%A9%EC%8B%9D.PNG)](/assets/images/Linux/%EC%9B%B9%EC%84%9C%EB%B2%84%20%EC%9A%94%EC%B2%AD%EB%B0%A9%EC%8B%9D.PNG)<br>
△ 웹 서버 사용자 요청 처리 과정
* * *

### WAS(Web Application Server)란? :
- 인터넷 상에서 HTTP를 통해 사용자 컴퓨터나 장치에 애플리케이션을 수행해 주는 미들웨어(소프트웨어 엔진)이며,<br>
  웹 애플리케이션 서버는 동적 서버 콘텐츠를 수행하는 것으로 일반적인 웹 서버와 구별이 되며, 주로 데이터베이스 서버와 같이 수행한다.
- 웹 서버 + 웹 컨테이너를 합친 형태다.

```javascript
- 컨테이너 : jsp, servlet을 실행시킬 수 있는 소프트웨어, 자바 계열에선 웹 애플리케이션을 컨테이너라고 부른다.
- 웹 애플리케이션 컨테이너 : 웹 애플리케이션이 배포되는 공간이다.
```

### WAS(Web Application Server) 기능 :
- 프로그램 실행 환경과 데이터베이스 접속 기능을 제공하고, 여러 개의 트랜잭션을 관리한다.<br>
<span style="color:#FA5858; font-size:12px">※ 대표적인 웹 서버 : Tomcat</span>
[![텍스트](/assets/images/Linux/WAS%20%EC%9A%94%EC%B2%AD%EB%B0%A9%EC%8B%9D.PNG)](/assets/images/Linux/WAS%20%EC%9A%94%EC%B2%AD%EB%B0%A9%EC%8B%9D.PNG)<br>
△ WAS 사용자 요청 처리 과정

* * *