# Codex Workflow

## 목차
- 학습 질문 누적
- 파일 분류
- commit과 push
- agent identity
- 병렬 작업 주의

## 기초 개념
Codex workflow는 학습 질문과 답변을 주제 파일에 누적하고, 변경을 agent identity로 커밋/푸시해 사용자의 커밋과 구분하는 방식이다.

```text
question -> classify -> append -> commit -> push
```

## 간단한 예시
```text
Linux network 질문
-> 02-linux-and-shell/linux-networking-basics.md
-> Q&A append
-> git commit
```

## 반드시 알아야 할 질문
- 언제 새 파일을 만들고 언제 기존 파일에 append할 것인가?
- push 전 pull/rebase는 왜 필요한가?
- agent identity는 왜 사용자 identity와 분리하는가?
- 학습 노트는 어느 정도 깊이로 작성할 것인가?
