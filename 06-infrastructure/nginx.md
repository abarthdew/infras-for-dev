# Nginx

## 목차
- 웹 서버와 리버스 프록시
- server block
- location matching
- proxy_pass
- TLS termination
- load balancing

## 기초 개념
Nginx는 정적 파일 제공, 리버스 프록시, 로드 밸런싱, TLS 종료에 자주 쓰이는 웹 서버다.

```text
browser -> nginx -> app server
```

## 간단한 예시
```nginx
server {
  listen 80;
  location / {
    proxy_pass http://127.0.0.1:8080;
  }
}
```

## 반드시 알아야 할 질문
- `server`와 `location`은 각각 무엇을 결정하는가?
- `proxy_pass`는 요청을 어떻게 전달하는가?
- access log와 error log는 어떻게 읽는가?
- reload와 restart는 무엇이 다른가?
