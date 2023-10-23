---
title: "[Gitlab] - Gitlab 연동 오류 해결하기"
categories:
  - Gitlab
tags:
  - [Git, Gitlab]

toc: true
toc_sticky: true

date: 2023-10-23
last_modified_at: 2023-10-23
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
[![url is blocked 에러 해결방법](/assets/images/Gitlab/url%20is%20blocked%20에러%20해결방법.png)]

* * *