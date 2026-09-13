# SSH

## 목차
- [SSH의 역할](#ssh의-역할)
- [client와 server](#client와-server)
- [사용자 인증](#사용자-인증)
- [공개키 인증](#공개키-인증)
- [포트와 방화벽](#포트와-방화벽)

## SSH의 역할
SSH(Secure Shell)는 원격 컴퓨터에 안전하게 접속해 CLI 세션을 열기 위한 프로토콜이자 도구다. 서버 관리, 배포, 로그 확인, 파일 전송에 널리 쓰인다.

```text
내 컴퓨터
-> SSH 연결
-> 원격 서버
-> 원격 shell
```

SSH는 원격 데스크탑과 다르다. 원격 데스크탑은 GUI 화면을 조작하고, SSH는 보통 원격 터미널을 조작한다.

```text
원격 데스크탑  화면, 마우스, GUI 앱 조작
SSH           터미널, 명령어, CLI 환경 조작
```

예를 들어 EC2 Linux 서버에 접속해 패키지를 설치하거나 서비스를 재시작하는 작업은 SSH로 하는 경우가 많다.

```bash
ssh ubuntu@server.example.com
sudo systemctl restart nginx
```

## client와 server
SSH 연결에는 client와 server가 있다. 사용자가 실행하는 `ssh` 명령은 client이고, 원격 컴퓨터에서 접속을 받아주는 프로세스는 `sshd` server다.

```text
ssh client
-> network
-> sshd server
-> login shell
```

다음 명령은 `server.example.com`에 `ubuntu` 사용자로 접속한다.

```bash
ssh ubuntu@server.example.com
```

흐름은 대략 다음과 같다.

```text
1. 도메인을 IP로 해석
2. 서버의 SSH 포트로 TCP 연결
3. 서버의 sshd가 연결 수락
4. 사용자 인증
5. 원격 shell 시작
```

접속 후 입력하는 명령은 내 컴퓨터가 아니라 원격 서버에서 실행된다.

## 사용자 인증
SSH는 원격 서버의 OS 사용자 계정으로 접속한다.

```bash
ssh shin@192.168.0.10
```

여기서 `shin`은 SSH 전용 이름이라기보다 원격 서버에 존재하는 사용자 계정이다. 인증 방식은 보통 비밀번호 또는 공개키 기반이다.

```text
비밀번호 인증  사용자가 비밀번호 입력
공개키 인증    개인키로 서명하고 서버가 공개키로 검증
```

인증이 성공하면 `sshd`는 해당 사용자 권한의 shell을 열어준다.

```text
sshd
-> 인증 성공
-> /bin/bash 또는 사용자 기본 shell 실행
```

root로 직접 접속하는 것은 보안상 막아두는 경우가 많고, 일반 사용자로 접속한 뒤 필요한 명령만 `sudo`로 실행하는 방식이 흔하다.

## 공개키 인증
공개키 인증은 비밀번호 대신 key pair를 사용하는 방식이다.

```text
private key  내 컴퓨터에 안전하게 보관
public key   서버의 authorized_keys에 등록
```

접속 예시는 다음과 같다.

```bash
ssh -i key.pem ubuntu@1.2.3.4
```

서버에는 사용자의 public key가 등록되어 있어야 한다.

```text
~/.ssh/authorized_keys
```

공개키 인증 흐름은 대략 다음과 같다.

```text
1. client가 private key를 사용해 인증 응답 생성
2. server가 등록된 public key로 검증
3. 검증 성공 시 로그인 허용
```

`known_hosts`는 클라이언트가 이전에 접속한 서버의 host key를 저장하는 파일이다. 이것은 "내가 접속하는 서버가 예전에 접속했던 그 서버가 맞는가"를 확인하는 데 필요하다.

```text
~/.ssh/known_hosts
```

서버 host key가 갑자기 바뀌면 중간자 공격 가능성을 의심해야 한다. 물론 서버를 재설치해 정상적으로 바뀐 경우도 있다.

## 포트와 방화벽
SSH 기본 포트는 22번이다.

```bash
ssh ubuntu@server.example.com
```

다른 포트를 쓰면 `-p` 옵션을 사용한다.

```bash
ssh -p 2222 ubuntu@server.example.com
```

SSH 접속이 되려면 여러 조건이 맞아야 한다.

```text
서버가 켜져 있음
sshd가 실행 중
서버가 해당 포트에서 listen 중
방화벽이 포트를 허용
클라우드 보안 그룹이 포트를 허용
사용자 인증 성공
```

SSH 터널링은 SSH 연결을 통해 다른 네트워크 연결을 안전하게 전달하는 기능이다. 외부에서 직접 접근할 수 없는 내부 서비스에 임시로 접근하거나, 로컬 포트를 원격 서비스로 연결할 때 사용한다.

```bash
ssh -L 15432:localhost:5432 ubuntu@server.example.com
```

이 예시는 내 컴퓨터의 `15432` 포트를 원격 서버 입장에서의 `localhost:5432`로 전달한다. 원격 DB를 로컬에서 접속하는 것처럼 다룰 때 사용할 수 있다.

```text
local app
-> localhost:15432
-> SSH tunnel
-> remote localhost:5432
```
