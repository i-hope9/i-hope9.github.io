---
layout: article
title: Oracle ORDER BY 원하는 순서로 조회하기(CASE 식을 활용한 커스텀 정렬)
aside:
    toc: false
---

유료 부가서비스 별 월간 신청/해지 현황을 뽑는 쿼리를 작성하고 있었다. <br/>
쿼리 자체는 금방 완성했는데, 팀장님이 결과 정렬 순서를 특정 순서로 맞춰달라고 하셨다. <br/>
서비스 코드가 알파벳순도 아니고, 등록일순도 아니고... 팀장님이 원하는 **임의의 순서**였다. <br/>
이걸 어떻게 `ORDER BY`에 녹일 수 있을까🤔?

### 상황 정리
대략 이런 형태의 쿼리였다.
```sql
SELECT 
    S.SERVICE_TYPE,
    I.SERVICE_NAME,
    SUM(CASE WHEN S.USE_YN = 'Y' THEN 1 ELSE 0 END) AS REG_CNT,
    SUM(CASE WHEN S.USE_YN = 'N' THEN 1 ELSE 0 END) AS STOP_CNT
FROM SERVICE_HISTORY S
JOIN SERVICE_INFO I ON S.SERVICE_TYPE = I.SERVICE_CD
WHERE S.REQ_DATE BETWEEN '20260301' AND '20260308'
GROUP BY S.SERVICE_TYPE, I.SERVICE_NAME
;
```
기본적으로 `ORDER BY SERVICE_TYPE`을 하면 알파벳순(ABC...)으로 정렬된다. <br/>
하지만 팀장님이 원하는 순서는 `'PRE'` → `'STD'` → `'ADV'` → `'PRM'` → `'SPL'` 이런 식의, 비즈니스 로직에 따른 순서였다.

### CASE 식으로 커스텀 정렬하기
결론부터 말하면, `ORDER BY`에 `CASE` 식을 넣으면 된다!
```sql
ORDER BY CASE S.SERVICE_TYPE
             WHEN 'PRE' THEN 1
             WHEN 'STD' THEN 2
             WHEN 'ADV' THEN 3
             WHEN 'PRM' THEN 4
             WHEN 'SPL' THEN 5
         END
```
각 코드에 원하는 숫자(우선순위)를 매핑하고, 그 숫자 기준으로 정렬하는 원리다. <br/>
`CASE` 식이 각 행마다 숫자를 반환하고, `ORDER BY`는 그 숫자를 기준으로 정렬한다. 단순하지만 강력하다.

### SELECT에 별도 컬럼으로 빼는 방법도 있다
정렬 순서를 자주 바꿔야 하거나, 정렬 기준을 눈으로 확인하고 싶을 때는 SELECT 절에 별도 컬럼으로 빼두는 방법도 괜찮다.
```sql
SELECT 
    S.SERVICE_TYPE,
    I.SERVICE_NAME,
    CASE S.SERVICE_TYPE
        WHEN 'PRE' THEN 1
        WHEN 'STD' THEN 2
        WHEN 'ADV' THEN 3
        WHEN 'PRM' THEN 4
        WHEN 'SPL' THEN 5
    END AS SORT_ORDER,
    SUM(CASE WHEN S.USE_YN = 'Y' THEN 1 ELSE 0 END) AS REG_CNT,
    SUM(CASE WHEN S.USE_YN = 'N' THEN 1 ELSE 0 END) AS STOP_CNT
FROM SERVICE_HISTORY S
JOIN SERVICE_INFO I ON S.SERVICE_TYPE = I.SERVICE_CD
WHERE S.REQ_DATE BETWEEN '20260301' AND '20260308'
GROUP BY S.SERVICE_TYPE, I.SERVICE_NAME
ORDER BY SORT_ORDER
;
```
Oracle에서는 `ORDER BY`에 SELECT 절의 alias를 바로 사용할 수 있다. <br/>
다만 `SORT_ORDER` 컬럼이 결과에 노출되는 게 싫다면, 첫 번째 방식(ORDER BY에 CASE 직접 작성)을 쓰면 된다.

### DECODE로도 가능하다
Oracle에는 `CASE`와 비슷한 `DECODE` 함수가 있다. 같은 로직을 한 줄로 좀 더 간결하게 쓸 수 있다.
```sql
ORDER BY DECODE(S.SERVICE_TYPE, 'PRE', 1, 'STD', 2, 'ADV', 3, 'PRM', 4, 'SPL', 5)
```
`DECODE`는 Oracle 전용 함수이고, `CASE`는 ANSI SQL 표준이라 다른 DBMS에서도 쓸 수 있다는 차이가 있다. <br/>
이식성을 생각하면 `CASE`가 더 범용적이지만, Oracle 환경에서만 쓸 거라면 `DECODE`도 깔끔한 선택이다.

***
`ORDER BY`에는 컬럼명이나 인덱스 번호만 올 수 있다고 막연히 생각했는데, `CASE` 식처럼 값을 반환하는 표현식이면 뭐든 올 수 있다는 걸 알게 됐다. <br/>
사소한 팁이지만 알아두면 실무에서 꽤 유용하게 써먹을 수 있을 것 같다.

<!--more-->
### 참고자료
- [Oracle Docs - CASE Expressions](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/CASE-Expressions.html)
- [Oracle Docs - DECODE](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/DECODE.html)
