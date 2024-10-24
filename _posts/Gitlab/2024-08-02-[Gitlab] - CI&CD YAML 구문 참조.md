---
layout: post
title: "[Gitlab] - Gitlab CI&CD YAML 구문 참조"
date: 2024.08.02
categories: Gitlab
tags: [Git, Gitlab, Runner, CI, CD]
image: /assets/img/post-title/gitLab-wallpaper.jpg
---

## 키워드 :
- GitLab CI/CD 파이프라인 구성에는 다음이 포함된다.

### 파이프라인 동작을 구성하는 글로벌 키워드 :

|키워드|설명|
|-----|-----|
|default|작업 키워드에 대한 사용자 정의 기본값|
|include|다른 YAML 파일에서 구성을 가져온다.|
|stages|파이프라인 단계의 이름과 순서|
|variables|	파이프라인의 모든 작업에 대해 CI/CD 변수를 정의|
|workflow|파이프라인 실행 유형을 제어|

* * *

## 헤더 키워드 :

|키워드|설명|
|-----|-----|
|spec|외부 구성 파일에 대한 사양을 정의|

* * *

### 작업 키워드로 구성된 작업 :

|키워드|설명|
|-----|-----|
|after_script|작업 후 실행되는 명령 집합을 재정의|
|allow_failure|작업이 실패하도록 허용하고, 실패한 작업은 파이프라인이 실패하게 하지 않는다.
|artifacts|빌드나 테스트 과정에서 생성된 파일을 저장하고 공유|
|before_script|작업 전에 실행되는 명령 집합을 재정의|
|cache|후속 실행 사이에 캐시되어야 하는 파일 목록|
|coverage|주어진 작업에 대한 코드 범위 설정|
|dast_configuration|작업 수준에서 DAST 프로필의 구성을 사용|
|dependencies|아티팩트를 가져올 작업 목록을 제공하여 특정 작업에 전달되는 아티팩트를 제한|
|environment|작업이 배포되는 환경의 이름|
|extends|이 작업이 상속받는 구성 항목|
|identity|ID 연합을 사용하여 타사 서비스를 인증|
|image|Docker 이미지를 사용|
|inherit|모든 작업이 상속받는 글로벌 기본값을 선택|
|interruptible|새로운 실행으로 인해 중복된 작업이 취소될 수 있는지 여부를 정의|
|manual_confirmation|수동 작업에 대한 사용자 정의 확인 메시지를 정의|
|needs|단계 순서보다 일찍 작업을 실행|
|pages|GitLab Pages에서 사용할 작업 결과를 업로드|
|parallel|병렬로 실행해야 하는 작업 인스턴스의 수|
|release|주자에게 릴리스 객체를 생성하도록 지시|
|resource_group|작업 동시성을 제한|
|retry|작업이 실패할 경우 자동으로 재시도할 수 있는 횟수와 시기|
|rulese|선택된 작업 속성을 평가하고 결정하고, 작업이 생성되었는지 여부를 확인하는 조건 목록|
|script|러너에 의해 실행되는 셸 스크립트|
|secrets|CI/CD는 작업에 필요한 비밀|
|services|Docker 서비스 이미지를 사용|
|stage|작업 단계를 정의|
|tags|주자를 선택하는 데 사용되는 태그 목록|
|timeout|프로젝트 전체 설정보다 우선하는 사용자 지정 작업 수준 시간 제한을 정의|
|trigger|다운스트림 파이프라인 트리거를 정의|
|variables|직무 수준에서 직무 변수를 정의|
|when|작업을 실행할 시기|

* * *

## 글로벌 키워드 :
### include :
- 파이프라인 설정을 여러 파일로 분할해 관리하는 기능으로 외부 yaml 파일의 확장자가 반드시 .yml 혹은 .yaml 이어야 한다.

|옵션|설명|
|-----|-----|
|local|로컬 프로젝트 repository 안의 파일을 포함|
|file|다른 프로젝트 repository 안의 파일을 포함|
|remote|원격 URL의 파일을 포함|
|template|GitLab 에서 제공하는 템플릿 포함|

- YAML 파일을 포함하는 가장 간단하고 직접적인 방법인 로컬 파일을 포함하는 방법은 아래와 같다.

```yaml
include: # Local 파일 포함
  - local: "workflow/workflow.gitlab-ci.yml"
  - local: "template/build/build.gitlab-ci.yml"
```

* * *

### stages :
- Job들을 단계적으로 실행하기 위해 사용
- Job이 어느 단계(Stage)에 속하는지 명시할 때 사용

> Job이 Stage에 속해야 하는 이유?
>
> 파이프라인 흐름을 정의하기 위해서다.
> 
> 스테이지를 지정하지 않은 Job은 오류가 발생하며, 파이프라인이 올바르게 실행되지 않는다.
> 
{: .prompt-info}

```yaml
stages:
  - build
  - test
  - deploy
```

* * *

## 잡 키워드 :
### artifats :
- 빌드나 테스트 과정에서 생성된 파일을 저장하고 공유한다.

```yaml
job_name:
  script:
    - echo "빌드 실행 중"
    - mkdir out/
    - echo "결과물 파일" > out/output.txt
  artifacts:
    paths:
      - out/output.txt # 파일 저장 경로
    expire_in: 1 week # 다운로드 가능 기간 설정
```

![artifacts 키워드 job 실행화면](/assets/img/post/Gitlab/artifacts%20키워드%20job%20실행화면.png)

> artifacts 저장 위치?
>
> 서버는 아래 적힌 경로에서 확인이 가능하다.
> 
> "[/home/계정명]/builds/[project]/[job_id]/out/"
>
> Gitlab 브라우저 → Pipelines → Job 상세 페이지에서도 다운로드 가능하다.
> 
{: .prompt-info}

* * *

### needs :
- 특정 Job이 다른 Job의 실행 결과에 의존할 때 사용한다.
- 다른 stage나 Job이 먼저 실행되도록 명시적으로 설정할 수 있다.

```yaml
stages:
  - build
  - test

job_name:
  stage: build
  script:
    - echo "빌드 실행 중"
    - mkdir out/
    - echo "결과물 파일" > out/output.txt
  artifacts:
    paths:
      - out/output.txt
    expire_in: 1 week

job2_name:
  stage: test
  before_script:
    - echo "before"
  script:
    - echo "main script"
  after_script:
    - echo "after script"
  needs:
    - job: job_name
```

![needs 키워드 job 실행화면](/assets/img/post/Gitlab/needs%20키워드%20job%20실행화면.png)

* * *

### before_script, script, after_script :
- **before_script**: 각 job의 메인 스크립트(script)가 실행되기 전에 공통적으로 실행할 명령어를 정의
- **script**: 해당 job의 핵심 작업을 수행하는 명령어를 정의하고 반드시 필요한 작업
- **after_script**: **script**가 끝난 후, 성공 여부와 상관없이 항상 실행되는 명령어를 정의

```yaml
stages:
  - build

build_job:
  stage: build
  before_script:
    - echo "Setting up build environment"
  script:
    - echo "Building project"
    - mkdir output
    - echo "Build complete" > output/result.txt
  after_script:
    - echo "Cleaning up..."
    - rm -rf output/
```

![script 키워드 job 실행화면](/assets/img/post/Gitlab/script%20키워드%20job%20실행화면.png)

> script에서 환경 변수나 설정 파일 재적용 방법
>
> 파이프라인의 일부 환경변수나 특정 설정은 init.var와 같은 파일에 정의하는데 **after_script**에서는 리소스를 정리하거나 마무리 작업을 수행해야 하므로, 해당 환경변수들이 필요하다.
> 
> 만약 이러한 변수나 경로 설정이 없다면, 정리 작업이 실패하거나 리소스 삭제가 누락될 수 있다.
> 
{: .prompt-tip}

* * *

### extends :
- 공통 설정을 재사용하고자 할 때 사용하는 기능
- 여러 Job에서 동일한 설정을 반복하지 않고, 공통된 Job 템플릿을 정의한 후 이를 상속하여 재사용할 수 있다.

```yaml
stages:
  - build
  - test

.default_template:
  stage: build
  before_script:
    - echo "Setting up environment"
  script:
    - echo "Running build"
  artifacts:
    paths:
      - build_output/
    expire_in: 1 week

build_job:
  extends: .default_template
  script:
    - echo "Compiling source code"

test_job:
  extends: .default_template
  stage: test
  script:
    - echo "Running tests"
```

![extends 키워드 job 실행화면](/assets/img/post/Gitlab/extends%20키워드%20job%20실행화면.png)

> extends 사용 시의 규칙
>
> **점(.)으로 시작하는 템플릿**: 보통 공통 템플릿은 이름 앞에 .을 붙여 표시하는데 이 점(.)으로 시작하는 이름은 GitLab에서 직접 실행되지 않는 템플릿으로 간주한다.
> 
{: .prompt-info}

> 템플릿 사용 시의 주의 사항
>
> **1. 이름 중복 방지**: 점(.)으로 시작하는 이름은 고유해야 하며, 실행되지 않는 템플릿으로 간주되기 때문에 이름을 구별하여 사용
>
> **2. 오버라이드 주의**: 개별 Job에서 템플릿 설정을 오버라이드할 때는 완전히 대체되기 때문에, 원하는 설정이 포함되었는지 확인 필요 (Job 내용이 우선)
> 
{: .prompt-warning}

* * *

### allow_failure :
- GitLab CI/CD에서 특정 Job이 실패해도 전체 파이프라인의 실패로 간주하지 않도록 설정하는 옵션
- 실패해도 파이프라인을 멈추지 않고 다음 Job을 계속 실행할 수 있다.

```yaml
stages:
  - build
  - test

job1:
  stage: build
  script:
    - echo "빌드 진행 중..."
    - exit 1 
  allow_failure: true

job2:
  stage: test
  script:
    - echo "테스트 진행 중..."
```

![allow_failure 키워드 job 실행화면](/assets/img/post/Gitlab/allow_failure%20키워드%20job%20실행화면.png)

> 주의할 점
> 
> **모니터링 어려움**: allow_failure를 설정하면 Job 실패가 무시되므로, 실패 상황을 놓칠 위험이 있습니다. 중요한 Job에는 사용하지 않는 것이 좋다.
> 
> **디버깅 어려움**: 여러 Job에 대해 allow_failure를 활성화하면, 파이프라인 문제를 추적하고 해결하기 어려울 수 있다.
{: .prompt-warning}

* * *

### rules :
- 파이프라인의 특정 Job을 언제 실행할지 결정하는 조건을 설정

```yaml
job_name:
  script:
    - echo "Running Job"
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"' 
```

```yaml
job_name:
  script:
    - echo "Merge Request Event"
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
```

* * *