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

### 작업 키워드로 구성된 작업 키워드 :

|키워드|설명|
|-----|-----|
|after_script|작업 후 실행되는 명령 집합을 재정의|
|allow_failure|작업이 실패하도록 허용하고, 실패한 작업은 파이프라인이 실패하게 하지 않는다.
|artifacts|작업이 성공했을 때 첨부할 파일 및 디렉토리 목록|
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
### default :
- 파이프라인 내 모든 작업에 대해 공통적으로 적용할 설정을 정의하는 데 사용된다.

> default 키워드는 GitLab 16.4 버전부터 지원되기 때문에, 이전 버전에서는 사용할 수 없다.
{: .prompt-warning}

- default 키워드 작업 설정

|:---:||:---:||:---:|
|after_script|artifacts|before_script|
|cache|hooks|id_tokens|
|image|interruptible|retry|
|services|tags|timeout|

- default 키워드 사용 (예시)

```yml
default:
  image: nginx:1.25.3 
  before_script:
    - echo "Starting job..."
```

- default 키워드 사용 (예시) (실습)

```yml
default:
  image: nginx:1.25.3 
  before_script:
    - echo "Starting job..."

stages:
  - test
  - deploy

test-job:
  stage: test
  script:
    - echo "Running tests..."
    - nginx -v

deploy-job:
  stage: deploy
  script:
    - echo "Deploying application..."
    - cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak
```

> 설명
>
> **image**: 모든 작업에서 기본적으로 node:16 Docker 이미지를 사용 <br>
> **before_script**: 각 작업 시작 전에 npm install을 실행하여 의존성을 설치 <br>
> **after_script**: 각 작업 후에 "Cleaning up..." 메시지를 출력 <br>
> **tags**: docker 태그를 사용하여 특정 Runner에서 작업이 실행되도록 설정 <br>
{: .prompt-example}

* * *

### include :
- CI/CD 구성에 외부 YAML 파일을 가져와 현재 파일의 설정에 병합하는 데 사용된다.

- include 키워드 사용 (예시)

```yml
# 같은 프로젝트 내 파일 포함인 경우
include:
  - local: '/path/to/common-config.yml'

# 다른 프로젝트 내 파일 포함인 경우
include:
  - project: 'my-group/my-project'
    file: '/path/to/file.yml'

# 원격 URL에서 파일 포함인 경우
include:
  - remote: 'https://myserver.com/configs/common-config.yml'

# GitLab 템플릿 포함인 경우
include:
  - template: 'Auto-DevOps.gitlab-ci.yml'
```

- include 키워드 사용 (실습)

```yml
# .gitlab-ci.yml
stages:
  - build
  - test

include:
  - local: 'extra-config.yml'

build-job:
  stage: build
  script:
    - echo "Building the application..."
```

```yml
# extra-config.yml
test-job:
  stage: test
  script:
    - echo "Running tests..."
```

* * *

### stages :
- 작업 그룹이 포함된 단계를 정의하는데 사용하고, 작업에서 사용하여 특정 단계에서 작업을 실행하도록 구성된다.

- 잡 실행 순서:
  - **Build 단계**: build-job이 실행됩니다.
  - **Test 단계**: test-job이 실행됩니다. build 단계가 성공적으로 완료되면 시작됩니다.
  - **Deploy 단계**: deploy-job이 실행됩니다. test 단계가 성공적으로 완료되면 시작됩니다.

- stages 키워드 사용 (예시)

```yml
stages:
  - build
  - test
  - deploy
```

- stages 키워드 사용 (실습)

```yml
# .gitlab-ci.yml
stages:
  - build
  - test
  - deploy

build-job:
  stage: build
  script:
    - echo "Building the project" # 빌드 관련 스크립트 실행

test-job:
  stage: test
  script:
    - echo "Running tests" # 테스트 관련 스크립트 실행

deploy-job:
  stage: deploy
  script:
    - echo "Deploying the application" # 배포 관련 스크립트 실행
```

* * *

### workflow :
- CI/CD 파이프라인의 단계별 실행을 제어하는 데 사용되는 기능이다.
- 특정 조건을 기반으로 파이프라인의 실행 여부를 결정하는 데 도움을 준다.

- 역할
  - **파이프라인 실행 제어**: workflow 키워드는 조건에 따라 전체 파이프라인의 실행 여부를 결정하고, 이 키워드를 사용하면 특정 브랜치, 파일 변경, 또는 특정 변수를 기반으로 파이프라인의 실행을 결정할 수 있다.
  - **효율적인 리소스 사용**: 조건에 따라 파이프라인을 실행하지 않음으로써 CI/CD 리소스를 절약할 수 있다. <br> 예를 들어, 문서 파일만 변경되었을 때 빌드나 테스트를 실행할 필요가 없다면, 이러한 조건을 설정할 수 있다.

- workflow 키워드 사용 (예시)

```yml
workflow:
  rules:
    - 조건
```

- workflow 키워드 사용 (실습)

```yml
default:
  image: node:16 # 모든 잡에 대해 기본 Node.js 16 이미지를 사용

stages:
  - build
  - test
  - deploy

workflow:
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"' # 'main' 브랜치에서만 파이프라인 실행

build-job:
  stage: build
  script:
    - echo "Building the project"
    - npm install
    - npm run build

build-job-test:
  stage: build
  script:
    - echo "Building the project"
    - npm install
    - npm run build
  rules:
    - if : '$CI_COMMIT_BRANCH == "test"' # 'test' 브랜치에서만 파이프라인 실행

test-job:
  stage: test
  script:
    - echo "Running tests"
    - pip install -r requirements.txt
    - pytest

deploy-job:
  stage: deploy
  script:
    - echo "Deploying the application"
```

> 'workflow: rules'와 rules in Jobs의 차이
> 
> **workflow: rules**: 전체 파이프라인의 실행 여부를 결정하며, 조건이 맞지 않으면 파이프라인 자체가 실행되지 않는다.<br>
> **rules in Jobs**: 개별 잡의 실행 여부를 결정하며, 파이프라인이 실행되더라도 특정 잡이 실행되지 않을 수 있다.
{: .prompt-tip}

* * *

## 잡 키워드 :
### after_script :
- 각 잡(job)이 완료된 후에 실행할 스크립트를 지정하는 데 사용된다.

- after_script 사용 목적
  - **정리 작업**: 테스트 중 생성된 임시 파일이나 디렉토리 등을 삭제하는 등의 정리 작업에 사용됩니다.
  - **로그 수집**: 로그 파일이나 시스템 상태를 수집하여 외부 저장소에 업로드하거나 저장하는 데 사용됩니다.
  - **알림 전송**: 잡의 결과를 슬랙(Slack)이나 이메일 등으로 알리는 스크립트를 실행할 수 있습니다.
  - **상태 보고**: 잡의 종료 상태와 관련된 메타데이터를 기록하거나 보고서에 추가할 수 있습니다.
  - **리소스 해제**: 사용한 외부 리소스(예: 데이터베이스 연결, 원격 서버 등)를 해제하는 작업을 포함할 수 있습니다.

> 잡이 성공(SUCCESS), 실패(FAILURE), 취소(CANCELED)된 경우 모두 실행된다.<br>
> script 키워드로 정의된 작업이 완료된 후에 실행된다.
{: .prompt-info}

- after_script 키워드 사용 (예시)

```yml
job_name:
  script:
    - echo "Running main script"
    - touch temp_file.txt  # 임시 파일 생성
  after_script:
    - echo "Running cleanup"
    - rm temp_file.txt  # 임시 파일 삭제
    - tar -czf logs.tar.gz log/  # 로그 디렉토리를 압축
```

- after_script 키워드 사용 (실습)

```yml
default:
  image: harbor.inno.com/secloudit-util/nginx:1.25.3

stages:
  - build
  - test
  - deploy

build-job:
  stage: build
  script:
    - echo "Building the project"

test-job:
  stage: test
  script:
    - echo "Running tests"
  after_script:
    - echo "This is after script"

deploy-job:
  stage: deploy
  script:
    - echo "Deploying the application"
```

* * *