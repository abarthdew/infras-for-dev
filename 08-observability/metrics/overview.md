# Metrics

## 목차
- metric type
- RED method
- USE method
- SLI/SLO
- cardinality

## 기초 개념
metrics는 시스템 상태를 숫자로 표현한 시계열 데이터다. 서비스 품질과 자원 상태를 빠르게 파악하는 데 유용하다.

```text
time -> metric name -> labels -> value
```

## 간단한 예시
```text
http_requests_total{method="GET", status="200"} 12345
```

## 반드시 알아야 할 질문
- request rate, error rate, duration은 왜 기본 지표인가?
- latency 평균만 보면 왜 위험한가?
- cardinality가 높으면 왜 문제가 되는가?
- SLI와 SLO는 무엇이 다른가?
