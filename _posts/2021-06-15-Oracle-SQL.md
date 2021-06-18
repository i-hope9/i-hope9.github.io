---
layout: article
title: Oracle 12c 데이터베이스 01 - 기초 지식, 실습 환경 구축
tags: [database, oracle, sql, rdb]
aside:
    toc: true
---

💡 본 글은 '실습과 함께하는! 데이터베이스 Oracle 12c' 강의 내용을 바탕으로 정리하였습니다.

## DBMS 기초
### DBMS란?
#### 데이터베이스를 관리하는 시스템
* 데이터베이스란? 테이블이 모여 이루는 데이터 단위
* 데이터 CRUD하는 시스템
* 대부분 시스템은 R(검색) > CUD(삽입, 수정, 삭제), 인덱싱과 소팅 등을 사용해 빠른 검색 가능

#### 정렬
* 빠른 검색을 위해서는 데이터가 반드시 정렬되어 있어야 함
* 퀵정렬 / 힙정렬 계열이 주로 사용됨

#### 인덱스
* 이진검색
  + `O(logN)`
  + 데이터를 CUD 할 때마다 한 가운데/왼쪽 가운데/ 오른쪽 가운데 값을 미리 계산해 놓음 → 인덱스라고 통칭함
* B-Tree 계열
  + 한 번에 두 개의 연산: 작은 값(a) 보다 작은가? 사이인가? 큰 값(b) 보다 큰가?
  + 데이터 CUD 할 때마다 a, b 값을 업데이트
* 데이터 CUD가 많이 없고 R만 많은 경우일 수록 유리. 왜냐면 인덱스도 매번 수정해줘야 해서.

#### DBMS 종류
* 계층형
* 네트워크형
* 관계형
* 객체지향
* 객체관계형
* NoSQL

### RDBMS란?
* 관계형(Relational) 데이터베이스 시스템
* 테이블 기반의 DBMS
* 각 테이블 간에는 연관관계가 있음
* 테이블을 엔티티(기본)와 릴레이션십(유도) 테이블로 구분하는 방식
  + 엔티티 테이블: 학생, 교사
  + 릴레이션십 테이블: 과목 (학생과 교사 정보가 더해져서 만들어진 테이블)
* 데이터를 테이블 단위로 관리, 중복 정보는 최소화 (동일한 데이터는 하나의 테이블에서 관리)
* 여러 테이블을 합쳐서 큰 테이블(Join) 생성해서 필요한 정보 찾아내는 방식

|용어|설명|
|:---|:---|
|스키마|DB, 테이블 정의 내역|
|SQL 쿼리| 관리형 DBMS를 사용하는 전용 질의 언어|
|기본키| PK. 하나의 레코드를 지정할 수 있는 하나 이상의 컬럼 집합|
|외래키| FK. 어떤 테이블의 기본키가 다른 테이블의 칼럼에 들어 있는 경우|
|테이블| 정보들의 묶음 단위|
|칼럼| 테이블을 구성하는 정보(속성)|
|레코드| 테이블에 들어있는 인스턴스 하나하나|
|도메인값| 각 컬럼에서 나올 수 있는 후보값|

## 오라클 설치 및 기본 사용 방법
### 도커 이용해서 설치하기
1. [도커](https://www.docker.com/get-started) 설치
2. cmd에서 `docker run -d -p 8080:8080 -p 1521:1521 truevoly/oracle-12c` 실행하여 이미지 pull하고 run
3. 2에서 pull만 한 것으로 보이면, Docker Desktop에서 pull한 컨테이너 눌러서 실행
4. 로그에 `100% complete` 뜨면 성공!

> 📌 Docker Desktop이 실행이 잘 안 된다면? [Prerequisites](https://docs.docker.com/docker-for-windows/wsl/#prerequisites) 참조하여 Linux kernel update package 설치


### 도커를 통한 오라클 컨테이너 접속
#### system 계정으로 접속
도커 컨테이너의 `CLI` 버튼 눌러서 오라클 DB에 접속 <br/>
```shell
  # sqlplus
  # user-name: system
  # password: oracle
```

#### sample schema zip 파일 복사
1. [Github](https://github.com/oracle/db-sample-schemas) 에서 zip 다운
2. 로컬 파일을 컨테이너로 복사 `docker cp db-sample-schemas-master.zip CONTAINER_NAME:/`
3. 컨테이너 접속하여 $ORACLE_HOME/demo/schema로 복사 `cp db-sample-schemas-master.zip $ORACLE_HOME/demo/schema`
4. 압축 해제 `unzip db-sample-schemas-master.zip`

## Oracle SQL 기본 사용법
### 계정 언락
사용자 이름이 scott, 비밀번호가 tiger인 계정을 unlock
``` sql
alter user scott identified by tiger account unlock
```

### 간단한 설정
``` sql
set linesize 100  # 한 줄에 몇 자 출력
set pagesize 100  # 한 페이지에 몇 줄 출력
set timing on   # SQL 쿼리를 수행한 결과 얼마나 시간이 걸렸는지 출력
```

## SQL의 이해와 종류
### SQL의 이해
#### SQL이란?
데이터베이스에 있는 필요한 정보를 사용할 수 있도록 도와주는 언어
#### SQL 종류

|구분|설명|예시|
|:---|:---|:---|
|**DML (Data Manipulation Language)**| {::nomarkdown}테이블의 데이터를 조작하는 기능</br>테이블 레코드를 CRUD{:/}| {::nomarkdown}INSERT</br>DELETE</br>UPDATE</br>SELECT</br>MERGE: 오라클에만 있는 명령어(합집합){:/}|
|**DDL (Data Definition Language)**|{::nomarkdown}DB, 테이블의 스키마를 정의, 수정하는 기능 </br> 테이블 생성, 칼럼 추가, 타입 변경 등{:/} | {::nomarkdown}CREATE</br>ALTER</br>DROP</br>TRUNCATE: 내용만 삭제</br>RENAME: 데이터베이스 객체 이름 변경{:/}|
|**TCL (Transactional Control Language)**| 트랜잭션 제어와 관련된 기능 | {::nomarkdown}COMMIT</br>ROLLBACK</br>SAVEPOINT: 롤백 시 특정한 시점까지만 수행을 롤백{:/}|
|**DCL (Data Control Language)**|{::nomarkdown}DB나 테이블 접근 권한이나 CRUD 권한을 정의하는 기능</br>특정 사용자에게 테이블의 조회 권한 허가 및 금지 등{:/}|{::nomarkdown}GRANT: 객체에 권한 부여</br>REVOKE: 객체 권한 취소{:/}|

### CRUD 이해와 실습
* CREATE
  - `INSERT INTO 테이블명(컬럼명) VALUES (값)`
* READ
  - `SELECT 컬럼명 FROM 테이블명 WHERE 조건절`
* UPDATE
  - `UPDATE 테이블명 SET 컬럼명=값, ... WEHRE 조건절`
* DELETE
  - `DELETE FROM 테이블명 WHERE 조건절`

> 📌 오라클 DDL은 바로 커밋되어 데이터베이스에 반영되지만, DML은 명시적으로 커밋을 해주어야 실제 데이터베이스에 반영됨


***
<!--more-->

