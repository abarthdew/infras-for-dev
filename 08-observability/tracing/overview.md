# Tracing

## 목차
- trace
- span
- context propagation
- distributed tracing
- sampling

## 기초 개념
트레이싱은 하나의 요청이 여러 서비스와 DB를 지나가는 흐름을 추적하는 관측성 기술이다.

```text
request -> service A span -> service B span -> DB span
```

## 간단한 예시
```text
trace id: t1
span: API handler 120ms
span: DB query 80ms
span: Redis cache 5ms
```

## 반드시 알아야 할 질문
- trace와 log는 무엇이 다른가?
- span은 어떤 단위를 나타내는가?
- context propagation은 왜 필요한가?
- sampling은 정확도와 비용 사이에서 어떤 선택인가?
