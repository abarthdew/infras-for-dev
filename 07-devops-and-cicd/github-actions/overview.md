# GitHub Actions

## 목차
- workflow
- event
- job
- step
- runner
- artifact

## 기초 개념
GitHub Actions는 GitHub 이벤트를 기반으로 테스트, 빌드, 배포 같은 자동화 작업을 실행하는 CI/CD 도구다.

```text
push event -> workflow -> job -> step
```

## 간단한 예시
```yaml
name: CI
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
```

## 반드시 알아야 할 질문
- workflow와 job은 무엇이 다른가?
- runner는 어디서 실행되는가?
- secret은 왜 필요하고 어떻게 보호되는가?
- CI와 CD는 무엇이 다른가?
