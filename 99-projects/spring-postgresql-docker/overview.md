# Project: Spring PostgreSQL Docker

## 목차
- 목표
- Spring 앱
- PostgreSQL 컨테이너
- 환경 변수
- 연결 확인

## 기초 개념
이 프로젝트는 Spring 애플리케이션이 Docker로 실행한 PostgreSQL에 연결하는 기본 구조를 학습한다.

```text
Spring app -> JDBC -> PostgreSQL container
```

## 간단한 예시
```text
DB_HOST=localhost
DB_PORT=5432
DB_NAME=study
```

## 반드시 알아야 할 질문
- 애플리케이션 설정은 환경별로 어떻게 분리하는가?
- Docker network 안의 hostname은 어떻게 정해지는가?
- DB migration은 언제 실행해야 하는가?
