# Nginx

## 목차
- [Nginx의 역할](#nginx의-역할)
- [웹 서버와 리버스 프록시](#웹-서버와-리버스-프록시)
- [Server Block](#server-block)
- [Location Matching](#location-matching)
- [Proxy_pass](#proxy_pass)
- [TLS Termination](#tls-termination)
- [Load Balancing](#load-balancing)
- [로그 분석](#로그-분석)
- [Reload와 Restart](#reload와-restart)

---

## Nginx의 역할

Nginx는 **높은 동시성을 처리하는 웹 서버이면서 동시에 강력한 리버스 프록시, 로드 밸런서, SSL/TLS 처리기**다. Apache와 달리 비동기 이벤트 기반 아키텍처로 메모리 효율적이다.

```text
브라우저
    ↓ (HTTP/HTTPS 요청)
Nginx (웹 서버 + 리버스 프록시 + 로드 밸런서)
    ├─ 정적 파일 제공 (이미지, CSS, JS)
    ├─ 캐싱
    ├─ 압축 (gzip)
    ├─ SSL/TLS 종료
    ├─ 요청 라우팅
    └─ 여러 백엔드 서버로 분산
    ↓
백엔드 애플리케이션 서버 (Node.js, Python, Java)
```

---

## 웹 서버와 리버스 프록시

### 웹 서버 역할

Nginx는 **정적 파일을 직접 제공**할 수 있다.

```nginx
server {
  listen 80;
  root /var/www/html;
  
  location / {
    try_files $uri $uri/ =404;
  }
}
```

```text
브라우저 요청: /index.html
    ↓
Nginx (디스크에서 /var/www/html/index.html 찾음)
    ↓
직접 응답 (빠름, 효율적)
```

### 리버스 프록시 역할

Nginx는 **클라이언트 요청을 백엔드 서버로 전달**하고, 응답을 돌려받아 클라이언트에게 전달한다.

```nginx
server {
  listen 80;
  
  location / {
    proxy_pass http://127.0.0.1:8080;
  }
}
```

```text
브라우저 요청
    ↓
Nginx (127.0.0.1:8080로 요청 전달)
    ↓
백엔드 서버 (응답 생성)
    ↓
Nginx (응답을 브라우저에게 전달)
```

### 클라이언트는 Nginx만 안다

```text
클라이언트 입장:
- 요청: 203.0.113.10:80
- 응답: 203.0.113.10:80
- 백엔드 서버의 존재를 모름

실제:
- Nginx (203.0.113.10:80)
- 백엔드 서버 (내부 IP, 클라이언트에게 숨겨짐)
```

---

## Server Block

### Server Block이란?

**"server`와 `location`은 각각 무엇을 결정하는가?"**

`server` block은 **어떤 호스트명, 포트로 들어온 요청을 처리할지 결정**한다.

```nginx
server {
  listen 80;
  server_name example.com www.example.com;
  
  # 이 server block은 example.com과 www.example.com로 들어온 요청을 처리
}

server {
  listen 443 ssl;
  server_name api.example.com;
  
  # 이 server block은 api.example.com의 HTTPS 요청을 처리
}

server {
  listen 8080;
  server_name internal.example.com;
  
  # 이 server block은 8080 포트로 들어온 요청을 처리
}
```

### 여러 domain 분리

```nginx
# example.com 웹사이트
server {
  listen 80;
  server_name example.com www.example.com;
  root /var/www/example;
}

# api.example.com API 서버
server {
  listen 80;
  server_name api.example.com;
  
  location / {
    proxy_pass http://127.0.0.1:3000;
  }
}

# admin.example.com 관리자 페이지
server {
  listen 80;
  server_name admin.example.com;
  
  location / {
    proxy_pass http://127.0.0.1:9000;
  }
}
```

### Server Block 매칭 순서

```text
요청: GET http://example.com/api/users

1. listen 포트 확인 (80)
2. server_name으로 서버 선택
   - example.com ← 매칭됨
   - api.example.com (매칭 안 됨)
   - admin.example.com (매칭 안 됨)
3. example.com server block의 location 처리 계속
```

---

## Location Matching

### Location Block이란?

**Location block은 server 내에서 경로에 따라 어떻게 처리할지 결정**한다.

```nginx
server {
  server_name example.com;
  root /var/www/html;
  
  # 정확한 경로
  location = /favicon.ico {
    access_log off;
  }
  
  # 정규표현식
  location ~ \.php$ {
    # PHP 파일 처리
  }
  
  # 경로 프리픽스
  location /api/ {
    proxy_pass http://127.0.0.1:3000;
  }
  
  # 기본 (가장 나중에 매칭)
  location / {
    try_files $uri $uri/ =404;
  }
}
```

### Location 매칭 타입

```text
= /path              정확히 일치
^~ /path             프리픽스 (정규식 무시)
~ /regex             정규식 (대소문자 구분)
~* /regex            정규식 (대소문자 무시)
/path                프리픽스 매칭
```

### 매칭 우선순위

```text
1. 정확한 매칭 (=) → 바로 사용
2. 프리픽스 매칭 (^~) → 정규식 무시하고 사용
3. 정규식 매칭 (~, ~*)  → 순서대로 평가
4. 프리픽스 매칭 (일반) → 가장 긴 매칭 사용

예시:
요청: /api/users/123

location = /api/users/123 { } → 이것은 정확히 일치하면 매칭
location ^~ /api/ { }  → 프리픽스 매칭, 우선도 높음
location ~ ^/api/ { }  → 정규식
location /api { }      → 일반 프리픽스
```

### 실제 예시

```nginx
server {
  server_name example.com;
  root /var/www/html;
  
  # API 요청 → 백엔드 로드밸런서로
  location /api/ {
    proxy_pass http://api-backend;
  }
  
  # 정적 파일 → 캐싱 헤더
  location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2)$ {
    expires 365d;
    add_header Cache-Control "public, immutable";
  }
  
  # PHP 파일 → PHP-FPM으로
  location ~ \.php$ {
    fastcgi_pass 127.0.0.1:9000;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
  }
  
  # 그 외 → 기본 처리
  location / {
    try_files $uri $uri/ /index.html =404;
  }
}
```

---

## Proxy_pass

### Proxy_pass란?

**"`proxy_pass`는 요청을 어떻게 전달하는가?"**

`proxy_pass`는 **들어온 요청을 백엔드 서버로 전달**하는 지시어다.

```nginx
location /api/ {
  proxy_pass http://127.0.0.1:8080;
}
```

```text
1. 클라이언트 요청: GET /api/users
2. Nginx가 요청을 http://127.0.0.1:8080로 전달
3. 백엔드에서 받는 요청: GET /api/users (경로 그대로)
4. 백엔드 응답
5. Nginx가 클라이언트에게 응답 전달
```

### Trailing Slash의 차이

```nginx
# Case 1: trailing slash 없음
location /api {
  proxy_pass http://127.0.0.1:8080;
}
# 요청: /api/users → 백엔드: /api/users

# Case 2: trailing slash 있음
location /api {
  proxy_pass http://127.0.0.1:8080/;
}
# 요청: /api/users → 백엔드: /users (앞의 /api가 제거됨)

# Case 3: 경로 지정
location /api {
  proxy_pass http://127.0.0.1:8080/v1/;
}
# 요청: /api/users → 백엔드: /v1/users
```

### 중요한 proxy headers

```nginx
location /api/ {
  proxy_pass http://127.0.0.1:8080;
  
  # 원본 클라이언트 IP 전달
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  
  # 원본 호스트명 전달
  proxy_set_header Host $host;
  
  # 원본 프로토콜 전달 (HTTPS → HTTP 변환 시)
  proxy_set_header X-Forwarded-Proto $scheme;
  
  # 웹소켓 지원
  proxy_http_version 1.1;
  proxy_set_header Upgrade $http_upgrade;
  proxy_set_header Connection "upgrade";
}
```

### 타임아웃 설정

```nginx
location /api/ {
  proxy_pass http://127.0.0.1:8080;
  
  # 연결 타임아웃 (기본 60s)
  proxy_connect_timeout 30s;
  
  # 응답 대기 타임아웃 (기본 60s)
  proxy_send_timeout 30s;
  
  # 읽기 타임아웃 (기본 60s)
  proxy_read_timeout 30s;
}
```

---

## TLS Termination

### TLS Termination이란?

**"TLS termination"은 Nginx에서 HTTPS를 복호화하고, 백엔드와는 HTTP로 통신**한다는 뜻이다.

```text
클라이언트 (HTTPS)
    ↓
Nginx (HTTPS 복호화)
    ↓
백엔드 서버 (HTTP - 내부 네트워크이므로 안전)
```

### 설정 예시

```nginx
server {
  listen 443 ssl;
  server_name example.com;
  
  # SSL 인증서 설정
  ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
  
  # SSL 보안 설정
  ssl_protocols TLSv1.2 TLSv1.3;
  ssl_ciphers HIGH:!aNULL:!MD5;
  ssl_session_cache shared:SSL:10m;
  ssl_session_timeout 10m;
  
  location / {
    proxy_pass http://127.0.0.1:8080;
    proxy_set_header X-Forwarded-Proto https;
  }
}

# HTTP를 HTTPS로 리다이렉트
server {
  listen 80;
  server_name example.com;
  return 301 https://$server_name$request_uri;
}
```

### Let's Encrypt 자동 갱신

```bash
# Certbot으로 자동 갱신 설정
certbot renew --quiet --agree-tos

# crontab에 매일 2회 실행
0 0,12 * * * /usr/bin/certbot renew --quiet && /bin/systemctl reload nginx
```

---

## Load Balancing

### Upstream 정의

```nginx
# 백엔드 서버 그룹 정의
upstream app-backend {
  server 10.0.1.10:8080;
  server 10.0.1.11:8080;
  server 10.0.1.12:8080;
}

server {
  listen 80;
  server_name example.com;
  
  location /api {
    proxy_pass http://app-backend;
  }
}
```

```text
요청이 들어올 때마다 라운드로빈으로 분산:
요청 1 → 10.0.1.10
요청 2 → 10.0.1.11
요청 3 → 10.0.1.12
요청 4 → 10.0.1.10 (다시 반복)
```

### 다양한 로드 밸런싱 알고리즘

```nginx
# 라운드로빈 (기본)
upstream app-backend {
  server 10.0.1.10:8080;
  server 10.0.1.11:8080;
}

# Least Connections (활성 연결이 적은 서버 우선)
upstream app-backend {
  least_conn;
  server 10.0.1.10:8080;
  server 10.0.1.11:8080;
}

# IP Hash (클라이언트 IP 기반, 같은 서버로 고정)
upstream app-backend {
  ip_hash;
  server 10.0.1.10:8080;
  server 10.0.1.11:8080;
}

# 가중치 (성능이 좋은 서버에 더 많은 요청)
upstream app-backend {
  server 10.0.1.10:8080 weight=3;
  server 10.0.1.11:8080 weight=1;
}
```

### Health Check (Nginx Plus 기능)

```nginx
upstream app-backend {
  server 10.0.1.10:8080;
  server 10.0.1.11:8080;
  
  zone backend 64k;
}

# 주기적으로 health check
upstream app-backend {
  server 10.0.1.10:8080;
  server 10.0.1.11:8080 max_fails=2 fail_timeout=30s;
  
  # max_fails: 실패 횟수, fail_timeout: 실패 후 대기 시간
}
```

---

## 로그 분석

### Access Log와 Error Log

**"access log와 error log는 어떻게 읽는가?"**

```nginx
# access log 설정 (기본 위치: /var/log/nginx/access.log)
access_log /var/log/nginx/access.log main;

# log format 정의
log_format main '$remote_addr - $remote_user [$time_local] '
                 '"$request" $status $body_bytes_sent '
                 '"$http_referer" "$http_user_agent"';

# error log 설정 (기본 위치: /var/log/nginx/error.log)
error_log /var/log/nginx/error.log warn;
```

### Access Log 분석

```
203.0.113.45 - - [23/May/2026:10:15:30 +0900] "GET /api/users HTTP/1.1" 200 1234 "-" "Mozilla/5.0"
```

```text
203.0.113.45          클라이언트 IP
-                     클라이언트 사용자명 (없으면 -)
[23/May/2026:...]     요청 시간
"GET /api/users ..." 요청 라인
200                   HTTP 상태 코드 (성공)
1234                  응답 크기 (바이트)
"-"                   Referer (없으면 -)
"Mozilla/5.0"         User-Agent
```

### 실시간 로그 모니터링

```bash
# 실시간 access log 보기
tail -f /var/log/nginx/access.log

# 특정 IP의 요청만 보기
tail -f /var/log/nginx/access.log | grep 203.0.113.45

# HTTP 상태별로 집계
awk '{print $9}' /var/log/nginx/access.log | sort | uniq -c

# 가장 많은 요청을 한 IP 찾기
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -10

# 느린 요청 찾기 (응답 시간이 1초 이상)
tail -f /var/log/nginx/access.log | awk '$NF > 1 {print}'
```

### Error Log 분석

```
2026/05/23 10:15:30 [error] 1234#0: *567 upstream timed out (110: Connection timed out)
```

```text
2026/05/23 10:15:30    로그 시간
[error]                로그 레벨 (debug, info, notice, warn, error, crit, alert, emerg)
1234#0                 PID#TID
*567                   Connection ID
upstream timed out     에러 메시지
```

---

## Reload와 Restart

### Reload와 Restart의 차이

**"reload와 restart는 무엇이 다른가?"**

| | Reload | Restart |
|---|---|---|
| 명령 | `nginx -s reload` | `systemctl restart nginx` |
| 기존 연결 | 유지됨 | 종료됨 |
| 가동 시간 | 0초 (무중단) | 초 단위 (약간의 다운타임) |
| 설정 적용 | 새로운 연결부터 적용 | 즉시 적용 |
| 사용처 | 일반적인 설정 변경 | 프로토콜 변경 등 드문 경우 |

### Reload 방식 (권장)

```bash
# Nginx 설정 파일 수정
vi /etc/nginx/nginx.conf

# 설정 문법 확인
nginx -t
# nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
# nginx: configuration file /etc/nginx/nginx.conf test is successful

# 무중단으로 적용
nginx -s reload
# 또는
systemctl reload nginx
```

```text
기존 Worker Process (old)
    ↓ (SIGHUP 신호)
graceful shutdown
    ↓ (기존 연결 처리 완료)
종료

새 Worker Process (new)
    ↓
새 설정으로 시작
    ↓
새 요청부터 처리
```

### Restart 방식

```bash
# 설정 파일 수정
vi /etc/nginx/nginx.conf

# 설정 문법 확인
nginx -t

# Nginx 재시작 (모든 연결 종료)
systemctl restart nginx
```

```text
기존 Worker Process
    ↓
강제 종료 (기존 연결 끊김)
    ↓
새 Master Process 시작
    ↓
새 Worker Process 시작
```

### 실제 운영 예시

```bash
# 1. 설정 변경
vi /etc/nginx/nginx.conf
# upstream이나 location 규칙 변경

# 2. 설정 확인
nginx -t

# 3. 무중단 적용
sudo systemctl reload nginx

# 4. 로그 확인
tail -f /var/log/nginx/error.log

# 5. 상태 확인
systemctl status nginx
```

---

## 실전 예시

```nginx
# /etc/nginx/nginx.conf

upstream api-backend {
  server 10.0.1.10:3000;
  server 10.0.1.11:3000;
  server 10.0.1.12:3000;
}

upstream web-backend {
  server 10.0.1.20:8080;
  server 10.0.1.21:8080;
}

# HTTP를 HTTPS로 리다이렉트
server {
  listen 80;
  server_name example.com *.example.com;
  return 301 https://$server_name$request_uri;
}

# HTTPS 메인 서버
server {
  listen 443 ssl http2;
  server_name example.com;
  root /var/www/html;
  
  # SSL 설정
  ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
  ssl_protocols TLSv1.2 TLSv1.3;
  ssl_ciphers HIGH:!aNULL:!MD5;
  
  # 로깅
  access_log /var/log/nginx/example.access.log main;
  error_log /var/log/nginx/example.error.log warn;
  
  # API 라우팅
  location /api/ {
    proxy_pass http://api-backend;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $host;
    proxy_connect_timeout 30s;
    proxy_read_timeout 30s;
  }
  
  # 정적 파일 캐싱
  location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf)$ {
    expires 365d;
    add_header Cache-Control "public, immutable";
  }
  
  # 기타 → 웹 애플리케이션
  location / {
    try_files $uri $uri/ @fallback;
  }
  
  location @fallback {
    proxy_pass http://web-backend;
    proxy_set_header Host $host;
  }
}

# API 서브도메인
server {
  listen 443 ssl http2;
  server_name api.example.com;
  
  ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
  
  access_log /var/log/nginx/api.access.log main;
  error_log /var/log/nginx/api.error.log warn;
  
  location / {
    proxy_pass http://api-backend;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $host;
  }
}
```

---

## 정리

| 개념 | 설명 |
|------|------|
| Server Block | 호스트명과 포트로 요청을 받을 방식 결정 |
| Location | 경로에 따라 어떻게 처리할지 결정 |
| Proxy_pass | 요청을 백엔드로 전달 |
| TLS Termination | HTTPS 처리를 Nginx에서 수행 |
| Load Balancing | 여러 백엔드에 요청 분산 |
| Reload | 무중단 설정 적용 |
| Restart | 설정 및 연결 모두 재시작 |

---

## Related Notes

- [L4 vs L7 로드밸런싱](../load-balancing/overview.md)
- [Kubernetes](../kubernetes/overview.md)
