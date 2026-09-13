# SQL

## 목차
- [SELECT](#select)
- [WHERE와 ORDER BY](#where와-order-by)
- [JOIN](#join)
- [GROUP BY](#group-by)
- [INSERT, UPDATE, DELETE](#insert-update-delete)
- [transaction](#transaction)

## SELECT
`SELECT`는 테이블에서 원하는 데이터를 조회하는 SQL 문이다. SQL은 선언형 언어라서 "어떤 데이터를 원한다"를 표현하고, 데이터베이스 엔진이 실제 실행 방법을 결정한다.

```sql
SELECT id, name
FROM users;
```

모든 컬럼을 가져올 때는 `*`를 사용할 수 있지만, 실무에서는 필요한 컬럼만 명시하는 편이 좋다.

```sql
SELECT *
FROM users;
```

필요한 컬럼만 가져오면 네트워크 전송량과 애플리케이션 처리량을 줄일 수 있고, 쿼리 의도도 더 명확하다.

```sql
SELECT id, email
FROM users;
```

## WHERE와 ORDER BY
`WHERE`는 조건에 맞는 행만 필터링한다.

```sql
SELECT id, name
FROM users
WHERE active = true;
```

`ORDER BY`는 결과 정렬을 담당한다.

```sql
SELECT id, name, created_at
FROM users
ORDER BY created_at DESC;
```

조건과 정렬은 인덱스와 성능에 큰 영향을 준다. 예를 들어 `WHERE user_id = ?` 같은 조건이 자주 쓰이면 해당 컬럼 인덱스가 도움이 될 수 있다.

```sql
CREATE INDEX idx_orders_user_id ON orders(user_id);
```

인덱스는 조회를 빠르게 할 수 있지만 쓰기 비용과 저장 공간을 증가시킨다. 그래서 쿼리 패턴을 보고 필요한 곳에 만들어야 한다.

## JOIN
`JOIN`은 여러 테이블의 데이터를 관계에 따라 함께 조회하는 문법이다.

```sql
SELECT users.name, orders.total_price
FROM users
JOIN orders ON orders.user_id = users.id;
```

`INNER JOIN`은 양쪽에 매칭되는 행이 있을 때만 결과에 포함한다.

```text
users에 있음 + orders에 있음 -> 결과 포함
users에 있음 + orders에 없음 -> 결과 제외
```

`LEFT JOIN`은 왼쪽 테이블의 행은 유지하고, 오른쪽에 매칭이 없으면 `NULL`로 채운다.

```sql
SELECT users.name, orders.id AS order_id
FROM users
LEFT JOIN orders ON orders.user_id = users.id;
```

```text
주문이 없는 사용자도 결과에 포함하고 싶다 -> LEFT JOIN
주문이 있는 사용자만 보고 싶다 -> INNER JOIN
```

JOIN은 강력하지만 데이터 양이 많아지면 비용이 커질 수 있다. JOIN 조건 컬럼에 인덱스가 필요한 경우가 많다.

## GROUP BY
`GROUP BY`는 여러 행을 그룹으로 묶고 집계할 때 사용한다.

```sql
SELECT user_id, COUNT(*) AS order_count
FROM orders
GROUP BY user_id;
```

주문 테이블에서 사용자별 주문 수를 구하는 예시다.

```text
orders rows
-> user_id별 그룹
-> COUNT 집계
```

집계 함수는 다음과 같이 자주 쓰인다.

```text
COUNT  개수
SUM    합계
AVG    평균
MIN    최솟값
MAX    최댓값
```

`WHERE`와 `HAVING`도 구분해야 한다.

```text
WHERE   그룹화 전 행 필터링
HAVING  그룹화 후 집계 결과 필터링
```

```sql
SELECT user_id, COUNT(*) AS order_count
FROM orders
GROUP BY user_id
HAVING COUNT(*) >= 3;
```

## INSERT, UPDATE, DELETE
`INSERT`는 새 행을 추가한다.

```sql
INSERT INTO users (id, name)
VALUES (1, 'shin');
```

`UPDATE`는 기존 행을 수정한다.

```sql
UPDATE users
SET name = 'kim'
WHERE id = 1;
```

`DELETE`는 행을 삭제한다.

```sql
DELETE FROM users
WHERE id = 1;
```

`UPDATE`와 `DELETE`에서 `WHERE`를 빠뜨리면 많은 데이터를 한 번에 바꿀 수 있으므로 매우 조심해야 한다.

```sql
-- 위험: 모든 사용자 이름이 바뀐다
UPDATE users
SET name = 'unknown';
```

데이터 변경 SQL은 제약 조건, 트랜잭션, rollback 가능성을 함께 생각해야 한다.

## transaction
transaction은 여러 SQL 문을 하나의 논리적 작업으로 묶는다. 모두 성공하면 `COMMIT`, 중간에 실패하면 `ROLLBACK`한다.

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 10000
WHERE id = 1;

UPDATE accounts
SET balance = balance + 10000
WHERE id = 2;

COMMIT;
```

계좌 이체처럼 여러 변경이 함께 성공해야 하는 작업에서 transaction이 필요하다.

```text
출금 성공 + 입금 성공 -> commit
출금 성공 + 입금 실패 -> rollback
```

transaction은 SQL 실행을 하나의 단위로 묶어 데이터가 중간 상태에 머물지 않게 한다. 동시에 여러 사용자가 데이터를 수정할 때 isolation 수준도 중요해진다.

```text
Atomicity    전부 성공하거나 전부 실패
Isolation    동시에 실행되는 작업 간 간섭 제어
Durability   commit된 결과 보존
```
