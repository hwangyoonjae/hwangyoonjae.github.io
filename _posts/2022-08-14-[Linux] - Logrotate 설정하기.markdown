---
title: "[Linux] - Logrotate 설정하기"
layout: post
date: 2022-08-15
image: /assets/images/Post/log.png
headerImage: true
tag:
- Linux
- Log
category: blog
author: hwangyoonjae
description: Markdown summary with different options
---

## Logrotate는 무엇인가?:
- Linux에서 log를 저장하며 관리 할때 특정 log 파일이 한 파일로 계속해서, 크기가 커지며 저장되는 걸 분산시켜줄때 사용한다.

* * *

### Logrotate 실행 순서
[![텍스트](/assets/images/Linux/Logrotate%20%EC%8B%A4%ED%96%89%EC%88%9C%EC%84%9C.PNG)](/assets/images/Linux/Logrotate%20%EC%8B%A4%ED%96%89%EC%88%9C%EC%84%9C.PNG)<br>
<span style="color:#FA5858; font-size:12px">※ Logrotate는 위 사진과 같은 순서대로 동작한다.</span>

* * *

### Logrotate 파일구조
```javascript
- 데몬 프로그램 : /usr/sbin/logrotate 
- Logrotate 데몬 설정파일 : /etc/logrotate.conf
- Logrotate를 프로세스 설정파일 : /etc/logrotate.d/
- Logrotate 작업내역 로그 : /etc/cron.daily/logrotate
```

* * *