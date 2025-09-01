---
layout: post
title: "Gitlab과 Slack 연동하기"
date: 2023-10-26
categories: [DevOps, Gitlab]
tags: [Git, Gitlab, Slack]
image: /assets/img/post-title/gitLab-wallpaper.jpg
---


## Gitlab과 Slack 연동하기:
### Slack 설정하기:
- Slack에서 왼쪽 하단에 앱 추가를 클릭하여 ***Incoming WebHooks(수신 웹후크)***를 추가한다.
[![Slack Incoming WebHooks 추가](/assets/img/post/Gitlab/Slack%20Incoming%20WebHooks%20추가.png)](/assets/img/post/Gitlab/Slack%20Incoming%20WebHooks%20추가.png)&nbsp; 
[![Slack Webhooks 채널 추가](/assets/img/post/Gitlab/Slack%20Webhooks%20채널%20추가.png)](/assets/img/post/Gitlab/Slack%20Webhooks%20채널%20추가.png)

- 추가 후, 페이지의 나오는 ***Webhook URL***를 복사하여, GitLab을 설정한다.
[![slack gitlab webhook url](/assets/img/post/Gitlab/slack%20gitlab%20webhook%20url.png)](/assets/img/post/Gitlab/slack%20gitlab%20webhook%20url.png)

* * *

### GitLab 설정하기:
- 연동할 GitLab에서 다음과 같이 클릭한다.
[![gitlab slack 알림 설치](/assets/img/post/Gitlab/gitlab%20slack%20알림%20설치.png)](/assets/img/post/Gitlab/gitlab%20slack%20알림%20설치.png)

- Slack 설정 부분에서 복사했던 ***Webhook URL***를 입력 후 저장하여 ***Test Settings***버튼을 클릭하여 슬랙의 알림이 가는지 확인한다.
[![Gitlab Slack Notification setting URL 적용](/assets/img/post/Gitlab/Gitlab%20Slack%20Notification%20setting%20URL%20적용.png)](/assets/img/post/Gitlab/Gitlab%20Slack%20Notification%20setting%20URL%20적용.png)

* * *