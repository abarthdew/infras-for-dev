# Prometheus

## 목차
- metrics 수집
- scrape model
- exporter
- PromQL
- alert rule

## 기초 개념
Prometheus는 시스템과 애플리케이션의 metrics를 주기적으로 수집하고 질의하는 도구다. pull 방식으로 대상 endpoint를 scrape한다.

```text
target /metrics -> Prometheus -> query/alert
```

## 간단한 예시
```promql
rate(http_requests_total[5m])
```

## 반드시 알아야 할 질문
- counter, gauge, histogram은 무엇이 다른가?
- scrape interval은 어떤 영향을 주는가?
- PromQL의 `rate`는 왜 필요한가?
- alert은 어느 조건에서 울려야 하는가?
