# SQL

## 목차
- SELECT
- WHERE와 ORDER BY
- JOIN
- GROUP BY
- INSERT, UPDATE, DELETE
- transaction

## 기초 개념
SQL은 관계형 데이터베이스에 질의하고 데이터를 변경하기 위한 선언형 언어다. "어떻게 가져올지"보다 "무엇을 원하지"를 표현한다.

```text
client -> SQL -> database engine -> result
```

## 간단한 예시
```sql
SELECT u.name, COUNT(o.id) AS order_count
FROM users u
LEFT JOIN orders o ON o.user_id = u.id
GROUP BY u.id, u.name;
```

## 반드시 알아야 할 질문
- INNER JOIN과 LEFT JOIN은 무엇이 다른가?
- GROUP BY는 언제 필요한가?
- 인덱스는 쿼리 성능에 어떤 영향을 주는가?
- 트랜잭션은 SQL 실행을 어떻게 묶는가?
