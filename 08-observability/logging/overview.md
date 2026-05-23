# Logging

## 목차
- log level
- structured logging
- correlation id
- log aggregation
- retention

## 기초 개념
로그는 시스템에서 발생한 사건을 시간 순서로 남긴 기록이다. 장애 분석, 감사, 디버깅에 필요하지만 너무 많으면 비용과 소음이 된다.

```text
application -> log -> collector -> storage -> search
```

## 간단한 예시
```json
{
  "level": "ERROR",
  "requestId": "abc-123",
  "message": "failed to create order"
}
```

## 반드시 알아야 할 질문
- DEBUG, INFO, WARN, ERROR는 어떻게 구분하는가?
- structured logging은 왜 검색에 유리한가?
- correlation id는 분산 시스템에서 왜 필요한가?
- 민감 정보는 로그에 왜 남기면 안 되는가?
