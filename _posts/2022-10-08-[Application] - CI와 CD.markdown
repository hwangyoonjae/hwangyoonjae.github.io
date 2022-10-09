---
title: "[Application] - CI/CD란"
layout: post
date: 2022-10-08
image: /assets/images/Post/infinity.png
headerImage: true
tag:
- CI
- CD
category: blog
author: hwangyoonjae
description: Markdown summary with different options
---

## CI/CD란?:
- 애플리케이션 개발 단계부터 배포 때까지 모든 단계를 자동화를 통해서 좀 더 효율적이고 빠르게 사용자에게 빈번히 배포할 수 있다.
- 아래에 CI와 CD에 대해서 자세하게 정리했다.

* * *

### CI (Continuos Integration)
- 빌드/테스트 자동화 과정으로 지속적인 통합을 의미한다.
- 애플리케이션의 버그 수정이나 새로운 코드 변경이 주기적으로 빌드 및 테스트되면서 공유되는 레퍼지토리에 통합되는 것을 의미한다.

#### CI 장점
```
- 코드의 검증에 들어가는 시간이 줄어든다.
- 개발 편의성이 증가한다.
- 항상 테스트 코드를 통과한 코드만이 레포지터리에 올라가기 때문에, 좋은 코드 퀄리티를 유지할 수 있다.
```

* * *

### CD (Continuous Delivery) & (Continuous Development)
- 배포 자동화 과정으로 지속적인 제공 또는 지속적인 배포를 의미한다.
- Bulid 되고 Test 된 후에, 배포 단계에서 release 할 준비 단계를 거치고 문제가 없는지 수정할만한 것들이 없는지 개발자가 검증하는 팀이 검증을 하고 나온 결론으로 배포를 자동화하여 진행한다.

#### CD 장점
```
- 개발자는 배포보다는 개발에 더욱 신경 쓸 수 있도록 도와준다.
- 개발자가 원클릭으로 수작업 없이 빌드, 테스트, 배포까지의 자동화를 할 수 있다.
```

* * *