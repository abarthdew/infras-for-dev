# Linux System Operations

## Q&A: 프로세스, 데몬, systemctl, tmux, 리버스 프록시

### 질문
1. 프로세스는 현재 돌아가는 프로그램(함수 콜백 등), daemon은 보통 백그라운드에서 도는 주요 배치 프로그램인가?
2. `systemctl`은 무엇인가?
3. `tmux`는 무엇인가?
4. 리버스 프록시는 사용자 요청과 웹서버 사이에 인터페이스를 하나 더 둔 것인가? nginx처럼 보안이나 로드 밸런싱 같은 것을 담당하도록?

### 답변
## 1. 프로세스와 데몬

**프로세스(process)** 는 실행 중인 프로그램이다.

예를 들어 디스크에 있는 `/bin/ls`는 실행 파일이고, 사용자가 `ls`를 실행하면 메모리 위에 실행 중인 `ls` 프로세스가 생긴다.

```bash
ls
```

정리하면 다음과 같다.

```text
프로그램 = 디스크에 저장된 실행 가능한 파일
프로세스 = 실행 중인 프로그램의 인스턴스
```

여기서 "함수 콜백"은 프로세스 자체라기보다는, 어떤 프로세스 안에서 실행되는 코드 흐름에 가깝다. 예를 들어 Node.js 서버 프로세스 안에서 콜백 함수들이 실행될 수는 있지만, 콜백 하나하나를 보통 OS 프로세스라고 부르지는 않는다.

**데몬(daemon)** 은 보통 백그라운드에서 오래 실행되며 특정 서비스를 제공하는 프로세스다.

```text
sshd    SSH 접속을 받아주는 데몬
nginx   HTTP 요청을 받아주는 웹 서버 데몬
cron    정해진 시간에 작업을 실행하는 데몬
mysqld  MySQL 서버 데몬
```

데몬은 백그라운드에서 돈다는 점이 중요하지만, 모든 백그라운드 프로세스가 데몬인 것은 아니다.

```bash
sleep 100 &
```

위 명령은 백그라운드 프로세스지만, 보통 데몬이라고 부르지는 않는다. 그냥 사용자가 뒤에서 실행시킨 임시 프로세스에 가깝다.

또 데몬을 "주요 배치 프로그램"이라고 이해하면 조금 좁다. 배치 작업을 담당하는 데몬도 있지만, 데몬은 배치보다 넓은 개념이다. 웹 서버, SSH 서버, DB 서버처럼 계속 대기하면서 요청을 처리하는 서비스도 데몬이다.

```text
프로세스   실행 중인 프로그램
백그라운드 터미널 앞을 차지하지 않고 뒤에서 실행되는 방식
데몬       백그라운드에서 오래 실행되며 서비스를 제공하는 프로세스
배치       정해진 작업을 한 번 또는 주기적으로 처리하는 작업 방식
```

## 2. systemctl

`systemctl`은 리눅스에서 **systemd가 관리하는 서비스들을 제어하는 명령어**다.

요즘 많은 리눅스 배포판은 부팅, 서비스 실행, 데몬 관리, 로그 연동 등을 `systemd`라는 시스템 매니저가 담당한다. `systemctl`은 그 systemd에게 명령을 내리는 도구다.

예를 들어 nginx 서비스를 시작하려면 다음처럼 쓴다.

```bash
sudo systemctl start nginx
```

서비스를 멈추려면 다음처럼 쓴다.

```bash
sudo systemctl stop nginx
```

상태를 확인하려면 다음처럼 쓴다.

```bash
systemctl status nginx
```

서버가 부팅될 때 자동으로 nginx가 켜지게 하려면 `enable`을 사용한다.

```bash
sudo systemctl enable nginx
```

자동 시작을 끄려면 `disable`을 사용한다.

```bash
sudo systemctl disable nginx
```

정리하면 다음과 같다.

```text
systemd    리눅스 시스템과 서비스를 관리하는 시스템 매니저
systemctl  systemd에게 명령을 내리는 CLI 도구
service    systemd가 관리하는 데몬 또는 백그라운드 작업 단위
```

자주 쓰는 명령은 다음과 같다.

```bash
systemctl status nginx
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl enable nginx
sudo systemctl disable nginx
```

여기서 `start`와 `enable`은 다르다.

```text
start   지금 즉시 실행
enable  다음 부팅부터 자동 실행되도록 등록
```

## 3. tmux

`tmux`는 터미널 멀티플렉서(terminal multiplexer)다. 쉽게 말하면 **하나의 SSH 터미널 안에서 여러 터미널 세션을 만들고, 접속이 끊겨도 작업을 계속 유지하게 해주는 도구**다.

SSH로 원격 서버에 접속해서 긴 작업을 실행한다고 해보자.

```bash
ssh ubuntu@server
```

그 상태에서 오래 걸리는 작업을 실행했다.

```bash
./deploy.sh
```

그런데 네트워크가 끊기면 일반적으로 SSH 세션이 종료되고, 그 안에서 실행 중이던 작업도 같이 영향을 받을 수 있다. `tmux`를 쓰면 작업을 tmux 세션 안에 띄워두고, SSH 연결이 끊겨도 나중에 다시 붙을 수 있다.

기본 사용 예시는 다음과 같다.

```bash
tmux new -s work
```

`work`라는 이름의 tmux 세션을 만든다. 그 안에서 작업을 실행한다.

```bash
./long-running-task.sh
```

세션에서 빠져나오려면 보통 다음 키를 누른다.

```text
Ctrl-b 를 누른 뒤 d
```

이것을 detach라고 한다. 작업은 계속 실행된다.

나중에 다시 붙으려면 다음처럼 한다.

```bash
tmux attach -t work
```

세션 목록은 다음처럼 본다.

```bash
tmux ls
```

정리하면 다음과 같다.

```text
tmux session   유지되는 터미널 작업 공간
detach         세션은 살려두고 빠져나오기
attach         살아 있는 세션에 다시 접속하기
```

`tmux`는 서버 관리, 배포, 로그 모니터링, 긴 작업 실행에서 유용하다. 다만 운영 배포를 tmux에만 의존하는 것은 좋은 자동화 방식은 아니다. 실무에서는 systemd, CI/CD, Docker, Kubernetes 같은 방식과 함께 이해하는 것이 좋다.

## 4. 리버스 프록시와 nginx

리버스 프록시(reverse proxy)는 사용자의 요청을 실제 애플리케이션 서버 앞에서 먼저 받아서, 적절한 내부 서버로 전달하는 서버다.

```text
사용자 -> 리버스 프록시 -> 애플리케이션 서버
```

예를 들어 사용자는 `https://example.com`으로 요청을 보낸다. 이 요청을 nginx가 먼저 받고, 내부의 Spring Boot나 Node.js 서버로 넘길 수 있다.

```text
브라우저
-> nginx
-> localhost:8080의 Spring Boot 앱
```

nginx 설정은 대략 이런 형태가 될 수 있다.

```nginx
server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://127.0.0.1:8080;
    }
}
```

이 경우 사용자는 `127.0.0.1:8080`에 직접 접속하지 않는다. 사용자는 nginx에 요청하고, nginx가 내부 앱 서버로 요청을 전달한다.

리버스 프록시가 하는 대표적인 역할은 다음과 같다.

```text
TLS/HTTPS 처리
로드 밸런싱
요청 라우팅
정적 파일 서빙
압축
캐싱
보안 헤더 추가
Rate limiting
내부 서버 주소 숨기기
```

그러나 리버스 프록시를 단순히 "보안을 담당하는 것"으로만 이해하면 부족하다. 리버스 프록시는 보안 기능도 할 수 있지만, 핵심은 **클라이언트 요청을 받아 내부 서버로 대신 전달하는 중간 서버**라는 점이다.

프록시와 리버스 프록시의 차이도 알아두면 좋다.

```text
포워드 프록시  클라이언트 앞에 서서 클라이언트를 대신해 외부로 나간다
리버스 프록시  서버 앞에 서서 서버를 대신해 요청을 받는다
```

예를 들어 회사 내부에서 인터넷 접속을 통제하는 프록시는 포워드 프록시에 가깝다.

```text
내 PC -> 포워드 프록시 -> 인터넷
```

반면 nginx가 여러 웹 서버 앞에서 요청을 받는 구조는 리버스 프록시다.

```text
사용자 -> nginx -> app1/app2/app3
```

로드 밸런싱을 하면 다음처럼 여러 서버 중 하나로 요청을 보낼 수 있다.

```text
사용자 -> nginx -> app1
              -> app2
              -> app3
```

정리하면 다음과 같다.

```text
프로세스        실행 중인 프로그램
데몬            백그라운드에서 오래 실행되며 서비스를 제공하는 프로세스
systemd         리눅스 서비스 관리자
systemctl       systemd를 제어하는 명령어
tmux            터미널 세션을 유지하고 나눠 쓰는 도구
리버스 프록시   사용자 요청을 앞에서 받아 내부 서버로 전달하는 서버
nginx           웹 서버이자 리버스 프록시로 자주 쓰이는 프로그램(웹서버, reverse proxy, load balancer 다 수행 가능)
```

사용자의 이해를 고쳐 말하면 다음과 같다.

```text
프로세스는 실행 중인 프로그램이다. 함수 콜백 자체는 보통 프로세스가 아니다.
데몬은 단순 배치 프로그램이 아니라, 백그라운드에서 지속적으로 서비스를 제공하는 프로세스다.
systemctl은 그런 데몬/서비스를 시작, 중지, 재시작, 자동 실행 설정하는 도구다.
tmux는 SSH 세션 안에서 작업 공간을 유지하고 여러 터미널을 다루게 해주는 도구다.
리버스 프록시는 사용자와 앱 서버 사이에서 요청을 받아 전달하고, nginx가 그 역할을 자주 맡는다.
```

전체 연결해서 보면
```
사용자 요청
↓
Nginx(reverse proxy daemon)
↓
Application process
↓
systemd가 관리
↓
SSH로 원격 운영
↓
tmux로 세션 유지
```

## Q&A: systemd는 무엇인가?

### 질문
`systemctl` 설명과 관련해서, `systemd`가 무엇인지 핵심만 보충해 달라. 리눅스의 네이티브 기능인가?

### 답변
`systemd`는 리눅스 시스템에서 **부팅 과정과 서비스 관리를 담당하는 시스템 매니저**다.

컴퓨터가 켜지면 리눅스 커널이 먼저 실행되고, 그 다음 사용자 공간의 첫 번째 주요 프로세스로 `systemd`가 실행된다. `systemd`는 이후 여러 서비스와 데몬을 시작하고 관리한다.

```text
전원 켜짐
-> 부트로더
-> 리눅스 커널
-> systemd
-> sshd, nginx, cron 같은 서비스 실행
```

`systemd`가 관리하는 대표적인 일은 다음과 같다.

```text
서비스 시작/중지/재시작
부팅 시 자동 실행 관리
서비스 의존성 관리
로그 시스템 연동
타이머 기반 작업 실행
마운트, 네트워크 등 시스템 자원 관리
```

`systemctl`은 이 `systemd`에게 명령을 내리는 CLI 도구다.

```bash
sudo systemctl start nginx
systemctl status ssh
sudo systemctl enable docker
```

중요한 점은 `systemd`가 **리눅스 커널 자체의 기능은 아니라는 것**이다. 커널 내부 기능이 아니라, 커널 위에서 실행되는 사용자 공간 프로그램이다.

다만 현대의 많은 리눅스 배포판, 예를 들어 Ubuntu, Debian, Fedora, CentOS 계열은 `systemd`를 기본 시스템 매니저로 사용한다. 그래서 실무에서는 리눅스의 표준적인 서비스 관리 방식처럼 자주 접하게 된다.

정리하면 다음과 같다.

```text
Linux kernel  운영체제의 핵심 커널
systemd       커널 위에서 실행되는 시스템/서비스 관리자
systemctl     systemd를 조작하는 명령어
```

따라서 `systemd`는 리눅스 커널의 네이티브 기능은 아니지만, 현대 리눅스 배포판에서 사실상 표준처럼 널리 쓰이는 기본 시스템 관리 도구라고 이해하면 된다.

## Q&A: systemd, systemctl, shell, kernel의 관계

### 질문
그러니까 `systemd`는 `systemctl`으로 조작되는 프로그램이고, `curl` 등을 명령어로 입력하면 shell이 그걸 커널로 전달하는 메커니즘과 비슷한 것인가?

### 답변
거의 맞지만, 조금만 정확히 다듬자.

`systemd`와 `systemctl` 관계는 이렇게 봐야 한다.

```text
systemd    실제로 시스템/서비스를 관리하는 데몬
systemctl  systemd에게 명령을 보내는 CLI 도구
```

예를 들어 다음 명령을 보자.

```bash
sudo systemctl restart nginx
```

이건 `systemctl`이라는 명령어가 실행되고, 그 명령어가 `systemd`에게 "nginx 서비스를 재시작해줘"라고 요청하는 구조다. 실제로 nginx 프로세스를 관리하고 상태를 추적하는 주체는 `systemd`다.

다만 `curl` 실행과는 조금 다르다.

```bash
curl https://example.com
```

이 경우 셸이 `curl`을 "커널로 전달한다"기보다는 다음 흐름에 가깝다.

```text
사용자 입력
-> shell이 curl 실행 파일을 찾음
-> shell이 커널에게 새 프로세스 실행을 요청
-> 커널이 curl 프로세스를 만듦
-> curl 프로세스가 실행됨
```

즉 셸은 커널에게 "이 프로그램 실행해줘"라고 시스템 콜을 통해 요청한다.

반면 다음 명령은 이렇게 볼 수 있다.

```bash
systemctl restart nginx
```

```text
사용자 입력
-> shell이 systemctl 실행
-> systemctl 프로세스가 systemd와 통신
-> systemd가 nginx 서비스 상태를 변경
-> 필요하면 커널에게 프로세스 생성/종료 등을 요청
```

그래서 비슷한 점은 있다.

```text
둘 다 사용자는 명령어를 입력한다.
둘 다 shell이 실행 파일을 찾아 프로세스로 실행한다.
둘 다 결국 커널의 도움을 받아 프로세스가 실행된다.
```

하지만 차이는 다음과 같다.

```text
curl      = curl 자신이 일을 하는 일반 명령어
systemctl = systemd에게 일을 시키는 조작 명령어
systemd   = 백그라운드에서 서비스들을 관리하는 관리자 프로세스
```

한 줄로 말하면 다음과 같다.

```text
systemctl은 systemd를 조작하는 리모컨이고, curl은 직접 일을 하는 도구다.
```

