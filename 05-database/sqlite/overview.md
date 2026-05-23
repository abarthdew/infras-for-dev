# SQLite

## 목차
- [SQLite의 특징](#sqlite의-특징)
- [파일 기반 데이터베이스](#파일-기반-데이터베이스)
- [table과 index](#table과-index)
- [transaction](#transaction)
- [사용 시나리오](#사용-시나리오)

## SQLite의 특징
SQLite는 서버 프로세스 없이 애플리케이션 안에 라이브러리처럼 포함되어 동작하는 관계형 데이터베이스다.

```text
application
-> SQLite library
-> database file
```

PostgreSQL 같은 서버형 DB는 별도의 DB 서버 프로세스에 클라이언트가 접속한다.

```text
application -> network/socket -> PostgreSQL server -> storage
```

반면 SQLite는 애플리케이션 프로세스가 직접 DB 파일을 읽고 쓴다.

```text
application process -> SQLite -> .db file
```

이 차이 때문에 SQLite는 설치와 운영이 단순하지만, 여러 서버가 동시에 접속하는 중앙 DB 용도로는 한계가 있다.

```text
SQLite      단일 파일, 내장형, 간단한 운영
PostgreSQL  서버형, 다중 사용자, 네트워크 접속
```

## 파일 기반 데이터베이스
SQLite 데이터베이스는 하나의 파일로 저장된다.

```text
app.db
```

이 파일 안에 table, index, schema, data가 함께 들어 있다. 그래서 백업은 파일 복사만으로 가능해 보이지만, 실행 중인 DB 파일을 복사할 때는 일관성을 주의해야 한다.

파일 기반 DB의 장점은 단순함이다.

```text
서버 설치 불필요
배포 쉬움
로컬 개발과 테스트에 편리
작은 앱에 적합
```

단점은 여러 프로세스나 여러 서버에서 동시에 쓰는 작업에 약하다는 점이다. SQLite는 여러 reader는 잘 처리하지만, 동시에 여러 writer가 강하게 몰리는 구조에는 적합하지 않다.

```text
읽기 많음    적합할 수 있음
동시 쓰기 많음 주의 필요
```

## table과 index
SQLite도 관계형 DB이므로 table과 index를 사용한다.

```sql
CREATE TABLE notes (
  id INTEGER PRIMARY KEY,
  title TEXT NOT NULL,
  body TEXT,
  created_at TEXT NOT NULL
);
```

조회가 자주 일어나는 컬럼에는 index를 만들 수 있다.

```sql
CREATE INDEX idx_notes_created_at ON notes(created_at);
```

index는 조회를 빠르게 하지만 쓰기 비용과 파일 크기를 증가시킨다.

```text
장점  WHERE, ORDER BY 성능 개선 가능
비용  INSERT, UPDATE, DELETE 때 index 갱신 필요
```

SQLite에서는 `EXPLAIN QUERY PLAN`으로 쿼리가 index를 사용하는지 확인할 수 있다.

```sql
EXPLAIN QUERY PLAN
SELECT *
FROM notes
WHERE created_at >= '2026-01-01';
```

## transaction
transaction은 여러 SQL 작업을 하나의 단위로 묶는다. SQLite에서도 transaction은 매우 중요하다.

```sql
BEGIN;

INSERT INTO notes (title, body, created_at)
VALUES ('hello', 'first note', '2026-05-23');

COMMIT;
```

여러 작업 중 하나가 실패하면 `ROLLBACK`으로 되돌릴 수 있다.

```sql
ROLLBACK;
```

SQLite는 단일 파일 DB지만, transaction을 통해 파일에 쓰는 중간 상태가 깨지지 않도록 관리한다.

```text
transaction 없음  중간 실패 시 애매한 상태 위험
transaction 있음  성공은 commit, 실패는 rollback
```

동시 쓰기에는 제약이 있다. SQLite는 파일 잠금을 사용하므로 writer가 많아지면 대기나 충돌이 늘 수 있다. WAL 모드를 사용하면 읽기와 쓰기 동시성을 개선할 수 있지만, 고부하 서버형 DB의 대체재가 되는 것은 아니다.

## 사용 시나리오
SQLite는 다음 상황에 잘 맞는다.

```text
로컬 개발
테스트 DB
모바일 앱 내장 DB
데스크톱 앱 설정/데이터 저장
작은 단일 서버 앱
CLI 도구의 로컬 저장소
```

반대로 다음 상황에는 PostgreSQL 같은 서버형 DB를 먼저 고려하는 것이 좋다.

```text
여러 애플리케이션 서버가 같은 DB에 접속
동시 쓰기가 많음
권한 관리와 네트워크 접근 제어가 중요
복제와 고가용성이 필요
대규모 운영 모니터링이 필요
```

정리하면 SQLite는 "작고 단순한 관계형 DB"로 매우 강력하지만, 모든 DB 운영 문제를 대신 해결하는 서버형 데이터베이스는 아니다.
