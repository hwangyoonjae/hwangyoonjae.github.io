---
title: "[Linux] - Ansible 설치하기"
layout: post
date: 2022-08-24
image: /assets/images/Post/ansible.png
headerImage: true
tag:
- Ansible
- 자동화
category: blog
author: hwangyoonjae
description: Markdown summary with different options
---

## Ansible 무엇인가?:
- 인프라 관리를 코드 기반으로 자동화하는 도구이다.
- Infrastructure as Code (IaC)는 IT 인프라를 코드 기반으로 자동 설치 및 구축/관리/프로비저닝 하는 프로세스를 말하며 관련 도구들이 나오면서 자동화된 인프라 구축이 가능하다.
- 코드 스크립트를 실행하기만 하면 코드가 알아서 다 해주게된다.

* * *

### Ansible 특징 :
1. **멱등성 지원**: 
    - 멱등성 : 여러번 실행해도 같은 결과 값이 나오는 성질
    - Ansible에서의 멱등성 : 결과의 상태값이 다르더라도 결국에 결과는 동일하게 나오게 하는 성질
2. **Modular** 
    - 많은 모듈 지원 
    - Shell에 의존하지 않고 Ansible에서 지원하는 Module로 구성관리에 용이하다.
3. **YAML 형식 지원** 
    - 기존의 Shell Scripts보다 간편하게 구성 가능하다.
4. **대형 워크로드에 용이** 
    - 많은 서버에 구성이 필요한 경우 Shell Scripts보다 신속하게 처리 가능하다.
    
* * *