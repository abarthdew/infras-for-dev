# PostgreSQL

## 목차
- [PostgreSQL의 특징](#postgresql의-특징)
- [schema와 table](#schema와-table)
- [index](#index)
- [transaction과 MVCC](#transaction과-mvcc)
- [query planner](#query-planner)
- [extension](#extension)

## PostgreSQL의 특징
PostgreSQL은 서버형 관계형 데이터베이스다. 애플리케이션은 PostgreSQL 서버에 연결하고, 서버가 SQL 실행, 트랜잭션, 저장, 동시성 제어를 담당한다.

```text
application
-> PostgreSQL server
-> memory / WAL / data files
```

PostgreSQL은 표준 SQL 지원뿐 아니라 JSON, window function, full-text search, extension 등 확장성이 강하다.

```text
관계형 데이터
트랜잭션
인덱스
JSONB
확장 기능
```

서버형 DB이므로 connection 관리가 중요하다. 애플리케이션 요청마다 새 DB 연결을 만들면 비용이 크다. 그래서 connection pool을 사용한다.

```text
application threads
-> connection pool
-> limited PostgreSQL connections
```

## schema와 table
schema는 데이터베이스 안에서 table, view, function 같은 객체를 묶는 namespace다.

```text
database
└── schema public
    ├── users table
    └── orders table
```

table은 행과 열로 데이터를 저장하는 구조다.

```sql
CREATE TABLE users (
  id BIGSERIAL PRIMARY KEY,
  email TEXT NOT NULL UNIQUE,
  name TEXT NOT NULL
);
```

schema를 사용하면 같은 DB 안에서도 도메인이나 용도별로 객체를 나눌 수 있다.

```sql
CREATE SCHEMA billing;
CREATE TABLE billing.invoices (
  id BIGSERIAL PRIMARY KEY,
  total_amount INTEGER NOT NULL
);
```

기본 schema는 보통 `public`이다. 처음에는 `public`만 사용해도 충분하지만, 규모가 커지면 schema 분리를 고려할 수 있다.

## index
index는 테이블 조회를 빠르게 하기 위한 자료구조다. PostgreSQL에서 가장 기본적인 인덱스는 B-tree index다.

```sql
CREATE INDEX idx_orders_user_id ON orders(user_id);
```

B-tree index는 동등 비교와 범위 검색에 널리 유용하다.

```sql
SELECT *
FROM orders
WHERE user_id = 1;
```

```sql
SELECT *
FROM orders
WHERE created_at >= '2026-01-01';
```

하지만 인덱스는 쓰기 비용과 저장 공간을 증가시킨다. `INSERT`, `UPDATE`, `DELETE` 때 테이블뿐 아니라 인덱스도 갱신해야 한다.

```text
읽기 성능 개선 가능
쓰기 비용 증가
디스크 공간 사용
```

PostgreSQL에는 B-tree 외에도 GIN, GiST, BRIN 같은 인덱스가 있다. JSONB 검색이나 full-text search에는 GIN index가 자주 등장한다.

## transaction과 MVCC
transaction은 여러 SQL 작업을 하나의 논리적 단위로 묶는다.

```sql
BEGIN;
UPDATE accounts SET balance = balance - 10000 WHERE id = 1;
UPDATE accounts SET balance = balance + 10000 WHERE id = 2;
COMMIT;
```

PostgreSQL은 MVCC(Multi-Version Concurrency Control)를 사용한다. MVCC는 데이터를 수정할 때 기존 행을 바로 덮어쓰기보다 여러 버전의 행을 관리해 읽기와 쓰기의 충돌을 줄인다.

```text
transaction A  이전 버전 읽기
transaction B  새 버전 쓰기
```

MVCC 덕분에 어떤 트랜잭션이 데이터를 읽는 동안 다른 트랜잭션이 같은 데이터를 수정해도, 읽는 쪽은 자기 시점에 맞는 일관된 버전을 볼 수 있다.

```text
읽기 작업이 쓰기 작업을 항상 막지 않음
쓰기 작업도 읽기 작업을 항상 막지 않음
```

단, 오래 열린 transaction은 죽은 행 버전 정리를 방해할 수 있다. 그래서 transaction은 필요한 범위에서 짧게 유지하는 것이 좋다.

## query planner
query planner는 SQL을 실제로 어떻게 실행할지 결정하는 PostgreSQL 내부 구성 요소다.

```sql
EXPLAIN ANALYZE
SELECT *
FROM orders
WHERE user_id = 1;
```

planner는 테이블 통계, 인덱스, 조건 선택도, JOIN 비용을 고려해 실행 계획을 고른다.

```text
Seq Scan
Index Scan
Nested Loop
Hash Join
Sort
Aggregate
```

실행 계획을 볼 때는 예상 rows와 실제 rows 차이를 확인해야 한다. 차이가 크면 통계가 부정확하거나 조건 분포가 planner 예상과 다를 수 있다.

```text
느린 쿼리 분석
-> EXPLAIN ANALYZE
-> scan 방식 확인
-> join 방식 확인
-> rows estimate 확인
-> index 필요성 판단
```

무조건 인덱스를 추가하는 것보다, 실행 계획을 보고 병목을 확인하는 습관이 중요하다.

## extension
PostgreSQL extension은 데이터베이스 기능을 확장하는 모듈이다.

```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;
```

자주 접하는 extension 예시는 다음과 같다.

```text
pgcrypto    암호화와 UUID 생성 함수
uuid-ossp   UUID 생성
postgis     지리 공간 데이터 처리
pg_trgm     문자열 유사도 검색
```

extension은 강력하지만 운영 환경에서는 설치 가능 여부, 권한, 백업/복구 호환성을 확인해야 한다.

```text
개발 DB에서 사용 가능
운영 DB에도 설치되어 있는가?
관리형 DB에서 허용되는가?
마이그레이션에 포함되는가?
```

PostgreSQL의 강점 중 하나는 이렇게 핵심 DB 기능 위에 확장을 얹어 다양한 사용 사례를 처리할 수 있다는 점이다.
