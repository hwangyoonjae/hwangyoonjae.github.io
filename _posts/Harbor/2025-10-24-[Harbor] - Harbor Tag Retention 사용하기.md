---
layout: post
title: "Harbor Tag Retention 사용하기"
date: 2025-10-24
categories: [DevOps, Harbor]
tags: [Harbor, Image, Tag]
image: /assets/img/post-title/harbor-wallpaper.jpg
---

## 1. Harbor Tag Retention 사용하기 :
### 1.1 Harbor Tag Retention 규칙 설정하기 :

- Harbor 관리자 로그인 > 좌측 메뉴 > Projects(프로젝트) 선택 > 관리하려는 프로젝트 클릭 > Tag Retention(정책) 탭 클릭합니다.

![Harbor Tag Retention 탭 화면](/assets/img/post/harbor/Harbor%20Tag%20Retention%20탭%20화면.png)

* * *

- "규칙 추가" 버튼을 클릭하고, 규칙에 대한 설정 정의는 아래와 같습니다.

| 항목 | 설정값 | 설명 |
|------|----------|------|
| 저장소 선택 | `**` | 모든 저장소에 적용 |
| 태그 선택 | `**` | 모든 태그에 적용 |
| 보관 규칙 유형 | 보관(Keep) | 유지할 이미지 기준 |
| 보관 기준 | 최근에 푸시된 태그 선택 |  |
| 기간 | 90일 | 90일(3개월) 이내 푸시된 이미지만 유지 |
| 매칭 방식 | 기본값 유지 | OR 조건 유지 |

![Harbor Tag Retention 규칙 추가](/assets/img/post/harbor/Harbor%20Tag%20Retention%20규칙%20추가.png)

* * *

- 정책 화면에서 Schedule (스케줄) 항목 오른쪽의 "편집" 클릭하여 스케줄 설정합니다.

> Schedule 옵션을 "사용자 정의"로 선택하여 진행합니다.
{: .prompt-warning}

```bash
# 매일 새벽 3시 0분 0초에 실행되는 cron 스케줄
0 0 3 * * *
```

![Harbor Tag Retention 스케줄](/assets/img/post/harbor/Harbor%20Tag%20Retention%20스케줄.png)

* * *

- "모의 테스트(Dry Run)" 실행 후 테스트 결과가 정상이라면 "즉시 실행"으로 실제 적용합니다.

![Harbor Tag Retention 정책 테스트 실행](/assets/img/post/harbor/Harbor%20Tag%20Retention%20정책%20테스트%20실행.png)

![Harbor Tag Retention 정책 테스트 결과](/assets/img/post/harbor/Harbor%20Tag%20Retention%20정책%20테스트%20결과.png)

* * *

### 1.2 실제 디스크 삭제하기 :
- Harbor 관리자 로그인 > 관리 > 정리를 클릭합니다.

> Retention은 태그만 삭제하기 때문에, 디스크 공간을 줄이려면 Harbor의 정리 기능(GC) 을 실행해야 합니다.
{: .prompt-info}

![Harbor Garvage Collection 설정](/assets/img/post/harbor/Harbor%20Garvage%20Collection%20설정.png)

* * *

- 가비지 컬렉션 예약 스케줄 설정합니다.

> 가비지 컬렉션 예약 옵션을 "사용자 정의"로 선택하여 진행합니다.
{: .prompt-warning}

```bash
# 매일 새벽 4시 0분 0초에 실행되는 cron 스케줄
0 0 4 * * *
```

![Harbor Garvage Collection 스케줄 결과](/assets/img/post/harbor/Harbor%20Garvage%20Collection%20스케줄%20결과.png)

* * *