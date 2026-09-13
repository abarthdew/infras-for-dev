# Data Modeling

## 목차
- [Entity와 Relationship](#entity와-relationship)
- [Cardinality](#cardinality)
- [Normalization](#normalization)
- [Denormalization](#denormalization)
- [Constraint](#constraint)
- [Query Pattern](#query-pattern)

---

## Entity와 Relationship

데이터 모델링은 현실의 개념을 데이터 구조로 옮기는 작업이다. **Entity(개체)**는 데이터베이스에서 저장할 의미 있는 객체이고, **Relationship(관계)**는 entity 간의 연관성을 나타낸다.

```text
business concept
    ↓
entity 정의 (User, Order, Product, ...)
    ↓
entity 간 관계 설계 (1:N, N:M, ...)
    ↓
table/document 구조로 구현
    ↓
조회 패턴 최적화
```

### Entity 정의

entity와 value object를 구분하는 것이 중요하다.

**Entity**는 고유한 식별자(ID)를 가지고 독립적으로 존재한다.
- User (id가 중심, 다른 entity와 관계)
- Order (id가 중심, 변경되어도 추적 가능)
- Product (id가 중심)

**Value Object**는 속성의 조합이며 고유 ID가 없다.
- Address (주소: 거리, 도시, 우편번호의 조합)
- Money (금액: 값과 통화의 조합)
- Color (색상: RGB 값)

```sql
-- Entity: User (독립적인 ID 필요)
CREATE TABLE User (
    user_id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100)
);

-- Address는 Value Object (User에 포함)
CREATE TABLE User (
    user_id INT PRIMARY KEY,
    name VARCHAR(100),
    street VARCHAR(100),
    city VARCHAR(50),
    zip_code VARCHAR(10)
);
```

### Relationship 종류

Entity 간 관계는 cardinality(기수)로 표현된다.

```text
1:1    User -- Profile (사용자는 프로필 1개)
1:N    User -- Order (사용자는 주문 여러 개)
N:M    User -- Role (사용자는 역할 여러 개, 역할도 사용자 여러 명)
```

---

## Cardinality

**Cardinality(기수)**는 한 entity의 한 인스턴스가 다른 entity의 몇 개 인스턴스와 연관되는지를 나타낸다.

### 1:1 (One-to-One)

한 User는 정확히 하나의 Profile을 가진다.

```text
User          Profile
----          -------
user_id    -- profile_id
name          age
email         phone
```

**구현 방식:**

```sql
-- 방법 1: Profile에 user_id 추가 (가장 흔함)
CREATE TABLE Profile (
    profile_id INT PRIMARY KEY,
    user_id INT UNIQUE,  -- 1:1 보장 (UNIQUE)
    age INT,
    phone VARCHAR(20),
    FOREIGN KEY (user_id) REFERENCES User(user_id)
);

-- 방법 2: User에 profile_id 추가
CREATE TABLE User (
    user_id INT PRIMARY KEY,
    profile_id INT UNIQUE,
    FOREIGN KEY (profile_id) REFERENCES Profile(profile_id)
);
```

### 1:N (One-to-Many)

한 User가 여러 Order를 할 수 있다.

```text
User           Order
----           -----
user_id ---->  order_id
name           user_id (FK)
email          amount
               order_date
```

**구현 방식:**

```sql
CREATE TABLE "Order" (
    order_id INT PRIMARY KEY,
    user_id INT,  -- 여러 주문이 같은 user_id 가능
    amount DECIMAL(10, 2),
    order_date TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES User(user_id)
);
```

조회:

```sql
-- 특정 사용자의 모든 주문
SELECT * FROM "Order" WHERE user_id = 1;

-- 사용자별 주문 개수
SELECT user_id, COUNT(*) as order_count FROM "Order" GROUP BY user_id;
```

### N:M (Many-to-Many)

여러 User가 여러 Role을 가질 수 있다.

```text
User           Role
----           ----
user_id        role_id
name           role_name

         User_Role
         ----------
         user_id
         role_id
```

**구현 방식:**

```sql
CREATE TABLE Role (
    role_id INT PRIMARY KEY,
    role_name VARCHAR(50)
);

-- 연결 테이블 (junction table)
CREATE TABLE User_Role (
    user_id INT,
    role_id INT,
    PRIMARY KEY (user_id, role_id),
    FOREIGN KEY (user_id) REFERENCES User(user_id),
    FOREIGN KEY (role_id) REFERENCES Role(role_id)
);
```

조회:

```sql
-- 사용자 1의 역할들
SELECT r.role_name FROM User_Role ur
JOIN Role r ON ur.role_id = r.role_id
WHERE ur.user_id = 1;

-- 역할 'admin'을 가진 사용자들
SELECT u.name FROM User u
JOIN User_Role ur ON u.user_id = ur.user_id
JOIN Role r ON ur.role_id = r.role_id
WHERE r.role_name = 'admin';
```

---

## Normalization

**Normalization(정규화)**는 데이터 중복을 제거하고 데이터 일관성을 보장하는 과정이다.

### 정규화가 해결하는 문제

**중복 데이터:**

```sql
-- 정규화 전 (한 테이블에 모두)
CREATE TABLE Order (
    order_id INT,
    user_id INT,
    user_name VARCHAR(100),  -- 중복 가능
    user_email VARCHAR(100), -- 중복 가능
    product_id INT,
    product_name VARCHAR(100),  -- 중복 가능
    quantity INT
);
```

같은 사용자의 여러 주문이 있으면 user_name, user_email이 반복된다. 사용자 이름이 변경되면 모든 주문 레코드를 수정해야 한다. (Update Anomaly)

### 정규화 단계

**1NF (First Normal Form)**: 모든 속성이 atomic(더 이상 분해 불가능)한 값을 가져야 한다.

```sql
-- 1NF 위반: phone이 복합 값
CREATE TABLE User (
    user_id INT,
    name VARCHAR(100),
    phone VARCHAR(50)  -- "010-1234-5678, 02-9876-5432" (여러 번호)
);

-- 1NF 만족: phone 분리
CREATE TABLE Phone (
    phone_id INT,
    user_id INT,
    phone_number VARCHAR(20)
);
```

**2NF (Second Normal Form)**: 1NF를 만족하고, 비키 속성이 전체 키에 fully dependent해야 한다.

```sql
-- 2NF 위반 (Partial Dependency)
CREATE TABLE OrderItem (
    order_id INT,
    product_id INT,
    quantity INT,
    product_name VARCHAR(100),  -- product_id만으로 결정 (order_id와 무관)
    PRIMARY KEY (order_id, product_id)
);

-- 2NF 만족: product 정보 분리
CREATE TABLE Product (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100)
);

CREATE TABLE OrderItem (
    order_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY (order_id, product_id),
    FOREIGN KEY (product_id) REFERENCES Product(product_id)
);
```

**3NF (Third Normal Form)**: 2NF를 만족하고, 비키 속성들 간에 transitive dependency가 없어야 한다.

```sql
-- 3NF 위반 (Transitive Dependency)
CREATE TABLE Order (
    order_id INT PRIMARY KEY,
    user_id INT,
    user_city VARCHAR(50),
    city_tax_rate DECIMAL(5, 2)  -- city에만 의존 (user나 order와 직접 무관)
);

-- 3NF 만족: City 정보 분리
CREATE TABLE City (
    city_id INT PRIMARY KEY,
    city_name VARCHAR(50),
    tax_rate DECIMAL(5, 2)
);

CREATE TABLE User (
    user_id INT PRIMARY KEY,
    city_id INT,
    FOREIGN KEY (city_id) REFERENCES City(city_id)
);
```

정규화를 통해 데이터 중복을 제거하므로, 데이터 변경이 간단하고 일관성이 보장된다. 하지만 조회 시 여러 테이블을 JOIN해야 할 수 있다.

---

## Denormalization

**Denormalization(반정규화)**는 성능 최적화를 위해 의도적으로 정규화 규칙을 어기는 것이다. 조회를 빠르게 하기 위해 중복 데이터를 허용한다.

### Denormalization이 필요한 경우

```sql
-- 정규화된 구조: 조회 시 JOIN 필요
SELECT u.name, COUNT(o.order_id) as total_orders
FROM User u
LEFT JOIN "Order" o ON u.user_id = o.user_id
WHERE u.user_id = 1;

-- 반정규화된 구조: 직접 조회
CREATE TABLE User (
    user_id INT PRIMARY KEY,
    name VARCHAR(100),
    total_orders INT  -- 주문 수를 미리 저장
);
```

User 테이블에 total_orders를 저장하면, 조회가 빠르지만 주문이 추가/삭제될 때마다 이 값을 업데이트해야 한다. (유지보수 비용 증가)

### Denormalization 전략

**1. 읽기 최적화 (조회 빈번)**

```sql
-- 카테고리별 상품 개수를 매번 계산하는 것이 비쌈
CREATE TABLE Category (
    category_id INT PRIMARY KEY,
    category_name VARCHAR(50),
    product_count INT  -- 반정규화된 값
);
```

**2. 자주 함께 조회되는 데이터**

```sql
-- 주문 시 사용자 정보도 항상 함께 조회하면
CREATE TABLE "Order" (
    order_id INT PRIMARY KEY,
    user_id INT,
    user_name VARCHAR(100),  -- 반정규화: 조회 시 JOIN 불필요
    amount DECIMAL(10, 2)
);
```

**3. 보고/분석 용도**

```sql
-- 일일 판매 현황을 매번 계산하기는 비쌈
-- 별도의 집계 테이블 생성 (정규화 구조 유지)
CREATE TABLE Daily_Sales (
    date DATE,
    total_sales DECIMAL(12, 2),
    order_count INT
);
```

### Denormalization의 트레이드오프

| 정규화 | 반정규화 |
|--------|---------|
| 읽기: 느림 (JOIN 필요) | 읽기: 빠름 (직접 접근) |
| 쓰기: 빠름 (1곳에만 저장) | 쓰기: 느림 (여러 곳 수정) |
| 일관성: 높음 | 일관성: 낮음 (동기화 필요) |
| 유지보수: 쉬움 | 유지보수: 어려움 |

**선택 기준:**
- **읽기가 많다**: denormalization 검토
- **쓰기가 많다**: 정규화 유지
- **일관성이 중요하다**: 정규화 유지
- **성능이 최우선**: denormalization (단, 동기화 전략 필수)

---

## Constraint

**Constraint(제약 조건)**는 데이터의 정합성을 보장하는 규칙이다. Primary Key, Foreign Key, Not Null, Unique, Check 등이 있다.

### 종류

**Primary Key**: 각 행을 유일하게 식별한다.

```sql
CREATE TABLE User (
    user_id INT PRIMARY KEY,  -- 중복 불가, NULL 불가
    name VARCHAR(100)
);
```

**Foreign Key**: 다른 테이블과의 참조 무결성을 보장한다.

```sql
CREATE TABLE "Order" (
    order_id INT PRIMARY KEY,
    user_id INT,
    FOREIGN KEY (user_id) REFERENCES User(user_id)
);

-- user_id가 User 테이블에 없으면 insert 불가
INSERT INTO "Order" VALUES (1, 999);  -- 오류 (user_id 999 없음)
```

**Unique**: 속성의 값이 중복되지 않아야 한다.

```sql
CREATE TABLE User (
    user_id INT PRIMARY KEY,
    email VARCHAR(100) UNIQUE  -- 같은 이메일 2개 불가
);
```

**Not Null**: 속성이 반드시 값을 가져야 한다.

```sql
CREATE TABLE User (
    user_id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL  -- NULL 불가
);
```

**Check**: 조건을 만족하는 값만 저장된다.

```sql
CREATE TABLE Product (
    product_id INT PRIMARY KEY,
    price DECIMAL(10, 2) CHECK (price > 0)  -- 양수만 가능
);
```

### 애플리케이션 vs 데이터베이스 제약

**DB에 제약을 두면:**
- ✅ 데이터 무결성 강함 (직접 SQL 실행 방지)
- ❌ 복잡한 검증 규칙 구현 어려움
- ❌ 성능 오버헤드

**애플리케이션에 제약을 두면:**
- ✅ 복잡한 검증 로직 구현 가능
- ❌ 직접 DB 접근 시 우회 가능
- ❌ 여러 애플리케이션이 같은 DB 사용 시 일관성 보장 안 됨

**권장사항:**
- **기본 제약** (PK, FK, Not Null, Unique): DB에 구현
- **복잡한 비즈니스 규칙**: 애플리케이션에 구현 + DB에 최소 제약

---

## Query Pattern

좋은 데이터 모델은 실제 조회 패턴을 반영해야 한다. 모델 설계 시 "이 데이터를 어떻게 조회할 것인가"를 먼저 생각해야 한다.

### 일반적인 조회 패턴

**1. 단일 조회**

```sql
-- 사용자 1의 정보 조회
SELECT * FROM User WHERE user_id = 1;
```

**2. 관계 조회**

```sql
-- 사용자 1의 모든 주문
SELECT * FROM "Order" WHERE user_id = 1;

-- 주문 1의 상품들
SELECT p.* FROM Product p
JOIN OrderItem oi ON p.product_id = oi.product_id
WHERE oi.order_id = 1;
```

**3. 집계**

```sql
-- 사용자별 주문 총액
SELECT user_id, SUM(amount) as total
FROM "Order"
GROUP BY user_id;

-- 가장 많이 주문한 사용자 TOP 5
SELECT user_id, COUNT(*) as order_count
FROM "Order"
GROUP BY user_id
ORDER BY order_count DESC
LIMIT 5;
```

### 성능을 고려한 모델 설계

```sql
-- 나쁜 예: 조회마다 계산
SELECT o.*, (SELECT SUM(price) FROM OrderItem WHERE order_id = o.order_id) as total
FROM "Order" o;

-- 좋은 예: 미리 계산된 값 저장 (denormalization)
CREATE TABLE "Order" (
    order_id INT PRIMARY KEY,
    user_id INT,
    total_amount DECIMAL(10, 2)  -- 미리 계산해 저장
);
```

### 인덱스와의 관계

```sql
-- 자주 조회되는 조건에 인덱스 생성
CREATE INDEX idx_user_email ON User(email);
CREATE INDEX idx_order_user_id ON "Order"(user_id);

-- 인덱스가 있으면 조회 성능 향상
SELECT * FROM User WHERE email = 'test@example.com';
```

---

## 정리

좋은 데이터 모델은 다음을 균형있게 고려한다:

1. **정규화**: 중복 제거, 일관성 보장
2. **성능**: 자주 조회되는 패턴에 최적화
3. **유연성**: 미래의 변화에 대응 가능
4. **유지보수성**: 이해하기 쉬운 구조

데이터 모델링에 "정답"은 없다. 비즈니스 요구사항, 조회 패턴, 성능 목표를 함께 고려해서 최적의 모델을 설계해야 한다.
