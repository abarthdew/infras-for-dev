# Authentication

## 목차
- 인증과 인가
- session
- cookie
- JWT
- OAuth 2.0
- password hashing

## 기초 개념
인증(authentication)은 사용자가 누구인지 확인하는 것이고, 인가(authorization)는 그 사용자가 어떤 행동을 할 수 있는지 판단하는 것이다.

```text
login -> identity 확인 -> 권한 검사 -> resource 접근
```

## 간단한 예시
```text
브라우저 로그인
-> 서버가 session 생성
-> Set-Cookie 응답
-> 이후 요청에 Cookie 포함
```

## 반드시 알아야 할 질문
- 인증과 인가는 왜 구분해야 하는가?
- session과 JWT는 무엇이 다른가?
- password는 왜 평문 저장하면 안 되는가?
- OAuth는 로그인 기술인가 권한 위임 기술인가?
