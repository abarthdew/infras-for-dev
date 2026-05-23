# Docker

## 목차
- [Docker의 역할](#docker의-역할)
- [Image와 Container](#image와-container)
- [Dockerfile](#dockerfile)
- [Volume](#volume)
- [Network와 Port Mapping](#network와-port-mapping)
- [Docker Compose](#docker-compose)

---

## Docker의 역할

Docker는 **애플리케이션과 실행 환경을 이미지로 묶고, 컨테이너라는 격리된 프로세스 환경에서 실행**하게 해주는 도구다.

```text
Dockerfile
    ↓ (docker build)
Image (읽기 전용, 패키지화)
    ↓ (docker run)
Container (실행 중인 프로세스, 격리)
```

---

## Image와 Container

**"image와 container는 무엇이 다른가?"**

### Image

**클래스처럼 정의된 템플릿**. 불변이고 재사용 가능하다.

```text
특징:
- 읽기 전용 (변경 불가)
- 재사용 가능 (여러 컨테이너 생성 가능)
- 계층화 (layer 기반 구성)
- 배포 단위

예: myapp:1.0 이미지는 영원히 같다
```

### Container

**Image로부터 실행된 인스턴스**. 프로세스처럼 시작/중지할 수 있다.

```text
특징:
- 실행 중인 프로세스
- 독립적 파일시스템
- 네트워크 격리
- 임시적 (종료하면 삭제 가능)

예: myapp:1.0에서 3개 컨테이너 실행 가능
```

### VM과의 차이

**"container는 VM과 무엇이 다른가?"**

```text
VM (Virtual Machine)           Container (Docker)
──────────────────────         ────────────────
Guest OS 포함                  Host OS 공유
시작 시간: 분                   시작 시간: 초
크기: GB                       크기: MB
자원 사용: 많음                 자원 사용: 적음
격리 수준: 높음                 격리 수준: 중간
예: AWS EC2                    예: Docker Container
```

**흐름:**
```text
Host OS
├─ VM (Ubuntu OS 포함 - 자체 커널)
│  └─ App
├─ VM (CentOS OS 포함)
│  └─ App
└─ Docker
   ├─ Container (OS 불포함, Host 커널 공유)
   │  └─ App
   └─ Container
      └─ App
```

---

## Dockerfile

**Dockerfile은 이미지를 구성하는 명령어들의 집합**이다.

```dockerfile
FROM ubuntu:20.04
# Base image 지정

RUN apt-get update && apt-get install -y python3
# 명령어 실행 (설치 등)

WORKDIR /app
# 작업 디렉토리 설정

COPY . .
# 호스트의 파일을 이미지로 복사

EXPOSE 8000
# 포트 노출 (문서화용)

CMD ["python3", "app.py"]
# 컨테이너 시작 시 실행 명령어
```

### 빌드 및 실행

```bash
# 이미지 빌드
docker build -t myapp:1.0 .

# 컨테이너 실행
docker run -d -p 8000:8000 myapp:1.0
```

---

## Volume

**"volume은 왜 필요한가?"**

컨테이너는 기본적으로 임시적이다. 컨테이너가 종료되면 모든 데이터가 사라진다.

### 문제점

```bash
docker run mydb:latest
# DB 데이터 저장 → 컨테이너 내부 파일시스템
docker stop mydb
# 컨테이너 삭제 → 모든 데이터 유실 😱
```

### Volume 솔루션

```bash
# Named Volume
docker run -d -v mydata:/var/lib/postgresql mydb:latest
# /var/lib/postgresql 디렉토리를 호스트의 mydata에 연결

# Host Directory Binding
docker run -d -v /home/user/data:/var/lib/postgresql mydb:latest
# 호스트의 /home/user/data와 연결
```

---

## Network와 Port Mapping

**"port mapping은 어떤 방향으로 연결하는가?"**

컨테이너는 자체 IP를 가지며, 외부에서 직접 접근 불가능하다. Port mapping으로 호스트 포트와 연결한다.

```text
호스트                컨테이너
──────               ────────
localhost:8000  →    127.0.0.1:3000 (앱 포트)

-p 8000:3000
   ↑    ↑
   │    └─ 컨테이너 포트
   └────── 호스트 포트
```

### 네트워크 연결

```bash
# 컨테이너 간 통신
docker network create mynet
docker run --network mynet --name web myapp:1.0
docker run --network mynet --name db mydb:1.0

# web 컨테이너에서 db로 접근
curl http://db:5432
```

---

## Docker Compose

**여러 컨테이너를 한 번에 관리하는 도구**이다.

```yaml
version: '3'
services:
  web:
    image: myapp:1.0
    ports:
      - "8000:3000"
    environment:
      - DB_HOST=db
    depends_on:
      - db
  
  db:
    image: postgres:13
    environment:
      - POSTGRES_PASSWORD=secret
    volumes:
      - dbdata:/var/lib/postgresql
    
  redis:
    image: redis:7
    ports:
      - "6379:6379"

volumes:
  dbdata:
```

```bash
docker-compose up      # 시작
docker-compose down    # 중지 및 삭제
docker-compose logs    # 로그 보기
```

---

## 정리

| 개념 | 설명 |
|------|------|
| Image | 불변 템플릿 (클래스처럼) |
| Container | 실행 중인 인스턴스 (객체처럼) |
| Volume | 영구 데이터 저장 |
| Port Mapping | 호스트 포트 ↔ 컨테이너 포트 |
| Docker Compose | 다중 컨테이너 관리 |

---

## Related Notes

- [데이터 센터와 클라우드 인프라](../data-center-and-cloud/overview.md)
- [Kubernetes](../kubernetes/overview.md)
