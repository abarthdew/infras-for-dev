# Project: Redis with Docker

## 목차
- 목표
- 필요한 개념
- 실행 흐름
- 검증 방법

## 기초 개념
이 프로젝트는 Docker로 Redis를 실행하고, 포트 매핑과 간단한 명령으로 Redis 동작을 확인하는 실습이다.

```text
docker run -> Redis container -> redis-cli
```

## 간단한 예시
```bash
docker run --name redis-study -p 6379:6379 redis:alpine
redis-cli SET hello world
redis-cli GET hello
```

## 반드시 알아야 할 질문
- 컨테이너 포트와 호스트 포트는 무엇이 다른가?
- Redis 데이터는 컨테이너 삭제 후 어떻게 되는가?
- volume은 왜 필요한가?
