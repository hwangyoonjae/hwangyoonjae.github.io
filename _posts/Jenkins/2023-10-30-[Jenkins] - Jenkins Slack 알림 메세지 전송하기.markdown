---
title: "[Jenkins] - Jenkins Slack 알림 메세지 전송하기"
categories:
  - Jenkins
tags:
  - [Jenkins, CI, CD, Slack]

toc: true
toc_sticky: true

date: 2023-10-30
last_modified_at: 2023-10-30
---

## Jenkins Slack 알림 메세지 전송하기:
- Slack에서 왼쪽 하단에 앱 추가를 클릭하여 ***Jenkins CI***를 추가한다.
[![Slack Jenkins 추가](/assets/images/Jenkins/Slack%20Jenkins%20추가.png)](/assets/images/Jenkins/Slack%20Jenkins%20추가.png)

* * *

### Slack Plugin 설치하기:
- ***Jenkins관리 -> 플러그인 -> 설치가능한 플러그인***에 들어가 Slack 플러그인을 설치한다.
[![Slack plugin 설치](/assets/images/Jenkins/Slack%20plugin%20설치.png)](/assets/images/Jenkins/Slack%20plugin%20설치.png)

- 추가 후, 페이지의 나오는 단계 중 ***3단계***의 적힌 ***팀 하위 도메인과 통합 토큰 자격 증명 ID***를 참고하여 Jenkins 연동할 때 작성한다.
[![Jenkins Slack 계정 연동 정보](/assets/images/Jenkins/Jenkins%20Slack%20계정%20연동%20정보.png)](/assets/images/Jenkins/Jenkins%20Slack%20계정%20연동%20정보.png)

* * *

### Jenkins Slack 연동하기:
- ***Jenkins관리 -> 시스템***에 들어가 Workspace와 계정 입력 후 ***Test Connection***을 클릭하여 연결 테스트를 진행한다.
[![jenkins slack 연동](/assets/images/Jenkins/jenkins%20slack%20연동.png)](/assets/images/Jenkins/jenkins%20slack%20연동.png)

- 계정은 ***Secret text***를 선택하여 Slack에서 발급받은 토큰을 ***Secret***부분에 추가하여 계정 등록 후 선택한다.
[![Jenkins Slack 계정](/assets/images/Jenkins/Jenkins%20Slack%20계정.png)](/assets/images/Jenkins/Jenkins%20Slack%20계정.png)

* * *