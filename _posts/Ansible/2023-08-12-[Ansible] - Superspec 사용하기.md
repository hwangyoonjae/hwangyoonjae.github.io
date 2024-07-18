---
layout: post
title: "[Ansible] - Superspec 사용하기"
date: 2023-08-12
categories: Ansible
tags: [Ansible, Superspec]
image: /assets/img/post-title/ansible-wallpaper.jpg
---

## Superspec을 알고 싶었던 계기:
- Ansible을 통하여 Pipeline 구축을 어떻게 할까 검색하던 중 참고하던 블로그에서 Superspec 사용법의 대하여 정리한게 있어 나도 한번 알아보고, 구축을 해보면 어떨까 싶어서다.

* * *

## Superspec이란 무엇인가?:
- 서버의 구성과 상태를 테스트하기 위한 인프라스트럭처 테스팅 도구이다.

* * *

### Serverspec의 주요 특징과 용도:
- ***풍부한 언어 지원***: Serverspec은 다양한 프로그래밍 언어로 작성할 수 있으며, Ruby가 가장 널리 사용된다.
- ***선언적 문법***: 테스트를 작성할 때 원하는 서버의 상태를 선언적으로 기술하고, 코드의 가독성을 높여준다.
- ***SSH를 통한 원격 테스트***: SSH를 통해 리모트 서버에서도 테스트를 수행할 수 있다.
- ***다양한 플러그인과 통합***: 서버 프로비저닝 도구 (예: Ansible, Chef, Puppet) 등과 통합하여 사용할 수 있다.
- ***문서화***: 코드로 테스트를 작성하므로 서버 구성과 의도를 문서화하는 효과도 있다.

* * *