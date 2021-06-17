---
layout: article
title: Oracle 12c 데이터베이스 02 - 중복 제거, 논리 연산자, 정렬, 함수
tags: [database, oracle, sql, rdb]
aside:
    toc: true
---


💡 본 글은 '실습과 함께하는! 데이터베이스 Oracle 12c' 강의 내용을 바탕으로 정리하였습니다.

## 쿼리 결과 중복 제거
### DISTINCT 연산자
* SELECT 문 결과값에서 특정 컬럼만 출력할 때 중복 값을 제거해서 표시하는 기능

(문법)
``` sql
  SELECT DISTINCT 컬럼명1, 컬럼명2, ... FROM 테이블명 WHERE 조건절;
```

## 논리 연산자
### AND, OR, NOT
* SELECT 문 WHERE 조건절에서 사용할 수 있는 연산자

(문법)
``` sql
  SELECT * FROM 테이블명 WHERE (NOT) 조건1 AND/OR (NOT) 조건2...;
```
(예시)
``` sql
  SELECT * FROM emp WHERE job = 'salesman' AND ename = 'allen';
  SELECT * FROM emp WHERE job = 'salesman' OR job = 'clerk';
  SELECT * FROM emp WHERE job != 'clerk' AND SALARY > 1500;
```

### IN, BETWEEN
* 경우에 따라 AND, OR을 더 효율적으로 쓸 수 있는 방법

(문법)
``` sql
  SELECT * FROM 테이블명 WHERE 컬럼명 IN (조건1, 조건2, ...);
  SELECT * FROM 테이블명 WHERE 컬럼명 BETWEEN a AND b;
```

## 결과 정렬
### ORDER BY
* SELECT 문의 결과값을 특정한 칼럼을 기준으로 오름차순(ASC)/내림차순(DESC)으로 정렬해서 표시
* 기본값은 오름차순 정렬, 여러 개 컬럼을 나열하면 순서대로 정렬

(문법)
``` sql
  SELECT * FROM 테이블명 WHERE 조건절 ORDER BY 컬럼명 ASC/DESC, ...
```
(예시)
``` sql
  SELECT * FROM emp ORDER BY job ASC, sal DESC;
```

## 결과값 일부 조회
### ROWNUM
* 오라클 내부에 있는 가상 칼럼(줄번호)
* SQL 쿼리 결과 중 상위 몇 개만 보여주는 쿼리
* MySQL에서 `LIMIT`으로 쓰던 것과 유사함

(문법)
``` sql
  SELECT 컬럼명1, 컬럼명2, ... FROM 테이블명 WHERE 조건절 ROWNUM <= 숫자;
```
(예시)
``` sql
  SELECT * FROM emp WHERE ROWNUM <= 5 ORDER BY sal DESC;
```

## 함수
* SELECT 문에서 사용할 수 있는 여러 유용한 함수

(문법)
``` sql
  SELECT function(컬럼명1) FROM 테이블명 WHERE 조건절;
```

### 집합함수
* 테이블의 전체레코드를 대상으로 특정 칼럼을 적용해서 한 개의 값을 리턴하는 함수

|함수|설명|
|:---|:---|
|`count()`|레코드의 개수를 리턴하는 함수|
|`sum()/avg()`|칼럼값의 합/평균을 리턴|
|`min()/max()`|칼럼값의 최소/최대값을 리턴|

### 그 외 유용한 함수

|함수|설명|
|:---|:---|:---|
|`length()`|레코드의 문자열 컬럼의 글자수 리턴|
|`substr()`|문자열의 중간 부분을 리턴|
|`upper()/lower()`|영문자의 경우, 대/소문자로 리턴|
|`round()`|레코드의 숫자 컬럼값을 반올림하여 리턴|

(예시)
``` sql
  SELECT upper(substr(ename, 1, 3)) FROM emp;
  SELECT ename, round(sal / 10) FROM emp;
```
***
<!--more-->

