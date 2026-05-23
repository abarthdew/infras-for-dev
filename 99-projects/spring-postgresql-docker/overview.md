# Project: Spring PostgreSQL Docker

## 목차
- [프로젝트 목표](#프로젝트-목표)
- [Spring 애플리케이션](#spring-애플리케이션)
- [PostgreSQL 컨테이너](#postgresql-컨테이너)
- [환경 변수](#환경-변수)
- [Docker Network](#docker-network)
- [DB 마이그레이션](#db-마이그레이션)
- [연결 확인](#연결-확인)
- [문제 해결](#문제-해결)

---

## 프로젝트 목표

**Spring 애플리케이션이 Docker 컨테이너에서 실행 중인 PostgreSQL 데이터베이스에 연결하고, 환경 변수를 이용해 다양한 환경을 관리**한다.

```text
아키텍처:
Spring Application → JDBC Driver → PostgreSQL Container
                                      (localhost:5432)
```

### 학습 목표

```text
1. Spring Data JPA로 PostgreSQL 연결
2. 환경 변수 기반 설정 분리
3. Docker Network 이해
4. DB 마이그레이션 자동화
5. 애플리케이션과 DB 라이프사이클 관리
```

---

## Spring 애플리케이션

### Spring Boot 프로젝트 구조

```text
spring-postgresql-docker/
├── pom.xml
├── src/
│   ├── main/
│   │   ├── java/com/example/demo/
│   │   │   ├── DemoApplication.java
│   │   │   ├── entity/
│   │   │   │   └── User.java
│   │   │   ├── repository/
│   │   │   │   └── UserRepository.java
│   │   │   └── controller/
│   │   │       └── UserController.java
│   │   └── resources/
│   │       └── application.properties
│   └── test/
├── Dockerfile
└── docker-compose.yml
```

### 의존성 설정 (pom.xml)

```xml
<dependencies>
    <!-- Spring Boot Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- Spring Data JPA -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <!-- PostgreSQL Driver -->
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <version>42.7.0</version>
        <scope>runtime</scope>
    </dependency>

    <!-- Flyway (마이그레이션) -->
    <dependency>
        <groupId>org.flywaydb</groupId>
        <artifactId>flyway-core</artifactId>
    </dependency>
</dependencies>
```

### 엔티티 정의

```java
// User.java
package com.example.demo.entity;

import jakarta.persistence.*;

@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String name;
    
    @Column(unique = true, nullable = false)
    private String email;
    
    private String phone;
    
    // Constructor, Getter, Setter
}
```

### Repository 정의

```java
// UserRepository.java
package com.example.demo.repository;

import com.example.demo.entity.User;
import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User, Long> {
    User findByEmail(String email);
}
```

### Controller 정의

```java
// UserController.java
package com.example.demo.controller;

import com.example.demo.entity.User;
import com.example.demo.repository.UserRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/users")
public class UserController {
    @Autowired
    private UserRepository userRepository;
    
    @GetMapping
    public List<User> getAll() {
        return userRepository.findAll();
    }
    
    @GetMapping("/{id}")
    public User getById(@PathVariable Long id) {
        return userRepository.findById(id).orElse(null);
    }
    
    @PostMapping
    public User create(@RequestBody User user) {
        return userRepository.save(user);
    }
}
```

---

## PostgreSQL 컨테이너

### PostgreSQL 이미지 선택

```text
postgres:15-alpine (권장)
- 크기: 작음 (~150MB)
- 성능: 좋음
- 보안: 안정적

postgres:15 (표준)
- 크기: 중간 (~350MB)
- 성능: 표준
- 기능: 더 많은 도구 포함
```

### 컨테이너 실행

```bash
# 간단한 방식
docker run -d \
  --name postgres-dev \
  -e POSTGRES_USER=developer \
  -e POSTGRES_PASSWORD=password123 \
  -e POSTGRES_DB=app_db \
  -p 5432:5432 \
  -v postgres-data:/var/lib/postgresql/data \
  postgres:15-alpine

# 확인
docker ps
docker logs postgres-dev
```

### PostgreSQL 접속

```bash
# psql 클라이언트로 접속
docker exec -it postgres-dev psql -U developer -d app_db

# SQL 쿼리 실행
docker exec postgres-dev psql -U developer -d app_db -c "SELECT version();"
```

---

## 환경 변수

### "애플리케이션 설정은 환경별로 어떻게 분리하는가?"

**Spring의 프로필(profile) 기능을 이용해 개발, 테스트, 운영 환경별 설정을 분리**한다.

```text
기본 설정:
application.properties
- 공통 설정

환경별 설정:
application-dev.properties
- 개발: localhost, 디버그 활성화
application-test.properties
- 테스트: H2 인메모리 DB
application-prod.properties
- 운영: 보안, 최적화
```

### application.properties 구조

```properties
# application.properties (공통)
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=false
spring.datasource.hikari.maximum-pool-size=10
```

```properties
# application-dev.properties (개발)
spring.datasource.url=jdbc:postgresql://localhost:5432/app_db
spring.datasource.username=developer
spring.datasource.password=password123
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
logging.level.root=INFO
logging.level.com.example.demo=DEBUG
```

```properties
# application-prod.properties (운영)
spring.datasource.url=jdbc:postgresql://postgres-prod:5432/app_db
spring.datasource.username=${DB_USER}
spring.datasource.password=${DB_PASSWORD}
spring.jpa.show-sql=false
logging.level.root=WARN
server.error.include-message=never
```

### 환경 변수 주입

```bash
# 환경 변수로 설정 오버라이드
java -jar app.jar \
  --spring.profiles.active=prod \
  --spring.datasource.url=jdbc:postgresql://db-host:5432/db_name \
  --spring.datasource.username=${DB_USER} \
  --spring.datasource.password=${DB_PASSWORD}

# Docker 실행
docker run -e SPRING_PROFILES_ACTIVE=prod \
  -e SPRING_DATASOURCE_URL=jdbc:postgresql://postgres:5432/app_db \
  -e SPRING_DATASOURCE_USERNAME=user \
  -e SPRING_DATASOURCE_PASSWORD=password \
  app:latest
```

### application.yml (YAML 형식)

```yaml
# application-dev.yml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/app_db
    username: developer
    password: password123
    hikari:
      maximum-pool-size: 10
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true
    properties:
      hibernate:
        format_sql: true
logging:
  level:
    com.example.demo: DEBUG
```

---

## Docker Network

### "Docker network 안의 hostname은 어떻게 정해지는가?"

**Docker Compose로 실행한 컨테이너들은 자동으로 DNS를 통해 서비스명으로 통신**한다.

```text
Docker Compose 네트워크:
- 각 서비스마다 DNS entry 생성
- 서비스명 = hostname
- 컨테이너가 시작되면 자동 등록

spring-app → "postgres:5432"로 통신
→ Docker DNS: postgres = 172.20.0.2
→ 자동 연결
```

### docker-compose.yml

```yaml
version: '3.8'

services:
  # PostgreSQL 데이터베이스
  postgres:
    image: postgres:15-alpine
    container_name: postgres-dev
    environment:
      POSTGRES_USER: developer
      POSTGRES_PASSWORD: password123
      POSTGRES_DB: app_db
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U developer"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - app-network

  # Spring 애플리케이션
  spring-app:
    build: .
    container_name: spring-app
    depends_on:
      postgres:
        condition: service_healthy
    environment:
      SPRING_PROFILES_ACTIVE: docker
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/app_db
      SPRING_DATASOURCE_USERNAME: developer
      SPRING_DATASOURCE_PASSWORD: password123
      SPRING_JPA_HIBERNATE_DDL_AUTO: validate
    ports:
      - "8080:8080"
    networks:
      - app-network

volumes:
  postgres-data:
    driver: local

networks:
  app-network:
    driver: bridge
```

### Dockerfile (Spring)

```dockerfile
# 멀티 스테이지 빌드
FROM maven:3.8.6-openjdk-17 AS builder

WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline

COPY src src
RUN mvn clean package -DskipTests

# 실행 이미지
FROM openjdk:17-slim

WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar

EXPOSE 8080
CMD ["java", "-jar", "app.jar"]
```

### 시작 및 확인

```bash
# 시작
docker-compose up -d

# 로그 보기
docker-compose logs -f

# 상태 확인
docker-compose ps

# 컨테이너 간 통신 확인
docker exec spring-app curl http://postgres:5432
docker exec spring-app nc -zv postgres 5432
```

---

## DB 마이그레이션

### "DB migration은 언제 실행해야 하는가?"

**마이그레이션은 애플리케이션 시작 전에 자동으로 실행**되어야 한다. Flyway를 사용하면 스키마 변경을 자동화할 수 있다.

```text
마이그레이션 타이밍:
애플리케이션 시작
    ↓
Spring 부팅
    ↓
Flyway: 마이그레이션 확인
    ↓
DB 스키마 업그레이드 (필요시)
    ↓
데이터베이스 준비 완료
    ↓
Spring: 트랜잭션 실행
```

### Flyway 설정

```properties
# application.properties
spring.flyway.locations=classpath:db/migration
spring.flyway.baseline-on-migrate=true

# 또는 Hibernate DDL (개발 환경)
spring.jpa.hibernate.ddl-auto=update  # 자동 스키마 생성 (주의 필요)
```

### 마이그레이션 파일 생성

```text
src/main/resources/db/migration/
├── V1__create_users_table.sql
├── V2__add_phone_column.sql
└── V3__create_products_table.sql

명명 규칙:
V{version}__{description}.sql
- V1: 버전
- __: 구분자
- description: 설명
```

### 마이그레이션 파일

```sql
-- V1__create_users_table.sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    phone VARCHAR(20),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_users_email ON users(email);
```

```sql
-- V2__add_age_column.sql
ALTER TABLE users ADD COLUMN age INT;
ALTER TABLE users ADD CONSTRAINT check_age CHECK (age > 0);
```

### SQL 마이그레이션 vs JPA Hibernate

```text
Hibernate DDL (자동 생성):
spring.jpa.hibernate.ddl-auto=create-drop
- 개발: 편함
- 운영: 위험 (데이터 손실)

Flyway (SQL 기반):
- 버전 관리
- 되돌리기 가능 (Undo 마이그레이션)
- 운영: 안전

권장:
- 개발: Hibernate (빠른 프로토타입)
- 운영: Flyway (버전 관리)
```

---

## 연결 확인

### Spring 애플리케이션 접속

```bash
# 로그에서 포트 확인
docker logs spring-app | grep "Started"
# Tomcat started on port(s): 8080

# 헬스 체크
curl http://localhost:8080/actuator/health

# API 테스트
curl -X POST http://localhost:8080/api/users \
  -H "Content-Type: application/json" \
  -d '{"name": "John", "email": "john@example.com"}'

curl http://localhost:8080/api/users
```

### 데이터베이스 연결 확인

```bash
# PostgreSQL 접속
docker exec -it postgres-dev psql -U developer -d app_db

# SQL 확인
SELECT * FROM users;
SELECT * FROM flyway_schema_history;
```

### 로그 확인

```bash
# Spring 로그
docker-compose logs spring-app | grep -i "error"
docker-compose logs spring-app | grep "DataSource"

# PostgreSQL 로그
docker-compose logs postgres | grep "connection"
```

---

## 문제 해결

### 연결 실패

```
Error: Failed to obtain JDBC Connection
```

**원인: 호스트명, 포트, 자격증명 오류**

```bash
# 확인 체크리스트
1. PostgreSQL 컨테이너 실행 중?
   docker ps | grep postgres

2. 호스트명 올바른가? (docker-compose에서는 서비스명)
   SPRING_DATASOURCE_URL=jdbc:postgresql://postgres:5432/app_db

3. 자격증명 일치?
   POSTGRES_USER=developer
   POSTGRES_PASSWORD=password123

4. 포트 확인
   docker port postgres
   # 5432/tcp -> 0.0.0.0:5432

5. 네트워크 확인
   docker network ls
   docker inspect app-network
```

### 마이그레이션 실패

```
Error: Flyway migration failed
```

```bash
# 마이그레이션 상태 확인
docker exec postgres-dev psql -U developer -d app_db -c \
  "SELECT * FROM flyway_schema_history;"

# 문제 해결
1. 이전 마이그레이션 성공했는가?
2. SQL 문법 오류 없는가?
3. 권한 문제 없는가?

# 리셋 (주의: 데이터 손실!)
docker-compose down -v
docker-compose up -d
```

### Dependency

```
No PostgreSQL driver found
```

```xml
<!-- pom.xml에 추가 -->
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>
</dependency>
```

---

## 다음 단계

### Liquibase 사용

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.liquibase</groupId>
    <artifactId>liquibase-core</artifactId>
</dependency>
```

### 환경 파일 분리

```bash
.env (로컬)
.env.prod (운영)

docker-compose.yml
docker-compose.prod.yml
```

### 모니터링 추가

```yaml
services:
  # ... 기존 설정 ...
  
  pgadmin:
    image: dpage/pgadmin4
    ports:
      - "5050:80"
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@example.com
      PGADMIN_DEFAULT_PASSWORD: admin
    depends_on:
      - postgres
```

---

## 정리

| 개념 | 설명 |
|------|------|
| Spring Boot | 자동 설정 프레임워크 |
| JPA | 객체-관계 매핑 |
| PostgreSQL | 관계형 데이터베이스 |
| JDBC | 데이터베이스 연결 드라이버 |
| Flyway | DB 마이그레이션 도구 |
| Docker Compose | 다중 컨테이너 관리 |
| Profile | 환경별 설정 분리 |
