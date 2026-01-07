---
layout: post
title: "Elasticsearch란"
date: 2022-08-12
categories: [검색 & 로그, Test] 
tags: [JAVA, DATA]
image: /assets/img/post-title/elasticsearch-wallpaper.jpg
---

## 1. Elasticsearch란? :
- Elasticsearch는 Apache Lucene(아파치 루씬) 기반의 java 오픈소스 분산 검색 엔진입니다.
- 방대한 양의 데이터를 신속하고 거의 실시간으로 저장,검색,분석할 수 있습니다.
- 검색을 위해 단독으로 사용되기도 하며,  ELK( Elasticsearch / Logstatsh / Kibana )스택으로 사용되기도 합니다.

### 1.1 ELK 스택 :
- **Logstash**
  - 다양한 소스( DB, csv파일 등 )의 로그 또는 트랜잭션 데이터를 수집, 집계, 파싱하여 Elasticsearch로 전달
- **Elasticsearch**
  - Logstash로부터 받은 데이터를 검색 및 집계를 하여 필요한 관심 있는 정보를 획득
- **Kibana**
  - Elasticsearch의 빠른 검색을 통해 데이터를 시각화 및 모니터링

* * *

### 1.2 Elasticsearch 흐름도 :
![텍스트](/assets/img/post/JAVA/ELK%20%EA%B5%AC%EC%A1%B0.PNG)

* * *

### 1.3 Elasticsearch 핵심 개념 :
1. 클러스터 :
   - 클러스터는 하나 이상의 노드(서버)가 모인 것이며, 이를 통해 전체 데이터를 저장하고 모든 노드를 포관하는 통합 색인화 및 검색 기능을 제공합니다.
   - 클러스터는 고유한 이름으로 식별 되는데, 기본 이름은 "elasticsearch"입니다.

2. 노드 :
   - 노드는 클러스터에 포함된 단일 서버로서 데이터를 저장하고 클러스터의 색인화 및 검색 기능을 제공합니다.
   - 노드는 클러스터처럼 이름으로 식별되는데, 기본이름은 시작 시 노드에 지정되는 임의 UUID(Universally Unique Identifier)입니다.

3. 인덱스 :
   - 색인은 다소 비슷한 특성을 가진 문서의 모음입니다.
   - 색인은 이름(모두 소문자여야함)으로 식별되며, 이 이름은 색인에 포함된 문서에 대한 색인화, 검색, 업데이트, 삭제 작업에서 해당 색인을 가르키는데 쓰인다.

4. 타입 :
   - 하나의 색인에서 하나 이상의 유형을 정의할 수 있습니다.
   - 유형이란 색인을 논리적으로 분류/구분한 것이며 그 의미 체계는 전적으로 사용자가 결정합니다.
   - 일반적으로 여러 공통된 필드를 갖는 문서에 대해 유형이 정의 됩니다.

5. 도큐먼트 :
   - 도큐먼트는 색인화 할 수 있는 기본 정보 단위입니다.

* * *