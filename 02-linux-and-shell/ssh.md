# SSH

## 목차
- SSH의 역할
- client와 server
- 사용자 인증
- 공개키 인증
- 포트와 방화벽

## 기초 개념
SSH는 원격 컴퓨터에 안전하게 CLI 세션을 열기 위한 프로토콜이자 도구다. 사용자는 `ssh` 클라이언트로 접속하고, 원격 서버는 `sshd`가 연결을 받는다.

```text
내 PC ssh client -> network -> remote sshd -> shell
```

## 간단한 예시
```bash
ssh ubuntu@server.example.com
ssh -i key.pem ec2-user@1.2.3.4
scp app.log ubuntu@server:/tmp/
```

## 반드시 알아야 할 질문
- SSH는 원격 데스크탑과 무엇이 다른가?
- 비밀번호 인증과 공개키 인증은 어떻게 다른가?
- `known_hosts`는 왜 필요한가?
- SSH 터널링은 어떤 문제를 해결하는가?
