# Process

## 목차
- [프로세스와 프로그램](#프로세스와-프로그램)
- [PID, PPID, process tree](#pid-ppid-process-tree)
- [fork, exec, signal](#fork-exec-signal)
- [foreground와 background](#foreground와-background)
- [daemon과 service](#daemon과-service)

## 프로세스와 프로그램
프로그램은 디스크에 저장된 실행 가능한 파일이고, 프로세스는 그 프로그램이 메모리에 올라와 실행 중인 상태다.

```text
program file
-> 실행
-> process
```

예를 들어 `/bin/ls`는 프로그램 파일이다. 터미널에서 `ls`를 입력하면 셸이 `/bin/ls`를 실행하고, 그 순간 `ls` 프로세스가 생긴다.

```bash
which ls
ls
```

프로세스는 실행 중에 자기만의 메모리 공간, 열린 file descriptor, 환경 변수, 현재 작업 디렉토리 같은 실행 상태를 가진다.

```text
process
├── PID
├── memory
├── file descriptors
├── environment variables
└── current working directory
```

프로세스와 스레드도 구분해야 한다. 프로세스는 자원 소유 단위에 가깝고, 스레드는 그 안에서 실행되는 흐름이다.

```text
process = 실행 중인 프로그램과 자원 묶음
thread  = 프로세스 안의 실행 흐름
```

## PID, PPID, process tree
PID는 프로세스 ID이고, PPID는 부모 프로세스 ID다. 리눅스에서 프로세스는 보통 다른 프로세스가 만든다. 그래서 부모-자식 관계가 생긴다.

```bash
ps -ef
```

출력에는 보통 PID와 PPID가 함께 나온다.

```text
UID   PID   PPID  CMD
root    1      0  systemd
shin 1200      1  sshd
shin 1300   1200  bash
```

process tree는 이 관계를 트리처럼 보는 것이다.

```bash
pstree
```

구조는 대략 다음과 같다.

```text
systemd
├── sshd
│   └── bash
│       └── vim
└── nginx
    ├── nginx worker
    └── nginx worker
```

프로세스 트리를 보면 어떤 명령이 어디서 실행되었는지, 어떤 서비스가 어떤 자식 프로세스를 만들었는지 이해하기 쉽다.

## fork, exec, signal
`fork`는 현재 프로세스를 복제해 자식 프로세스를 만드는 시스템 콜이다. `exec`는 현재 프로세스의 프로그램 내용을 다른 실행 파일로 바꾸는 시스템 콜 계열이다.

셸에서 명령어를 실행하면 보통 다음 흐름이 일어난다.

```text
shell
-> fork로 자식 프로세스 생성
-> 자식 프로세스에서 exec로 명령어 실행
-> 부모 shell은 기다리거나 다음 명령을 받음
```

예를 들어 `ls`를 실행할 때 셸 자체가 `ls`로 바뀌는 것이 아니다. 셸은 자식을 만들고, 그 자식이 `ls`가 된다.

signal은 프로세스에게 보내는 짧은 제어 메시지다.

```text
SIGINT   Ctrl-C로 자주 발생하는 인터럽트
SIGTERM  정상 종료 요청
SIGKILL  강제 종료
```

예를 들어 다음 명령은 PID 1234 프로세스에 종료 요청을 보낸다.

```bash
kill -TERM 1234
```

`SIGTERM`은 정리하고 종료하라는 요청이고, `SIGKILL`은 프로세스가 처리하거나 무시할 수 없는 강제 종료다.

## foreground와 background
foreground 프로세스는 현재 터미널 입력을 차지하고 실행되는 프로세스다.

```bash
sleep 10
```

이 명령을 실행하면 10초 동안 같은 터미널에서 다음 명령을 입력하기 어렵다.

background 프로세스는 터미널 뒤에서 실행된다.

```bash
sleep 10 &
```

`&`를 붙이면 셸은 프로세스를 백그라운드로 보내고 곧바로 다음 명령을 받을 수 있다.

```text
foreground  터미널 앞에서 실행
background  터미널 뒤에서 실행
```

하지만 백그라운드 실행은 출력 숨김과 다르다. 백그라운드 프로세스도 stdout/stderr가 터미널에 연결되어 있으면 화면에 출력할 수 있다.

```bash
long-job > app.log 2>&1 &
```

이렇게 써야 백그라운드 실행과 출력 리다이렉션을 함께 처리할 수 있다.

## daemon과 service
daemon은 백그라운드에서 오래 실행되며 특정 기능을 제공하는 프로세스다.

```text
sshd    SSH 접속 처리
nginx   HTTP 요청 처리
cron    예약 작업 실행
```

일반 백그라운드 프로세스와 daemon은 다르다. `sleep 100 &`는 백그라운드 프로세스지만 보통 daemon이라고 부르지는 않는다. daemon은 시스템 서비스로 지속 실행되도록 설계된 프로세스다.

service는 systemd 같은 서비스 관리자가 관리하는 실행 단위다.

```bash
systemctl status nginx
systemctl restart ssh
```

정리하면 다음과 같다.

```text
process     실행 중인 프로그램
daemon      백그라운드에서 지속적으로 기능을 제공하는 프로세스
service     systemd가 관리하는 실행 단위
systemctl   service를 제어하는 명령어
```
