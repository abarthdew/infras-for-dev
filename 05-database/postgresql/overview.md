# PostgreSQL

## 목차
- PostgreSQL의 특징
- schema와 table
- index
- transaction과 MVCC
- query planner
- extension

## 기초 개념
PostgreSQL은 강력한 기능과 표준 SQL 지원을 가진 서버형 관계형 데이터베이스다. 트랜잭션, 인덱스, 동시성 제어, JSON, 확장 기능을 폭넓게 지원한다.

```text
client -> PostgreSQL server -> storage
```

## 간단한 예시
```sql
CREATE INDEX idx_orders_user_id ON orders(user_id);

EXPLAIN ANALYZE
SELECT * FROM orders WHERE user_id = 1;
```

## 반드시 알아야 할 질문
- MVCC는 읽기와 쓰기를 어떻게 동시에 처리하게 하는가?
- query planner는 어떤 기준으로 실행 계획을 고르는가?
- B-tree index는 언제 유용한가?
- connection pool은 왜 필요한가?
