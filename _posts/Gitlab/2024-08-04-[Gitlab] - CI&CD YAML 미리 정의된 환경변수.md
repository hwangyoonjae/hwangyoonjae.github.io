---
layout: post
title: "Gitlab CI&CD YAML 미리 정의된 환경변수"
date: 2024.08.04
categories: [DevOps, Gitlab] 
tags: [Git, Gitlab, Runner, CI, CD]
image: /assets/img/post-title/gitLab-wallpaper.jpg
---

## 미리 정의된 주요 GitLab CI/CD 환경 변수 :
- GitLab CI/CD는 여러 가지 자동 환경 변수를 제공하여 파이프라인 내에서 다양한 정보를 사용할 수 있도록 지원합니다.

|환경변수|설명|
|-----|-----|
|CI_COMMIT_BRANCH|현재 커밋이 속한 브랜치를 나타낸다.|
|CI_COMMIT_TAG|현재 커밋이 태그로 작성된 경우 이 변수에 태그 이름이 포함된다.|
|CI_PIPELINE_ID|현재 파이프라인의 고유 ID를 나타낸다.|
|CI_PROJECT_NAME|현재 파이프라인이 실행되고 있는 프로젝트의 이름을 나타낸다.|
|CI_JOB_NAME|현재 실행 중인 잡의 이름을 나타낸다.|
|CI_COMMIT_SHA|현재 커밋의 SHA 해시를 나타낸다.|

## 사용 제약이 있는 GitLab CI/CD 환경 변수 :
- 러너가 사용 가능하게 한 사전 정의된 변수들 중 트리거 작업이나 다음 키워드에서 사용할 수 없다.

> workflow, include, rules
{: .prompt-info}

### 위 키워드에서 사용이 제한될 수 있는 변수 :

|환경변수|설명|
|-----|-----|
|CI_JOB_ID|현재 잡의 ID를 나타냅니다. workflow 및 rules에서는 사용되지 않는다.|
|CI_JOB_NAME|현재 잡의 이름을 나타냅니다. workflow와 rules에서는 사용할 수 없다.|
|CI_JOB_STAGE|현재 잡이 속한 단계의 이름을 나타냅니다. workflow 및 rules에서 사용할 수 없다.|
|CI_RUNNER_ID|현재 파이프라인을 실행하는 러너의 ID를 나타냅니다. workflow 및 rules에서 사용할 수 없다.|
|CI_RUNNER_DESCRIPTION|현재 러너의 설명을 나타냅니다. workflow 및 rules에서 사용할 수 없다.|
|CI_PIPELINE_SOURCE|파이프라인이 시작된 원인을 나타냅니다 (예: push, merge_request, schedule 등). workflow 및 rules에서는 직접적으로 사용할 수 없다.|
|CI_PROJECT_ID|현재 프로젝트의 ID를 나타냅니다. workflow 및 rules에서는 사용할 수 없다.|
|CI_PROJECT_PATH|프로젝트의 경로를 나타냅니다. workflow 및 rules에서 사용할 수 없다.|
|CI_SERVER_URL|GitLab 서버의 URL을 나타냅니다. workflow 및 rules에서 사용할 수 없다.|
|CI_USER_LOGIN|현재 사용자의 로그인 이름을 나타냅니다. workflow 및 rules에서 사용할 수 없다.|

* * *