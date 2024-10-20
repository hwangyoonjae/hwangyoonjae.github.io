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
- 파이프라인 설정을 여러 파일로 분할해 관리하는 기능이다.

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