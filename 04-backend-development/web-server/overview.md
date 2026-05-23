# Web Server

## 목차
- HTTP server
- static file serving
- reverse proxy
- TLS termination
- worker model
- keep-alive

## 기초 개념
웹 서버는 HTTP 요청을 받고 응답을 반환하는 서버다. 정적 파일을 직접 제공할 수도 있고, 리버스 프록시로 내부 애플리케이션 서버에 요청을 넘길 수도 있다.

```text
browser -> web server -> app server
```

## 간단한 예시
```nginx
location / {
  proxy_pass http://127.0.0.1:8080;
}
```

## 반드시 알아야 할 질문
- 웹 서버와 애플리케이션 서버는 무엇이 다른가?
- reverse proxy는 왜 앞단에 두는가?
- TLS termination은 무엇인가?
- keep-alive는 성능에 어떤 영향을 주는가?
