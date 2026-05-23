# Web Server

## 목차
- [HTTP server](#http-server)
- [static file serving](#static-file-serving)
- [reverse proxy](#reverse-proxy)
- [TLS termination](#tls-termination)
- [worker model](#worker-model)
- [keep-alive](#keep-alive)

## HTTP server
HTTP server는 HTTP 요청을 받고 HTTP 응답을 반환하는 서버다. 브라우저, 모바일 앱, 다른 서버가 HTTP client가 될 수 있다.

```text
client
-> HTTP request
-> HTTP server
-> HTTP response
```

웹 서버와 애플리케이션 서버는 역할이 다를 수 있다. 웹 서버는 정적 파일 제공, 리버스 프록시, TLS 처리에 강하고, 애플리케이션 서버는 비즈니스 로직과 데이터 처리를 담당한다.

```text
web server          nginx, Apache
application server  Spring Boot, Express, Django
```

실제 구조는 다음처럼 나뉘는 경우가 많다.

```text
browser -> nginx -> Spring Boot app -> database
```

## static file serving
static file serving은 HTML, CSS, JavaScript, 이미지 같은 정적 파일을 그대로 제공하는 것이다.

```text
/index.html
/assets/app.js
/images/logo.png
```

정적 파일은 요청마다 비즈니스 로직을 실행할 필요가 없다. 그래서 nginx 같은 웹 서버가 빠르게 제공하기 좋다.

```nginx
location /static/ {
  root /var/www;
}
```

정적 파일 제공에서는 cache header도 중요하다.

```http
Cache-Control: public, max-age=31536000
```

파일 이름에 hash를 붙이면 오래 캐시해도 새 배포 시 파일명이 바뀌므로 안전하다.

```text
app.a8f31c.js
```

## reverse proxy
reverse proxy는 클라이언트 요청을 앞에서 먼저 받고 내부 애플리케이션 서버로 전달한다.

```text
browser -> reverse proxy -> app server
```

nginx 설정 예시는 다음과 같다.

```nginx
location / {
  proxy_pass http://127.0.0.1:8080;
}
```

reverse proxy를 앞단에 두는 이유는 여러 가지다.

```text
내부 app 포트 숨김
TLS 처리 집중
정적 파일 처리
로드 밸런싱
보안 헤더 적용
rate limit
```

사용자는 `https://example.com`으로 접속하지만, 실제 앱은 내부의 `127.0.0.1:8080`에서 실행될 수 있다.

## TLS termination
TLS termination은 HTTPS 연결의 TLS 처리를 앞단 서버에서 끝내는 구조다.

```text
browser --HTTPS--> nginx --HTTP--> app server
```

이 구조에서 nginx는 인증서와 private key를 가지고 TLS handshake를 처리한다. 내부 앱 서버는 암호화가 풀린 HTTP 요청을 받는다.

장점은 인증서 관리와 TLS 설정을 한 곳에서 집중할 수 있다는 점이다.

```text
nginx
-> certificate 관리
-> HTTPS 처리
-> 내부 app으로 전달
```

내부 통신도 민감하거나 네트워크 경계가 넓으면 nginx와 app 사이도 HTTPS 또는 mTLS로 보호할 수 있다.

## worker model
웹 서버는 많은 연결을 동시에 처리해야 한다. worker model은 요청 처리를 여러 worker process나 thread로 나누는 구조다.

nginx는 보통 master process와 worker process 구조를 가진다.

```text
nginx master
├── worker
├── worker
└── worker
```

master는 설정 로드와 worker 관리를 담당하고, worker가 실제 요청을 처리한다.

```text
master  설정, worker 관리
worker  client connection 처리
```

worker는 이벤트 기반 I/O를 사용해 많은 연결을 효율적으로 처리할 수 있다. 연결 하나마다 무조건 스레드 하나를 만드는 방식과 다르다.

## keep-alive
keep-alive는 하나의 TCP 연결을 여러 HTTP 요청에 재사용하는 방식이다.

```text
TCP 연결 생성
-> HTTP 요청 1
-> HTTP 응답 1
-> HTTP 요청 2
-> HTTP 응답 2
-> 연결 종료
```

keep-alive가 없으면 요청마다 TCP handshake와 TLS handshake 비용이 반복될 수 있다.

```text
keep-alive 없음  요청마다 연결 생성 비용
keep-alive 있음  연결 재사용으로 latency 감소
```

다만 연결을 오래 유지하면 서버의 connection 자원을 계속 사용한다. 그래서 timeout과 최대 요청 수를 적절히 설정해야 한다.

```text
장점  연결 재사용, 성능 개선
비용  연결 자원 점유
```
