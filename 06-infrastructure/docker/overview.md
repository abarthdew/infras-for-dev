# Docker

## 목차
- image와 container
- Dockerfile
- volume
- network
- port mapping
- compose

## 기초 개념
Docker는 애플리케이션과 실행 환경을 이미지로 묶고, 컨테이너라는 격리된 프로세스 환경에서 실행하게 해주는 도구다.

```text
Dockerfile -> image -> container
```

## 간단한 예시
```dockerfile
FROM nginx:alpine
COPY ./html /usr/share/nginx/html
```

```bash
docker build -t my-nginx .
docker run -p 8080:80 my-nginx
```

## 반드시 알아야 할 질문
- image와 container는 무엇이 다른가?
- container는 VM과 무엇이 다른가?
- volume은 왜 필요한가?
- port mapping은 어떤 방향으로 연결하는가?
