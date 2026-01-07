---
layout: post
title: "CI/CD란"
date: 2022-10-08
categories: DevOps 
tags: [CI, CD]
image: /assets/img/post-title/devops.jpg
---


## 1. CI/CD란?:
- 애플리케이션 개발 단계부터 배포 때까지 모든 단계를 자동화를 통해서 좀 더 효율적이고 빠르게 사용자에게 빈번히 배포할 수 있습니다.
- 아래에 CI와 CD에 대해서 자세하게 정리했습니다.

* * *

### 1.1 CI(Continuos Integration)란?:
- 빌드/테스트 자동화 과정으로 지속적인 통합을 의미합니다.
- 애플리케이션의 버그 수정이나 새로운 코드 변경이 주기적으로 빌드 및 테스트되면서 공유되는 레퍼지토리에 통합되는 것을 의미합니다.

[![텍스트](/assets/img/post/Application/CI%20%ED%9D%90%EB%A6%84%EB%8F%84.PNG)](/assets/img/post/Application/CI%20%ED%9D%90%EB%A6%84%EB%8F%84.PNG)

#### 1.1.1 CI(Continuos Integration) 장점:
- 코드의 검증에 들어가는 시간이 줄어든다.
- 버그 수정에 용이하다.
- 항상 테스트 코드를 통과한 코드만이 레포지터리에 올라가기 때문에, 좋은 코드 퀄리티를 유지할 수 있습니다.

* * *

### 1.2 CD(Continuous Delivery) & (Continuous Development)란?:
- 배포 자동화 과정으로 지속적인 제공 또는 지속적인 배포를 의미합니다.
- Bulid 되고 Test 된 후에, 배포 단계에서 Release 할 준비 단계를 거치고 문제가 없는지 수정할만한 것들이 없는지 개발자가 검증하는 팀이 검증을 하고 나온 결론으로 배포를 자동화하여 진행합니다.

[![텍스트](/assets/img/post/Application/CD%20%ED%9D%90%EB%A6%84%EB%8F%84.PNG)](/assets/img/post/Application/CD%20%ED%9D%90%EB%A6%84%EB%8F%84.PNG)

#### 1.2.1 CD(Continuous Delivery) & (Continuous Development) 장점:
- 개발자는 배포보다는 개발에 더욱 신경 쓸 수 있도록 도와준다.
- 개발자가 원클릭으로 수작업 없이 빌드, 테스트, 배포까지의 자동화를 할 수 있습니다.

* * *

## 2. CI/CD 파이프라인(Pipeline):
### 2.1 CI/CD 파이프라인(Pipeline)이란?:
- 제공된 데이터 또는 코드에 대해 사전 정의된 작업을 수행하는 일련의 처리 단계입니다.
- 반복적인 프로세스를 자동화하여 <span style='color: #000000; background-color: #81BEF7'>시간을 절약하고 정밀도를 높이는 것</span>이 파이프 라인 사용의 목적입니다.
- <span style='color: #000000; background-color: #F78181'>빌드, 테스트 및 제공을 수동 처리</span>보다 <span style='color: #000000; background-color: #81BEF7'>더 빠르고 자동화되고 안정적으로 만드는 것입니다.</span>

* * *

### 2.2 CI/CD 파이프라인(Pipeline) 구성요소
1. **코드작성(Code)** - 개발자는 주기적으로 메인 레포지토리에 통합합니다.
2. **빌드(Build)** -  자동으로 애플리케이션을 빌드합니다.
3. **테스트(Test)** - 빌드한 코드를 테스트합니다.
4. **릴리즈(Realese)** - 테스트한 코드가 이상없으면 레포지토리에 배포준비합니다.
5. **배포(Deploy)** - 준비된 애플리케이션을 배포합니다.

* * *