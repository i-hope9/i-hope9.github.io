---
layout: article
title: MySQ에서 `LONGVARBINARY`를 쓰고 싶다면?
aside:
    toc: false
---

로컬 환경에서는 H2 데이터베이스를 쓰다가 스테이징 환경의 MySQL에 배포하려다 배운 점들.<br/>

### MySQL BLOB, LONG VARBINARY
H2 데이터베이스에서 테이블을 생성할 때 LONGVARBINARY 타입을 사용했다. 이를 MySQL에서 그대로 생성하려고 하니 `not valid` 타입이라고 뜬다. <br/>
[MySQL 공식 문서](https://dev.mysql.com/doc/refman/8.0/en/blob.html)에서 다음의 설명을 찾을 수 있었다. <br/>
> In most respects, you can regard a BLOB column as a VARBINARY column that can be as large as you like. Similarly, you can regard a TEXT column as a VARCHAR column. BLOB and TEXT differ from VARBINARY and VARCHAR in the following ways:
> + For indexes on BLOB and TEXT columns, you must specify an index prefix length. For CHAR and VARCHAR, a prefix length is optional.
> + BLOB and TEXT columns cannot have DEFAULT values.

보아하니 `BLOB` - `VARBINARY`, `TEXT` - `VARCHAR`가 서로 유사하다고 간주하면 된다. 둘 다 원하는 만큼 큰(large) 데이터를 넣은 수 있는 칼럼이다. `BLOB` - `VARBINARY`, `TEXT` - `VARCHAR` 사이의 차이점은:
- `BLOB`, `TEXT`는 index prefix length 명시해야 함. `CHAR`, `VARCHAR`는 선택 사항.
- `BLOB`, `TEXT`는 DEFAULT 값을 가질 수 없음.

결국 우리 상황에서는 `VARBINARY` 대신 `BLOB`을 써도 상관 없었다.<br/>
그러면 `LONGVARBINARY`는 뭘까? 이것도 같은 문서에서 힌트를 얻었다.
>LONG and LONG VARCHAR map to the MEDIUMTEXT data type. This is a compatibility feature.

MySQL에서는 `LONG VARCHAR`를 `MEDIUMTEXT`로 맵핑된다. 확인해보니 `LONG VARBINARY`는 `MEDIUMBLOB`으로 맵핑된다. <br/><br/>

**결론은 MySQL에서 `LONGVARBINARY` 타입을 쓰고 싶다면 `LONG VARBINARY`로 쓰거나 혹은 `MEDIUMBLOB`으로 사용하면 된다!**

<!--more-->
### 참고자료
본문 속 링크 참조
