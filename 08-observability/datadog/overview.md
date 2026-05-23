# Datadog

## 목차
- metrics
- logs
- traces
- APM
- monitor
- integration

## 기초 개념
Datadog은 metrics, logs, traces를 한 플랫폼에서 수집하고 분석하는 SaaS 관측성 도구다. 인프라와 애플리케이션 상태를 함께 보기에 좋다.

```text
agent -> Datadog platform -> dashboard/monitor
```

## 간단한 예시
```text
service latency spike
-> trace 확인
-> 관련 log 확인
-> host metric 확인
```

## 반드시 알아야 할 질문
- APM trace는 log와 무엇이 다른가?
- monitor threshold는 어떻게 잡아야 하는가?
- tag 설계는 왜 중요한가?
- vendor lock-in은 어떤 고민을 만든다?
