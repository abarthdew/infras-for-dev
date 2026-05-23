# Git

## 목차
- repository
- commit
- branch
- merge와 rebase
- remote
- conflict

## 기초 개념
Git은 파일 변경 이력을 commit 단위로 저장하는 분산 버전 관리 시스템이다. branch는 독립적인 변경 흐름을 만들고, merge/rebase는 변경 흐름을 통합한다.

```text
working tree -> staging area -> commit -> remote
```

## 간단한 예시
```bash
git status
git add README.md
git commit -m "Update README"
git push
```

## 반드시 알아야 할 질문
- working tree와 index는 무엇이 다른가?
- commit hash는 무엇을 식별하는가?
- merge와 rebase는 무엇이 다른가?
- conflict는 왜 발생하는가?
