---
title: "[JAVA] - Elasticsearch란"
layout: post
date: 2022-08-12
image: /assets/images/Post/elasticsearch.png
headerImage: true
tag:
- DB
category: blog
author: hwangyoonjae
description: Markdown summary with different options
---

## Elasticsearch란? :
- Elasticsearch는 Apache Lucene(아파치 루씬) 기반의 java 오픈소스 분산 검색 엔진이다.
- 방대한 양의 데이터를 신속하고 거의 실시간으로 저장,검색,분석할 수 있다.
- 검색을 위해 단독으로 사용되기도 하며,  ELK( Elasticsearch / Logstatsh / Kibana )스택으로 사용되기도 한다.

#### [ELK 스택]
- **Logstash**
  - 다양한 소스( DB, csv파일 등 )의 로그 또는 트랜잭션 데이터를 수집, 집계, 파싱하여 Elasticsearch로 전달
- **Elasticsearch**
  - Logstash로부터 받은 데이터를 검색 및 집계를 하여 필요한 관심 있는 정보를 획득
- **Kibana**
  - Elasticsearch의 빠른 검색을 통해 데이터를 시각화 및 모니터링
* * *

### Elasticsearch 흐름도 :
![텍스트](/assets/images/JAVA/ELK%20%EA%B5%AC%EC%A1%B0.PNG)

* * *