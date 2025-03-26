---
layout: post
title: "[Gitlab] - Gitlab, Runner Pipeline 구축"
date: 2024.07.02
categories: Gitlab
tags: [Git, Gitlab, Runner, Pipeline]
image: /assets/img/post-title/gitLab-wallpaper.jpg
---

## gitlab-runner config.toml 네트워크 방식 추가 :
- [runners.docker]의 network_mode = "host" 값을 추가한다.
[![gitlab-runner config.toml 내용](/assets/img/post/Gitlab/gitlab-runner%20config.toml%20내용.png)](/assets/img/post/Gitlab/gitlab-runner%20config.toml%20내용.png)

- gitlab-runner 재시작한다.

```bash
$ gitlab-runner restart
```

* * *

## .gitlab-ci.yml 파일 생성 :
1. 왼쪽 사이드바에서 **Code > Repository**로 이동한다.
2. 파일 목록 위에서, 커밋할 브랜치를 선택한 후, 플러스(+) 아이콘을 클릭하고, **New file**을 선택한다.
[![gitlab-ci yaml 파일 생성](/assets/img/post/Gitlab/gitlab-ci%20yaml%20파일%20생성.png)](/assets/img/post/Gitlab/gitlab-ci%20yaml%20파일%20생성.png)

3. 파일 이름`.gitlab-ci.yml`을 입력하고 큰 창에서 다음 샘플 코드를 붙여 넣는다.
```bash
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

4. **Commit changes** 버튼을 클릭한다.
- 커밋되면 파이프라인이 시작되고, `.gitlab-ci.yml` 파일에 정의한 job을 실행한다.
[![gitlab-ci yaml 파일 작성](/assets/img/post/Gitlab/gitlab-ci%20yaml%20파일%20작성.png)](/assets/img/post/Gitlab/gitlab-ci%20yaml%20파일%20작성.png)

* * *

## 파이프라인과 job 상태 보기 :
- 변경 사항을 커밋하면 파이프라인이 시작되고, **Build > Pipelines**로 이동한다.
- 세 단계(Stage)의 파이프라인이 표시되어야 한다.
[![gitlab-runner pipeline 확인](/assets/img/post/Gitlab/gitlab-runner%20pipeline%20확인.png)](/assets/img/post/Gitlab/gitlab-runner%20pipeline%20확인.png)

- Job의 세부 정보를 보려면, job 이름을 클릭한다.
![gitlab-runner pipeline job 확인](/assets/img/post/Gitlab/gitlab-runner%20pipeline%20job%20확인.png)

* * *