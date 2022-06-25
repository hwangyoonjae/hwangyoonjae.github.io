---
title: "MAC에서 jekyll 로컬 서버 실행"
layout: post
date: 2022-06-08
image: /assets/images/Post/start.png
headerImage: false
tag:
- markdown
- components
- extra
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
brew -v
```

### 2. Ruby 설치

Homebrew로 최신버전의 Ruby를 설치한다.
```javascript
brew install ruby
```
설치한 Ruby 버전을 확인한다.
```javascript
ruby -v
```

### 3. Jekyll 설치

jekyll과 bundle을 설치한다.
```javascript
gem install jekyll bundle
```

하지만 나는 아래 그림처럼 맥북에서도 오류가 발생하여 실행을 못했다....
![텍스트](/assets/images/local/맥북&#32;jekyll&#32;설치&#32;오류.png)