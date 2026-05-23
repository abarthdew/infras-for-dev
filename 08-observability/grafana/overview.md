# Grafana

## 목차
- dashboard
- data source
- panel
- variable
- alert

## 기초 개념
Grafana는 Prometheus, Elasticsearch, PostgreSQL 같은 데이터 소스를 시각화하는 대시보드 도구다.

```text
data source -> query -> panel -> dashboard
```

## 간단한 예시
```text
CPU usage panel
Memory usage panel
HTTP latency panel
Error rate panel
```

## 반드시 알아야 할 질문
- 좋은 dashboard는 어떤 질문에 답해야 하는가?
- panel은 metrics를 어떻게 오해하게 만들 수 있는가?
- variable은 언제 유용한가?
- alert과 dashboard는 역할이 어떻게 다른가?
