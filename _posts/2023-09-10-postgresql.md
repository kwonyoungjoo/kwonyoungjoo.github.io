---
layout: post
title:  postgresql 15분 리뷰
---


### postgresql란
postgresql은 오픈소스 객체 관계형 데이터 베이스이며 매우 활발하게 업데이트 되고 있는 디비 중에 하나이다.

오라클이 비용 문제로 사용하기 어렵게 되는 경우 기능이 유사한 postgresql로 마이그레이션을 많이 하고 있다.

다만 CRUD는 mysql 보다 성능이 낮아서 단순한 조회가 많은 서비스의 경우 mysql이 더 좋은 성능을 보인다.

### db engin ranking
https://db-engines.com/en/rankingskda

### postgres 장점 
- 표준 강력히 SQL 준수	CRUD 성능이 RDBMS에 비해 떨어짐 
- ACID(원자성, 일관성, 고립성, 내구성)가 뛰어남	모든 접속에 대해 일정량의 메모리가 할당되어 메모리 사용량이 많다
- 오픈소스, 안전성, 신뢰성	update 시 이전 데이터 삭제후 추가로 성능 낮음, MVCC로 주기적으로 vacumm을 실행해줘야 함 
- NoSql 데이터 유형 지원
- key-value 
- graph
- json 데이터 타입 저장 가능(json 타입으로 저장된 내용을 검색가능)
- 각종 통계함수 사용 - FIRST_VALUE, LAST_VALUE, LAG, LEAD	
- 대량 처리
- table partitioning 
- partial index(null value index) , parallel index scan (9.6부터 자동 실행)
- paraller join
``` 
create unique index T on T.c
where ( c is not null )
```

### postgres 단점
- CRUD 성능이 RDBMS에 비해 떨어짐 
- 모든 접속에 대해 일정량의 메모리가 할당되어 메모리 사용량이 많다
- update 시 이전 데이터 삭제후 추가로 성능 낮음, MVCC로 주기적으로 vacumm을 실행해줘야 함

MVCC - 다중 버젼 병행 수행(Multiversion concurrency control)로 DBMS에서 쓰기세션이 읽기세션이 서로를 블로킹하지 않고 각 세션마다 스냅샷을 보장해 동시성을 높이는 매커니즘

### mysql 과 비교
<img src="/assets/postgresql1.png" width="60%" height="90%"/>

repeatable read - A 트랜잭션이 데이터 조회후  B트랜잭션이 데이터 업데이트 하여도 A트랜잭션에서 변경전 데이터 조회가능

### postgresql 구조(single)

<img src="/assets/postgresql2.png" width="70%" height="90%"/>

- postmaster - 클라이언트 접속마다 1개의 backend process가 생성된다.
- backend process - work_men 크기의 메모리가 개별 할당되고 클라이언트의 요청을 수행
  - shared memory에 원하는 data block 여부 체크
  - 디스크 읽어서 shared memory에 캐싱 응답
  - 변경요청 작업은 shared buffer에서 처리되고 변경값은 check-pointer, bg-writer에 의해 data disk 에 동기화됨
- shared buffer - 데이터를 캐싱하는 메모리
- WAL buffer
  - 접속 세션들이 수행하는 트랜잭션 변경 로그 캐싱
  - wal buffer이 가득차면 wal files 로 저장
- statistics collector  - 통계정보 생성 
- CLOG buffer - commit log 

### postgresql 데이터 구조
<img src="/assets/postgresql3.png" width="50%" height="90%"/>

- Users/Groups - 사용자 정보 관리
- Databases - database 관리
- Tablespaces - 데이터 자장 파일 시스템 경로 관리
- Schemas - database를 논리적으로 구분하여 관리, default pulbic 스키마가 생성됨, 스키마별 별로 권한 설정
- Tables - row, column 으로 구성된 테이블이 저장되는 공간
- Views - view 테이블 관리

### postgresql 데이터 저장소
하드디스크내 데이터베이스의 정보를 폴더 구조와 파일 형태로 저장
<img src="/assets/postgresql4.png" width="70%" height="90%"/>
- global - 데이터베이스 메터정보, 데이터베이스가 공통으로 사용하는 정보
- base - 데이터베이스의 테이블등 객체, 데이터등이 저장되는 공간 /base/{database_oid}/{object_id}
- pg_tblsd - 테이블스페이지 정보 저장
- pg_xlog or pg_wal - redo 로그 개념의 wal 파일 저장
- pg_log - 로그 저장
- status directories - 트랜잭션 상태 정보 저장
- cofiguration files -  디비 설정값 정보 저장
- postmaster info files - postmaster 프로세스 정보 저장

### query 실행 순서
<img src="/assets/postgresql5.png" width="70%" height="90%"/>

### ser(role)관리
다른 RDBMS의 유저와 postgres의 role는 거의 유사한 개념,  클러스터레벨에서 공통으로 사용
```
-- 아래 두개는 같은 역활
create user test with password 'test1234'
create roll test with LOGIN password 'test1234]

-- 권한 설정
grant [SELECT|INSERT|UPDATE ..] ON [TABLE|SCHEMA..] TO ROLL_NAME
```

### schema관리
```
create schema schema_name authorization  user_name
-- 특정 스키마 테이블 조회
select * from schema_name.tablen_name
 
-- SEARCH_PATH 설정
show search_path ;
set search_path to schema_name1, schema_name2 ;
 
-- 특정 유저에 영구적으로 사용 등록
alter roll user_name set search_path = schema_name
```
- 스키마별 권한 설정, 스키마간의 join가능(데이터베이스간의 조인X)

### 권한설정
```
GRANT USAGE ON SCHEMA adstats TO 유저이름;
GRANT select, insert, update, delete ON ALL TABLES IN SCHEMA 스키마이름 TO 유저이름;
GRANT usage ON ALL SEQUENCES IN SCHEMA  스키마이름 TO 유저이름;
```