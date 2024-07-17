---
layout: post
title: "[DB] - GROUP BY절과 Having절"
date: 2023-03-08
categories: DB
tags: [GROUP BY, Having]
image: /assets/post/db-wallpaper.jpg
---


## GROUP BY절과 Having절을 알고싶었던 계기:
- 고객사에서 사용하는 데이터베이스 데이터 중 중복된 데이터를 확인하고 싶어 쿼리문을 작성하던 중 Having 절을 사용하게 되어 좀 더 자세하게 알고싶어서다.

* * *

## GROUP BY절이란?:
- 같은 값을 가진 행을 그룹짓는 SQL 명령어
- COUNT(), MAX(), MIN(), SUM(), AVG() 등 집계 함수와 함께 사용된다.
- FROM절과 WHERE절 뒤에 위치한다.

* * *

## GROUP BY절 사용법:
```sql
SELECT
	COUNT(CULUMN) AS cnt,
	CULUMN1
FROM
	table
{필요 시 WHERE 조건구문}
GROUP BY
	CULUMN
```
<span style="color:#FA5858; font-size:12px">※ GROUP BY절은 중복되지 않게 데이터를 보여준다.</span>

### DISTINCT와 GROUP BY절 사용하는 경우:
```
▸ DISTINCT는 주로 UNIQUE(중복을 제거)한 컬럼이나 레코드를 조회하는 경우 사용한다.
▸ GROUP BY는 데이터를 그룹핑해서 그 결과를 가져오는 경우 사용한다.
```

* * *

## Having절이란?:
- 집계함수를 가지고 조건비교를 할 때 사용한다.
- COUNT(), MAX(), MIN(), SUM(), AVG() 등 집계 함수를 사용 할 수 없다.
- GROUP BY절과 함께 사용한다.

* * *

## Having절 사용법:
```sql
SELECT
	COUNT(CULUMN) AS cnt,
	CULUMN1
FROM
	table
{필요 시 WHERE 조건구문}
GROUP BY
	CULUMN
HAVING
	COUNT(CULUMN) > 1;
```
<span style="color:#FA5858; font-size:12px">※ GROUP BY에 사용되지 않은 컬럼을 조건으로 사용하면 오류가 발생한다.</span>

* * *
