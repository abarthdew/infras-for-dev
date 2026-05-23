# Database Theory

## 목차
- 데이터베이스가 해결하는 문제
- 테이블, 행, 열
- 키와 제약 조건
- 인덱스
- 트랜잭션과 ACID
- 정규화와 모델링
- 동시성 제어
- 쿼리 실행 계획

## 기초 개념
데이터베이스는 데이터를 안전하고 일관성 있게 저장하고 조회하기 위한 시스템이다. 단순 파일 저장과 달리, 여러 사용자가 동시에 접근해도 데이터가 깨지지 않도록 구조, 제약 조건, 트랜잭션, 인덱스 같은 기능을 제공한다.

```text
애플리케이션
-> SQL
-> 데이터베이스 엔진
-> 메모리/디스크
```

관계형 데이터베이스를 배울 때 먼저 잡아야 할 개념은 다음과 같다.

```text
table       같은 형태의 데이터를 모아 둔 구조
row         하나의 데이터 기록
column      데이터의 속성
primary key 행을 식별하는 대표 키
foreign key 다른 테이블과의 관계를 나타내는 키
index       조회를 빠르게 하기 위한 자료구조
transaction 여러 작업을 하나의 논리적 단위로 묶는 것
```

## 간단한 예시
사용자와 주문을 저장한다고 하자.

```sql
CREATE TABLE users (
  id BIGINT PRIMARY KEY,
  name TEXT NOT NULL
);

CREATE TABLE orders (
  id BIGINT PRIMARY KEY,
  user_id BIGINT NOT NULL,
  total_price INTEGER NOT NULL
);
```

`orders.user_id`는 주문이 어떤 사용자에게 속하는지 나타낸다. 실제 설계에서는 foreign key 제약 조건을 걸어 잘못된 관계가 들어가지 않게 할 수 있다.

```sql
SELECT *
FROM orders
WHERE user_id = 1;
```

이 조회가 자주 실행된다면 `orders.user_id`에 인덱스를 추가할 수 있다.

## 반드시 알아야 할 질문
- 파일 저장과 데이터베이스는 무엇이 다른가?
- primary key는 왜 필요한가?
- foreign key는 데이터 정합성에 어떤 도움을 주는가?
- 인덱스는 왜 조회를 빠르게 하지만 쓰기는 느리게 만들 수 있는가?
- 트랜잭션은 왜 필요한가?
- ACID는 각각 무엇을 보장하는가?
- 정규화는 언제 도움이 되고 언제 복잡해지는가?
