---
layout: post
title: "Gitlab, Runner CI&CD 구축"
date: 2024.07.11
categories: [DevOps, Gitlab]
tags: [Git, Gitlab, Runner, CI, CD]
image: /assets/img/post-title/gitLab-wallpaper.jpg
---

## 1. gitlab-runner config.toml 네트워크 방식 추가 :
- [runners.docker]의 network_mode = "host" 값을 추가하고, gitlab-runner 재시작합니다.
![gitlab-runner 네트워크 방식 추가](/assets/img/post/Gitlab/gitlab-runner%20네트워크%20방식%20추가.png)
```bash
$ gitlab-runner restart
```

* * *

## 2. .gitlab-ci.yml 파일 생성 :
1. 왼쪽 사이드바에서 **Code > Repository**로 이동합니다.
2. 파일 목록 위에서, 커밋할 브랜치를 선택한 후, 플러스(+) 아이콘을 클릭하고, **New file**을 선택합니다.
![gitlab-ci yaml 파일 생성](/assets/img/post/Gitlab/gitlab-ci%20yaml%20파일%20생성.png)

3. 파일 이름`.gitlab-ci.yml`을 입력하고 큰 창에서 다음 샘플 코드를 붙여 넣는다.

```yml
build-job:
  stage: build
  script:
    - echo "Hello, $GITLAB_USER_LOGIN!"
test-job1:
  stage: test
  script:
    - echo "This job tests something"
test-job2:
  stage: test
  script:
    - echo "This job tests something, but takes more time than test-job1."
    - echo "After the echo commands complete, it runs the sleep command for 20 seconds"
    - echo "which simulates a test that runs 20 seconds longer than test-job1"
    - sleep 20
deploy-prod:
  stage: deploy
  script:
    - echo "This job deploys something from the $CI_COMMIT_BRANCH branch."
  environment: production
```

4. **Commit changes** 버튼을 클릭합니다.
- 커밋되면 파이프라인이 시작되고, `.gitlab-ci.yml` 파일에 정의한 job을 실행합니다.
![gitlab-runner ci 정의](/assets/img/post/Gitlab/gitlab-runner%20ci%20정의.png)

* * *

## 파이프라인과 job 상태 보기 :
- 변경 사항을 커밋하면 파이프라인이 시작되고, **Build > Pipelines**로 이동합니다.
- 세 단계(Stage)의 파이프라인이 표시되어야 합니다.
![gitlab-runner 파이프라인 표시](/assets/img/post/Gitlab/gitlab-runner%20파이프라인%20표시.png)

- Job의 세부 정보를 보려면, job 이름을 클릭합니다.
![gitlab-runner job 세부 정보](/assets/img/post/Gitlab/gitlab-runner%20job%20세부%20정보.png)

* * *