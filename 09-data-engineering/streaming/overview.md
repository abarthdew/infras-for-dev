# Streaming

## 목차
- event stream
- producer와 consumer
- offset
- window
- delivery semantics
- late event

## 기초 개념
스트리밍은 데이터가 발생하는 즉시 또는 짧은 지연으로 계속 처리하는 방식이다. 로그 수집, 실시간 알림, 이상 탐지에 자주 쓰인다.

```text
producer -> stream -> consumer
```

## 간단한 예시
```text
order-created event
-> Kafka topic
-> inventory consumer
-> notification consumer
```

## 반드시 알아야 할 질문
- event와 message는 어떻게 구분할 수 있는가?
- offset은 consumer에게 왜 중요한가?
- exactly-once는 왜 어렵고 비싼가?
- window 집계는 왜 필요하다?
