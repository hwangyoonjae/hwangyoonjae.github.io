---
title: "[Jekyll] - MAC에서 jekyll 로컬 서버 실행"
layout: post
date: 2022-06-08
image: /assets/images/Post/blogging.png
headerImage: true
tag:
- MAC
- Jekyll
category: blog
author: hwangyoonjae
description: Markdown summary with different options
---

## MAC에서 jekyll 로컬 서버 실행:
### 1. Homebrew 설치
Jekyll은 Ruby라는 개발언어를 기반으로 만들어졌기 때문에 Ruby를 설치해야한다.
<br>
맥에는 시스템 상에 Ruby가 설치되어 있지만 보안상의 문제로 접근이 막혀있으며 sudo를 사용해 강제로 접근하는 것은 권장하지 않아 별도의 Ruby를 설치하기 위해 맥용 패키지 관리자인 Homebrew를 설치한다.
<br>
터미널에 아래 명령어를 입력한다.
```javascript
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```
설치가 정상적으로 되었는지 확인한다.
```javascript
$ brew -v
```

* * *

### 2. Ruby 설치

Homebrew로 최신버전의 Ruby를 설치한다.
```javascript
$ brew install ruby
```
설치한 Ruby 버전을 확인한다.
```javascript
$ ruby -v
```

* * *

### 3. Jekyll 설치

Ruby의 패키지 매니저인 gem을 통해 jekyll과 bundle을 설치한다.
```javascript
$ gem install jekyll bundle
```

하지만 나는 아래 그림처럼 맥북에서도 오류가 발생하여 실행을 못했다....
![텍스트](/assets/images/local/맥북&#32;jekyll&#32;설치&#32;오류.png)

<span style="color:#FA5858; font-size:12px">* 오류 발생 원인은 아래와 같다.</span>
```
시스템 ruby를 이용하고 있기 때문에 권한이 없어 gem 설치가 안된 것이였다.
sudo를 통해 root 권한으로 실행하면 설치가 가능하지만, 보안상 이유로 권장하지 않는 설치법이므로, rbenv를 통해서 문제를 해결해보겠다.
```

* 조치방법
    + 먼저 brew를 통해 rbenv 를 설치한다.
      ```javascript
      $ brew update
      $ brew install rbenv ruby-build
      ```
    + **rbenv**가 정상적으로 설치되었는지 확인한다.
      ```javascript
      $ rbenv versions
      ```
      <span style="color:#FA5858; font-size:10px">* 아래 그림처럼 버전을 확인할 수 있다.</span>
      ![텍스트](/assets/images/local/rbenv&#32;정상&#32;설치&#32;확인.png)
    + rbenv로 관리되는 Ruby를 설치한다.
      ```javascript
      $ rbenv install -l
      ```
      ![텍스트](/assets/images/local/설치할&#32;수&#32;Ruby&#32;버전.png)
      ```javascript
      $ rbenv install 2.6.10
      ```
    + 아래와 같이 로그가 보이면서 설치가 완료된다.
      [![텍스트](/assets/images/local/설치할&#32;수&#32;Ruby&#32;버전&#32;설치&#32;로그.png)](/assets/images/local/설치할&#32;수&#32;Ruby&#32;버전&#32;설치&#32;로그.png)
    + rbenv로 글로벌 버전을 2.6.10로 변경한다.
      ```javascript
      $ rbenv versions
      ```
    + 마지막으로 rbenv PATH를 추가하기 위해 본인의 쉘 설정 파일 (..zshrc, .bashrc) 을 열어 다음의 코드를 추가한다.
      ```javascript
      $ vim ~/.zshrc
      ```
      ```javascript
      [[ -d ~/.rbenv  ]] && \
      export PATH=${HOME}/.rbenv/bin:${PATH} && \
      eval "$(rbenv init -)"
      ```
    + 코드를 추가하면 source로 코드를 적용한다.
    + 그리고 bundler 설치 후, jekyll과 bundle을 설치한다.
      ```javascript
      $ gem install bundler
      $ gem install jekyll bundle
      ```
    + MAC에서 블로그 실행해본다.
      ```javascript
      $ bundle exec Jekyll serve
      ```

* * *

### 4. MAC 로컬 실행 
**http://127.0.0.1:4000/** 로 접속하면 완료!
![텍스트](/assets/images/local/jekyll%20%EB%A1%9C%EC%BB%AC%20%EC%8B%A4%ED%96%89%ED%99%94%EB%A9%B4.PNG)

* * *