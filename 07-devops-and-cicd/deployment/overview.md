# Deployment

## 목차
- build와 release
- artifact
- environment
- rollback
- blue-green deployment
- canary deployment

## 기초 개념
배포는 개발된 애플리케이션을 사용자가 접근 가능한 환경에 안전하게 반영하는 과정이다. 핵심은 재현 가능성, 점진적 반영, 빠른 rollback이다.

```text
source -> build -> artifact -> deploy -> monitor
```

## 간단한 예시
```bash
docker build -t app:1.0 .
docker run -d -p 8080:8080 app:1.0
```

## 반드시 알아야 할 질문
- build artifact는 왜 필요한가?
- 환경별 설정은 어떻게 분리할 것인가?
- rollback은 어떤 조건에서 실행해야 하는가?
- 무중단 배포는 어떤 문제를 해결하는가?
