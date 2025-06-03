---
layout: post
title: "Vue HTTP 통신 라이브러리 Axios란"
date: 2022-07-24
categories: Vue
tags: [Vue, HTTP, Axios, 라이브러리]
image: /assets/img/post-title/vue-wallpaper.jpg
---

## Axios란?:
- HTTP 통신 라이브러리이다.
- Promise 기반의 API 형식으로 자바스크립트 비동기 처리 방식을 사용하며, IE8 이상을 포함한 모든 최신 브라우저를 지원한다.

* * *

## 설치:
```javascript
-- npm으로 설치하는 경우
$ npm install axios

-- yarn으로 설치하는 경우
$ yarn add axios

-- CDN으로 설치하는 경우
<script src="https://unpkg.com/axios/dist/axios.min.js"></script> 
```

* * *

## Vue에서 Axios 사용해보기:
<span style="color:#FA5858; font-size:12px">※ Main.js에 아래 내용을 추가한다.</span>
```javascript
import Vue from 'vue'
import App from './App.vue'
import axios from 'axios' // import axios

Vue.prototype.$axios = axios; // prototype에 axios 추가

Vue.config.productionTip = false

new Vue({
  el: '#app',
  router,
  components: { App },
  template: '<App/>'
})
```

* * *

## Axios HTTP 요청 메서드 종류:
### GET (불러오기)
- 말 그대로 서버로부터 데이터를 가져올 때 사용한다.
- 두 번째 파라미터 config 객체에는 헤더(header), 응답 초과시간 (timeout), 인자 값(params) 등의 요청 값을 같이 넘길 수 있다.
```javascript
// HTTP GET 요청
axios.get('URL 주소'[,config]).then().catch();
```

### POST (입력하기)
- 말 그대로 서버에 데이터를 전달할 때 사용한다.
- 두 번째 파라미터 data에 생성할 데이터를 넘긴다.
```javascript
// HTTP POST 요청
axios.post('URL 주소'[, data[,config]]).then().catch();
```

### then, catch란?
- then : 비동기 통신이 성공했을 경우 **.then()**은 콜백을 인자로 받아 결과값을 처리할 수 있다.
- catch : **.catch()** 를 통해 오류를 처리한다. error 객체에서는 오류에 대한 주요 정보를 확인할 수 있다.

* * *

## Axios HTTP 요청 Config 옵션:
- **method** : method는 요청을 할 때 사용할 요청 메서드이고, method의 기본값은 get이다.
<br>
- **url** : url은 액시오스 요청에 사용될 서버의 URL을 말한다.
<br>
- **baseURL** : baseURL은 액시오스 인스턴스를 생성할 때, 인스턴스의 기본 URL 값을 정할 수 있는 속성이며, 보통 API 서버의 기본 도메인을 설정하고, 인스턴스 별로 URL을 뒤에 추가하여 사용한다.
<br>
- **headers** : 헤더를 수정해서 보내야 한다면 headers를 사용한다.
<br>
- **params** : params는 HTTP 요청에 붙일 URL 파라미터를 의미하고, params가 null이나 undefined인 경우에는 파라미터가 조합되지 않는다.
<br>
- **data** : data는 HTTP 요청 보디에 실어서 보낼 데이터를 의미하고, 주로 데이터를 조작해야 하는 PUT, POST, DELETE, PATCH 등의 메서드에서 사용한다.
<br>
- **timeout** : timeout은 HTTP 요청을 보내고 응답을 받기까지의 제한 시간을 설정하는 속성이고, 요청 시간이 지정된 값을 초과하면 에러가 발생한다.
<br>
- **responseType** : responseType은 서버로부터 어떠한 데이터 형식으로 응답받을지 지정하는 것이고, 옵션으로는 arraybuffer, 
document, json, text, stream이 가능하며, 기본 값은 json이다.

* * *