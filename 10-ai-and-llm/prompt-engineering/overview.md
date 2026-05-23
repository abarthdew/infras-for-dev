# Prompt Engineering

## 목차
- instruction
- context
- examples
- constraints
- output format
- evaluation

## 기초 개념
Prompt engineering은 모델이 원하는 작업을 잘 수행하도록 입력을 설계하는 기술이다. 명확한 목표, 충분한 맥락, 출력 형식, 검증 기준이 중요하다.

```text
task + context + constraints + examples -> model output
```

## 간단한 예시
```text
역할: Linux 튜터
목표: 초보자에게 TCP handshake 설명
제약: 10줄 이하, 비유 1개 포함
```

## 반드시 알아야 할 질문
- prompt에 역할을 주는 것은 언제 도움이 되는가?
- 예시는 모델 출력 형식에 어떤 영향을 주는가?
- 긴 context는 항상 좋은가?
- 모델 답변은 어떻게 평가해야 하는가?
