# Project: Nginx Reverse Proxy

## 목차
- [프로젝트 목표](#프로젝트-목표)
- [App Server](#app-server)
- [Nginx Proxy](#nginx-proxy)
- [Port Mapping](#port-mapping)
- [Request Flow](#request-flow)
- [실습 과정](#실습-과정)

---

## 프로젝트 목표

**Nginx가 외부 요청을 받아 내부 애플리케이션 서버로 전달하는 리버스 프록시 구조를 실습**하고, 다중 계층 웹 애플리케이션의 기본 아키텍처를 이해한다.

```text
아키텍처:
사용자 (Internet)
    ↓ (HTTP:80)
Nginx (Reverse Proxy)
    ↓ (Internal Network)
App Server (localhost:8080)
```

### 왜 리버스 프록시가 필요한가?

```text
직접 접속 (위험):
- 사용자가 app 서버의 실제 주소:포트를 알게 됨
- 보안 취약: 서버 정보 노출
- 유지보수 어려움: 서버 변경 시 모든 클라이언트 영향

리버스 프록시 (안전):
- 사용자는 nginx만 봄
- 앱 서버는 숨겨짐
- 로드 밸런싱, TLS, 압축 등 추가 기능 가능
- 서버 변경 시 nginx 설정만 수정
```

---

## App Server

### 애플리케이션 서버 구성

```bash
# 간단한 Python Flask 앱 (app.py)
from flask import Flask

app = Flask(__name__)

@app.route('/')
def index():
    return 'Hello from App Server!'

@app.route('/api/status')
def status():
    return {'status': 'ok', 'port': 8080}

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080)
```

### Docker로 실행

```dockerfile
# Dockerfile
FROM python:3.9-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install flask

COPY app.py .

EXPOSE 8080
CMD ["python", "app.py"]
```

### 컨테이너 실행

```bash
# 빌드
docker build -t app-server .

# 실행 (localhost:8080에서만 접근 가능)
docker run --name app-server -p 127.0.0.1:8080:8080 app-server
```

---

## Nginx Proxy

### Nginx 설정

```nginx
# nginx.conf
upstream app_backend {
    server app:8080;  # Docker network 내 hostname
}

server {
    listen 80;
    server_name localhost;

    client_max_body_size 10M;

    # 정적 파일 (있다면)
    location /static/ {
        alias /var/www/static/;
        expires 30d;
    }

    # 모든 요청을 app 서버로 전달
    location / {
        proxy_pass http://app_backend;
        
        # 클라이언트 정보 전달
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # 타임아웃 설정
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }

    # 헬스 체크 엔드포인트
    location /health {
        access_log off;
        return 200 'ok';
        add_header Content-Type text/plain;
    }
}
```

### Dockerfile (Nginx)

```dockerfile
FROM nginx:alpine

COPY nginx.conf /etc/nginx/nginx.conf
COPY default.conf /etc/nginx/conf.d/default.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

---

## Port Mapping

### "사용자는 왜 app 서버 포트에 직접 접속하지 않는가?"

**직접 접속은 서버 정보 노출, 보안 취약, 확장성 제한 등의 문제를 야기한다. 리버스 프록시를 통해 추상화**한다.

```text
직접 접속 (❌ 위험):
브라우저 → app.example.com:8080
- 포트 8080 노출 → 서버 기술 스택 드러남
- 직접 포트 변경 불가능
- 다중 앱 서버 로드 밸런싱 불가능

리버스 프록시 (✓ 안전):
브라우저 → example.com:80 → 내부 app:8080
- 외부: 포트 80만 노출 (표준)
- 내부: 앱 서버 주소/포트 숨김
- 여러 앱 서버 관리 가능
```

### Docker Compose로 네트워크 구성

```yaml
version: '3.8'

services:
  nginx:
    build: ./nginx
    ports:
      - "80:80"      # 외부에 80 포트 노출
    depends_on:
      - app
    networks:
      - web-net

  app:
    build: ./app
    # ports: 주석 처리 (외부 노출 안 함)
    networks:
      - web-net
    environment:
      - FLASK_ENV=production

networks:
  web-net:
    driver: bridge
```

### 포트 매핑 설명

```text
호스트 포트: 실제 시스템 (또는 Docker 호스트)의 포트
컨테이너 포트: 컨테이너 내부 포트

-p 80:80
├─ 호스트 포트: 80
└─ 컨테이너 포트: 80

-p 127.0.0.1:8080:8080
├─ 127.0.0.1: 루프백 (localhost만 접근)
├─ 호스트 포트: 8080
└─ 컨테이너 포트: 8080
```

---

## Request Flow

### 요청이 흐르는 과정

```text
1. 브라우저 요청 (사용자)
   GET http://example.com/ (포트 80)
   
2. Nginx 수신
   포트 80 → listen 80
   → location / 매칭
   
3. Upstream 결정
   upstream app_backend: app:8080
   
4. App 서버로 전달
   proxy_pass http://app:8080/
   + 헤더: X-Real-IP, X-Forwarded-For
   
5. App 서버 처리
   Flask: request.remote_addr
   (X-Real-IP 헤더로부터 클라이언트 IP 가져옴)
   
6. 응답 반환
   App → Nginx → 브라우저
   
7. 로그 기록
   Nginx access log: 요청/응답 기록
```

### 헤더의 중요성

```text
클라이언트: 1.2.3.4

Proxy_pass 없이:
app.remote_addr = "127.0.0.1"
(localhost에서 온 것처럼 보임)

X-Real-IP 헤더 포함:
X-Real-IP: 1.2.3.4
app.request.headers.get('X-Real-IP') = "1.2.3.4"
(실제 클라이언트 IP 알 수 있음)

X-Forwarded-For:
X-Forwarded-For: 1.2.3.4, 127.0.0.1
(프록시 체인 기록)
```

### "nginx와 app은 어떤 네트워크로 연결되는가?"

**Docker Compose로 실행하면 같은 custom bridge network (web-net)로 자동 연결되어, hostname으로 통신**한다.

```text
Docker Network 종류:

1. Bridge Network (기본)
   - 각 컨테이너에 IP 할당
   - 컨테이너끼리 hostname으로 통신
   - 예: app:8080 (DNS 자동 해석)

2. Host Network
   - 호스트 네트워크 공유
   - 성능 좋음, 포트 격리 없음

3. None Network
   - 네트워크 없음 (고립)
```

---

## Request Flow 검증

### "access log로 요청 흐름을 어떻게 확인하는가?"

**Nginx access log를 통해 누가, 언제, 무엇을 요청했는지 추적할 수 있다.**

```nginx
# access_log 설정 (nginx.conf)
log_format main '$remote_addr - $remote_user [$time_local] '
                '"$request" $status $body_bytes_sent '
                '"$http_referer" "$http_user_agent" '
                '"$http_x_forwarded_for"';

access_log /var/log/nginx/access.log main;
```

### 로그 예시

```
192.168.1.100 - - [23/May/2026 10:15:30 +0000] "GET / HTTP/1.1" 200 25 "-" "Mozilla/5.0" "1.2.3.4"

해석:
192.168.1.100: Nginx를 접속한 주소 (호스트 또는 프록시)
[23/May/2026 10:15:30]: 접속 시각
GET / HTTP/1.1: 요청 메서드, 경로, 프로토콜
200: 응답 상태 코드 (성공)
25: 응답 바이트 크기
Mozilla/5.0: User-Agent
1.2.3.4: X-Forwarded-For (실제 클라이언트 IP)
```

### 로그 보기

```bash
# 실시간 로그 보기
docker logs -f nginx_container

# 액세스 로그 모니터링
docker exec nginx_container tail -f /var/log/nginx/access.log

# 에러 로그
docker exec nginx_container tail -f /var/log/nginx/error.log

# 특정 IP 요청만 보기
docker exec nginx_container grep "1.2.3.4" /var/log/nginx/access.log

# 상태 코드 통계
docker exec nginx_container awk '{print $9}' /var/log/nginx/access.log | sort | uniq -c
```

---

## 실습 과정

### 1단계: 파일 준비

```bash
mkdir -p nginx-reverse-proxy
cd nginx-reverse-proxy

# 디렉토리 구조
.
├── app/
│   ├── Dockerfile
│   ├── app.py
│   └── requirements.txt
├── nginx/
│   ├── Dockerfile
│   └── default.conf
└── docker-compose.yml
```

### 2단계: 빌드 및 실행

```bash
docker-compose up -d

# 확인
docker ps
# 두 컨테이너 실행: nginx, app-server

# 상태 확인
curl http://localhost/
# 응답: Hello from App Server!

curl http://localhost/api/status
# 응답: {"status": "ok", "port": 8080}
```

### 3단계: 로그 확인

```bash
# Nginx 액세스 로그
docker exec nginx tail -f /var/log/nginx/access.log
```

### 4단계: 문제 해결

```bash
# 컨테이너 간 통신 확인
docker exec nginx ping app
# app:8080 응답 확인

# Nginx 설정 검증
docker exec nginx nginx -t
# nginx: configuration file /etc/nginx/nginx.conf test is successful

# 포트 확인
docker ps -a
netstat -tulpn | grep 80
```

---

## 확장 사항

### 로드 밸런싱 (다중 app 서버)

```nginx
upstream app_backend {
    server app1:8080;
    server app2:8080;
    server app3:8080;
    
    # 로드 밸런싱 방식
    # 라운드 로빈 (기본)
    # least_conn: 연결 수 최소
    # ip_hash: 클라이언트 IP 기반
}
```

### TLS/HTTPS

```nginx
server {
    listen 443 ssl;
    
    ssl_certificate /etc/nginx/cert.pem;
    ssl_certificate_key /etc/nginx/key.pem;
    
    location / {
        proxy_pass http://app_backend;
    }
}

server {
    listen 80;
    return 301 https://$host$request_uri;  # HTTP → HTTPS 리다이렉트
}
```

---

## 정리

| 개념 | 설명 |
|------|------|
| 리버스 프록시 | 클라이언트와 서버 사이 중개 |
| Upstream | 백엔드 서버 그룹 정의 |
| Proxy Pass | 요청 전달 대상 |
| Port Mapping | 호스트와 컨테이너 포트 연결 |
| Request Flow | 요청 흐름 추적 |
| Access Log | 요청/응답 기록 |
