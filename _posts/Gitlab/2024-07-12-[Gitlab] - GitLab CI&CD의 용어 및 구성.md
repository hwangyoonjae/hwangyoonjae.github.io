---
layout: post
title: "GitLab CI&CD의 용어 및 구성"
date: 2024.07.12
categories: [DevOps, Gitlab] 
tags: [Git, Gitlab, Runner, Harbor]
image: /assets/img/post-title/gitLab-wallpaper.jpg
---

## Pipelines :
- CI/CD 의 최상위 구성 요소이며 수행할 작업을 정의하는 Jobs
  예를 들어, 코드를 컴파일하거나 테스트하는 Job과 작업을 실행할 시기를 정의하는 Stages(예를 들어, 코드를 컴파일하는 단계 후에 테스트를 실행하는 단계로 구성
- 한 단계의 모든 Job이 성공하면, 파이프라인은 다음 단계로 넘어간다.
- 한 단계의 어떤 Job이 실패하면, 다음 단계는 (일반적으로) 실행되지 않고 파이프라인이 일찍 종료됩니다.

* * *

## Jobs :
- .gitlab-ci.yml 파일의 가장 기본적인 요소  
- Runner가 실행해야 하는 명령 모음으로 Job의 결과물(Output)이 무엇인지 실시간으로 볼 수 있으므로, 개발자는 Job이 실패한 이유를 이해할 수 있습니다.
	- 임의의 이름을 가진 최상위 요소이며 최소한 script 절을 포함해야 합니다.

```yaml
test:
  script:
    - python setup.py test
run:
  script:
    - python setup.py bdist_wheel
```

### 전역 파라미터 :
- 모든 job에 공툥으로 사용되는 전역 파라미터는 default 키워드를 사용하여 설정 가능하다.

>default 키워드 내 사용 가능한 job 파라미터
>
>- image
>- services
>- before_script
>- after_script
>- tags
>- cache
>- artifacts
>- retry
>- timeout
>- interruptible
{: .prompt-info }

* * *

## Variables :
- CI/CD 변수는 환경 변수의 한 유형
	- Jobs 및 파이프라인의 동작을 제어
	- 재사용하려는 값을 저장
	- .gitlab-ci.yml 파일에 값을 하드 코딩하는 것을 방지
```yaml
variables:
  HARBOR_REGISTRY_URL: harbor.com
  HARBOR_USER: admin
  HARBOR_PASSWORD: 1234
  HARBOR_PROJECT: hyj-test
```

* * *

## Environments :
- 코드가 배포되는 위치를 설명
- 프로젝트와 연결된 Kubernetes와 같은 배포 서비스가 있는 경우, 이를 사용하여 배포를 지원 가능

* * *

## Runners :
- GitLab CI/CD의 코디네이터 API를 통해 CI Job을 선택하고, Job을 실행한 다음, 결과를 GitLab 인스턴스로 다시 보내는 경량의 확장성이 뛰어난 에이전트

>Runner의 유형
>
>- 공유 러너(Shared runners)는 GitLab 인스턴스의 모든 그룹 및 프로젝트에서 사용
>- 그룹 러너(Group runners)는 그룹의 모든 프로젝트와 하위 그룹에서 사용
>- 특정 러너(Specific runners)는 특정 프로젝트와 연결됩니다. 일반적으로 특정 러너는 한 번에 하나의 프로젝트에 사용
{: .prompt-info }

* * *

## Artifacts :
- 단계(Stages) 간에 전달되는 단계 결과에 사용
- 저장 및 업로드할 수 있도록 Job에 의해 생성되는 파일로 동일한 파이프라인의 이후 단계의 Job에서 아티팩트를 가져와 사용할 수 있습니다.
- 한 단계의 Job에서 아티팩트를 생성하여 동일한 단계의 다른 Job에서 이 아티팩트를 사용할 수 없고, 이 데이터는 다른 파이프라인에서 사용할 수 없지만 UI에서 다운로드할 수 있습니다.
- 애플리케이션을 빌드하는 동안 모듈을 다운로드하는 경우, 이를 아티팩트로 선언하고 후속 단계의 Job에서 이를 사용할 수 있습니다.

* * *

## Cache :
- 프로젝트 의존성(Dependencies)을 저장하는 데 사용
- 캐시는 후속 파이프라인에서 지정된 Job의 속도를 높일 수 있습니다.
- 인터넷에서 다시 가져올 필요가 없도록 다운로드 한 의존성을 저장할 수 있습니다. 
- 의존성에는 npm 패키지, Go의 vendor 패키지 등이 포함됩니다. 단계 간에 중간 빌드 결과를 전달하도록 캐시를 구성할 수 있지만 대신 아티팩트를 사용해야 합니다.

* * *

# GitLab CI 의 속도 향상 :
- GitLab CI 파이프라인에서 Cache 를 사용하면 속도를 향상시킬 수 있습니다.

```yaml
cache:
  paths:
    - .cache/pip
    - venv/
```

>위 설정 상세설명
>
>- cache/pip : pip가 다운로드한 패키지들이 저장되는 디렉토리로 이 디렉토리를 캐싱하면, 동일한 패키지를 매번 다운로드할 필요가 없다.
>- venv/Python 가상환경 디렉토리로 이 디렉토리를 캐싱하면, 가상환경을 매번 새로 생성할 필요가 없다.
{: .prompt-info }

>cache 사용 시 장점
>
>- **의존성 설치 시간 절약**: 패키지 관리자(pip 등)로부터 의존성을 다시 다운로드할 필요 없이, 이전에 설치된 의존성을 캐시에서 가져올 수 있습니다.
>- **빌드 시간 단축**: 빌드 아티팩트, 컴파일된 파일 등을 캐시하면, 매 빌드마다 다시 컴파일할 필요 없이 캐시된 파일을 재사용할 수 있습니다.
>- **네트워크 대역폭 절약**: 캐시를 사용하면 매번 인터넷에서 파일을 다운로드할 필요가 없어 네트워크 대역폭을 절약할 수 있습니다.
{: .prompt-tip }

---