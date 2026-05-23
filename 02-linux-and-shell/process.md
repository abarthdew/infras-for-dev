# Process

## 목차
- 프로세스와 프로그램
- PID, PPID, process tree
- fork, exec, signal
- foreground와 background
- daemon과 service

## 기초 개념
프로세스는 실행 중인 프로그램이다. 리눅스는 프로세스를 PID로 식별하고, 부모-자식 관계로 관리한다.

```text
program -> 실행 -> process
```

## 간단한 예시
```bash
ps -ef
pstree
kill -TERM 1234
```

## 반드시 알아야 할 질문
- 프로세스와 스레드는 무엇이 다른가?
- `fork`와 `exec`는 왜 함께 등장하는가?
- signal은 프로세스에게 무엇을 전달하는가?
- daemon은 일반 백그라운드 프로세스와 무엇이 다른가?
