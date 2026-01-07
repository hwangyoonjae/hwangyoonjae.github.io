---
layout: post
title: "IN, NOT IN 연산자 사용법"
date: 2022-12-10
categories: DB 
tags: [DB, IN, NOT IN]
image: /assets/img/post-title/db-wallpaper.jpg
---

## 1. IN, NOT IN 연산자? :
- 값이 포함되는 혹은 포함되지 않는 데이터를 추출하고 싶을 때 사용하는 것이 IN, NOT IN문입니다.

* * *

## 2. IN 연산자 사용법 :
- CULUMN에 'Data1', 'Data2'가 하나라도 일치한 값이 있으면 조회합니다.

```sql
SELECT *
FROM TABLE
WHERE CULUMN IN ('Data1', 'Data2');
```

* * *

## 3. IN 연산자 서브쿼리 사용법 :
### 3.1 IN 연산자 서브쿼리 Select문 사용 :

```sql
SELECT *
FROM TABLEA
WHERE CULUMN IN ( 
  SELECT COLUMN
  FROM TABLEB
  WHERE COLUMN = 'Data1' );
```

* * *

### 3.2 IN 연산자 서브쿼리 Update문 사용:

```sql
UPDATE
TABLEA
SET COLUMNA = 'Data1'
WHERE
   TABLEA.COLUMNA IN (
      SELECT COLUMNB
      FROM TABLEB );
```

* * *

## 3.3 NOT IN 연산자 사용법 :
- CULUMN에 'Data1', 'Data2'에 있는 값을 제외하고 조회합니다.

```sql
SELECT *
FROM TABLE
WHERE CULUMN NOT IN ('Data1', 'Data2');
```

* * *

## 4. NOT IN 연산자 서브쿼리 사용법 :
### 4.1 NOT IN 연산자 서브쿼리 Select문 사용 :

```sql
SELECT *
FROM TABLEA
WHERE CULUMN NOT IN ( 
  SELECT COLUMN
  FROM TABLEB
  WHERE COLUMN = 'Data1' );
```

* * *

### 4.2 NOT IN 연산자 서브쿼리 Update문 사용 :

```sql
UPDATE
TABLEA
SET COLUMNA = 'Data1'
WHERE
   TABLEA.COLUMNA NOT IN (
      SELECT COLUMNB
      FROM TABLEB );
```

* * *