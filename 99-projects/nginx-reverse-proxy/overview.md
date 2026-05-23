# Project: Nginx Reverse Proxy

## 목차
- 목표
- app server
- nginx proxy
- port mapping
- request flow

## 기초 개념
이 프로젝트는 nginx가 외부 요청을 받고 내부 애플리케이션 서버로 전달하는 리버스 프록시 구조를 실습한다.

```text
browser -> nginx:80 -> app:8080
```

## 간단한 예시
```nginx
location / {
  proxy_pass http://app:8080;
}
```

## 반드시 알아야 할 질문
- 사용자는 왜 app 서버 포트에 직접 접속하지 않는가?
- nginx와 app은 어떤 네트워크로 연결되는가?
- access log로 요청 흐름을 어떻게 확인하는가?
