---
title: "[DB] - IN, NOT IN 연산자 사용법"
categories:
  - DB
tags:
  - [DB, IN, NOT IN]

toc: true
toc_sticky: true

date: 2022-12-10
last_modified_at: 2022-12-10
---

## IN, NOT IN 연산자?:
- 값이 포함되는 혹은 포함되지 않는 데이터를 추출하고 싶을 때 사용하는 것이 IN, NOT IN문이다.

* * *

## IN 연산자 사용법:
- CULUMN에 'Data1', 'Data2'가 하나라도 일치한 값이 있으면 조회한다.
```sql
SELECT *
FROM TABLE
WHERE CULUMN IN ('Data1', 'Data2');
```

* * *

## IN 연산자 서브쿼리 사용법:
### IN 연산자 서브쿼리 Select문 사용:
```sql
SELECT *
FROM TABLEA
WHERE CULUMN IN ( 
  SELECT COLUMN
  FROM TABLEB
  WHERE COLUMN = 'Data1' );
```

* * *

### IN 연산자 서브쿼리 Update문 사용:
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

## NOT IN 연산자 사용법:
- CULUMN에 'Data1', 'Data2'에 있는 값을 제외하고 조회한다.
```sql
SELECT *
FROM TABLE
WHERE CULUMN NOT IN ('Data1', 'Data2');
```

* * *

## NOT IN 연산자 서브쿼리 사용법:
### NOT IN 연산자 서브쿼리 Select문 사용:
```sql
SELECT *
FROM TABLEA
WHERE CULUMN NOT IN ( 
  SELECT COLUMN
  FROM TABLEB
  WHERE COLUMN = 'Data1' );
```

* * *

### NOT IN 연산자 서브쿼리 Update문 사용:
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