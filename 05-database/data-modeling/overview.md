# Data Modeling

## 목차
- entity와 relationship
- cardinality
- normalization
- denormalization
- constraint
- query pattern

## 기초 개념
데이터 모델링은 현실의 개념을 데이터 구조로 옮기는 작업이다. 좋은 모델은 데이터 중복, 정합성, 조회 패턴, 변경 가능성을 함께 고려한다.

```text
business concept -> entity -> table/document -> query
```

## 간단한 예시
```text
User 1 --- N Order
Order 1 --- N OrderItem
Product 1 --- N OrderItem
```

## 반드시 알아야 할 질문
- entity와 value object는 어떻게 구분할 수 있는가?
- 정규화는 어떤 중복을 줄이는가?
- denormalization은 언제 필요한가?
- 제약 조건은 애플리케이션과 DB 중 어디에 둘 것인가?
