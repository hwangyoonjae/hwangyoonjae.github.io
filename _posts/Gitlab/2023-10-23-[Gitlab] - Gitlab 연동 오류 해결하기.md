---
layout: post
title: "[Gitlab] - Gitlab 연동 오류 해결하기"
date: 2023-10-23
categories: Gitlab
tags: [Git, Gitlab]
image: /assets/img/post-title/gitLab-wallpaper.jpg
---

## GitLab 연동 오류 해결하기:
- GitLab 구성 과정에서 오류 발생에 대한 조치방안을 작성하였다.

* * *

### Url is Blocked 오류 해결하기:
- GitLab을 이용해서 Webhooks Push 이벤트 등록할 때, 아래와 같은 오류 메시지가 발생한다.
```html
Url is blocked: Requests to the local network are not allowed
```

- 아래 그림과 같이 조치하면된다.
[![url is blocked 에러 해결방법](/assets/img/post/Gitlab/url%20is%20blocked%20에러%20해결방법.png)](/assets/img/post/Gitlab/url%20is%20blocked%20에러%20해결방법.png)

* * *

### Hook executed successfully but returned HTTP 403 오류 해결하기:
- GitLab에서 Webhook 설정 후, Test 진행 시 아래와 같은 오류 메시지가 발생한다.
```html
Hook executed successfully but returned HTTP 403 <html> <head> <meta http-equiv="Content-Type" content="text/html;charset=ISO-8859-1"/> <title>Error 403 No valid crumb was included in the request</title> </head> <body><h2>HTTP ERROR 403 No valid crumb was included in the request</h2>
```

- GitLab에서 Webhook 설정 시 Jenkins URL 넣는 부분에서 경로 중 ***/job/ -> /project***로 변경하여 저장 후 push event 테스트 진행하면된다. 
[![HTTP 403 에러 해결방법](/assets/img/post/Gitlab/HTTP%20403%20에러%20해결방법.png)](/assets/img/post/Gitlab/HTTP%20403%20에러%20해결방법.png)

* * *