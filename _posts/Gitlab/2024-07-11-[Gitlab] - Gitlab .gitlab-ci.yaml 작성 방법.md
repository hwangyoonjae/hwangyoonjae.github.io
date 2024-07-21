---
layout: post
title: "[Gitlab] - Gitlab .gitlab-ci.yaml 작성 방법"
date: 2024.07.11
categories: Gitlab
tags: [Git, Gitlab, Runner]
image: /assets/img/post-title/gitLab-wallpaper.jpg
---

## 1. 기본 구조 :
```yaml
stages:
  - build
  - test
  - deploy

variables:
  VARIABLE_NAME: value
  
before_script:
  - echo "이 스크립트는 모든 작업 전에 실행됩니다"

job_name:
  stage: build
  script:
    - echo "이것은 실제 작업을 수행하는 스크립트입니다"
```

* * *

## 2. 파이프라인 설정 :
### before_script : 
- 작업 전 실행되는 명령어

```bash
before_script: 
  - echo "이 명령어는 script 섹션 전에 실행됩니다."
  - npm install # 의존성 설치 등의 준비 작업
```

### variables : 
- .gitlab-ci.yml에서 사용할 변수 선언

```bash
variables:
  DEPLOY_ENVIRONMENT: production
  API_TOKEN: $SECRET_API_TOKEN # GitLab의 보호된 변수
```

### stages :
- stages에서 배포할 때의 파이프라인의 개수(단계)를 설정해주어야 한다.

```html
stages:
  - prepare # 빌드 환경 설정, 의존성 설치 등 준비 작업
  - build # 애플리케이션 코드 컴파일 및 빌드
  - test # 단위 테스트, 통합 테스트 등 실행
  - analyze # 코드 품질 분석, 보안 검사 등 수행
  - package # 애플리케이션 패키징 (예: Docker 이미지 생성)
  - deploy # 스테이징 또는 프로덕션 환경에 배포
  - monitor # 배포 후 모니터링 및 건강 검사
  - cleanup # 임시 파일 삭제, 리소스 정리 등
```

* * *