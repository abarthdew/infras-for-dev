# Project: Redis with Docker

## 목차
- [프로젝트 목표](#프로젝트-목표)
- [Redis 개념](#redis-개념)
- [Docker 실행](#docker-실행)
- [Redis CLI 사용](#redis-cli-사용)
- [데이터 영속성](#데이터-영속성)
- [검증 방법](#검증-방법)
- [문제 해결](#문제-해결)

---

## 프로젝트 목표

**Docker를 이용해 Redis를 실행하고, 포트 매핑, 데이터 저장, 볼륨 마운트를 통해 Redis의 기본 개념을 이해**한다.

```text
학습 목표:
1. Docker 이미지 실행
2. 포트 매핑의 의미
3. 컨테이너 데이터 영속성
4. 볼륨 사용
```

---

## Redis 개념

### Redis란?

```text
Redis (REmote DIctionary Server):
- 메모리 기반 데이터 저장소
- Key-Value 형식
- 매우 빠른 읽기/쓰기
- 캐시, 세션, 실시간 데이터에 사용

특징:
✓ 메모리 저장 (매우 빠름)
✓ 단순한 자료구조 (String, List, Set, Hash)
✓ TTL (Time To Live) 지원
✓ Pub/Sub 메시징
✗ 메모리에만 저장 (휘발성)
```

### 데이터 구조

```text
String (문자열):
key: "username"
value: "john"

List (리스트):
key: "tasks"
value: ["task1", "task2", "task3"]

Set (집합):
key: "tags"
value: {"python", "redis", "docker"}

Hash (해시):
key: "user:1"
value: {name: "john", age: 30, email: "john@example.com"}

Sorted Set (정렬된 집합):
key: "scores"
value: {john: 100, jane: 95, bob: 85}
(점수 기준 정렬)
```

---

## Docker 실행

### 기본 실행

```bash
# Redis 컨테이너 실행 (가장 간단한 방식)
docker run --name redis-study -p 6379:6379 redis:alpine

# 백그라운드 실행
docker run -d --name redis-study -p 6379:6379 redis:alpine

# 확인
docker ps
docker logs redis-study
```

### "컨테이너 포트와 호스트 포트는 무엇이 다른가?"

**호스트 포트는 실제 시스템 포트, 컨테이너 포트는 컨테이너 내부 포트**이다. 포트 매핑으로 둘을 연결한다.

```text
호스트 (컴퓨터):
포트 6379 ← 외부 접속

         ↓ 포트 매핑 (-p 6379:6379)

컨테이너 (Redis):
포트 6379 (Redis 기본 포트)

포트 매핑 없으면:
호스트에서 localhost:6379로 접속 불가능
컨테이너 내부에서만 접속 가능
```

### 포트 매핑 옵션

```bash
# 모든 인터페이스에서 수신
docker run -p 6379:6379 redis:alpine
# 외부: 0.0.0.0:6379

# localhost에서만 수신 (안전)
docker run -p 127.0.0.1:6379:6379 redis:alpine
# 외부: 127.0.0.1:6379

# 다른 호스트 포트 사용
docker run -p 6380:6379 redis:alpine
# 호스트:6380 → 컨테이너:6379
```

### Docker Compose로 실행

```yaml
# docker-compose.yml
version: '3.8'

services:
  redis:
    image: redis:alpine
    container_name: redis-study
    ports:
      - "6379:6379"  # 호스트:컨테이너
    volumes:
      - redis-data:/data
    command: redis-server --appendonly yes
    restart: unless-stopped

volumes:
  redis-data:
    driver: local
```

### Docker Compose 사용

```bash
# 시작
docker-compose up -d

# 로그 보기
docker-compose logs -f redis

# 중지
docker-compose down

# 볼륨 포함 완전 삭제
docker-compose down -v
```

---

## Redis CLI 사용

### redis-cli 접속

```bash
# 방법 1: 호스트에서 직접 (redis-cli 설치 필요)
redis-cli -h localhost -p 6379

# 방법 2: 컨테이너 내부에서
docker exec -it redis-study redis-cli

# 방법 3: 대화형 쉘
docker run -it redis:alpine redis-cli -h host.docker.internal -p 6379
```

### 기본 명령어

```bash
# String
SET key1 "Hello"
GET key1                    # "Hello"
APPEND key1 " World"
GET key1                    # "Hello World"
STRLEN key1                 # 11

# Key 관리
EXISTS key1                 # 1 (존재)
DEL key1                    # 1 (삭제됨)
KEYS *                      # 모든 key 조회
FLUSHALL                    # 모든 데이터 삭제 (주의!)

# 만료 시간 (TTL)
SET session_id "abc123" EX 3600  # 1시간 후 자동 삭제
TTL session_id              # 3599 (남은 시간)
EXPIRE key1 60              # 60초 후 삭제

# List
LPUSH mylist "a"            # 왼쪽에 추가
RPUSH mylist "b"            # 오른쪽에 추가
LRANGE mylist 0 -1          # ["a", "b"]
LPOP mylist                 # "a" (제거)

# Set
SADD myset "python"
SADD myset "redis"
SMEMBERS myset              # {"python", "redis"}
SISMEMBER myset "python"    # 1 (포함)

# Hash
HSET user:1 name "john" age 30
HGET user:1 name            # "john"
HGETALL user:1              # {name: "john", age: 30}
```

### 모니터링

```bash
# 실시간 명령 모니터링
MONITOR

# 서버 정보
INFO
INFO memory
INFO stats

# 메모리 사용
DBSIZE                      # key 개수
INFO memory                 # 메모리 사용량
MEMORY USAGE key1           # 특정 key 메모리
```

---

## 데이터 영속성

### "Redis 데이터는 컨테이너 삭제 후 어떻게 되는가?"

**Redis 데이터는 메모리에 저장되므로, 컨테이너가 삭제되면 데이터도 사라진다. 영속성을 위해서는 볼륨 마운트나 RDB/AOF 저장이 필요**하다.

```text
영속성 없이:
컨테이너 생성
→ SET data
→ docker rm redis-study
→ 데이터 손실!

영속성 있이:
docker run -v redis-data:/data redis:alpine
→ /data에 저장된 데이터는 유지
→ 컨테이너 재시작해도 복구
```

### "volume은 왜 필요한가?"

**Volume은 컨테이너가 삭제되어도 데이터를 호스트에 보존**한다.

```text
볼륨 없이:
컨테이너 A (메모리: key=value)
→ docker rm (모두 삭제)

볼륨 있이:
컨테이너 A (메모리)
    ↓ 마운트
호스트 /var/lib/docker/volumes/redis-data/
    (디스크에 저장, 삭제 안 됨)
```

### RDB vs AOF

```text
RDB (Snapshot):
- 주기적으로 메모리 전체 저장
- 빠르지만 변경 손실 가능
- 파일: dump.rdb
- 설정: save 900 1 (15분마다 1개 변경 시)

AOF (Append Only File):
- 모든 명령 기록
- 느리지만 데이터 손실 최소
- 파일: appendonly.aof
- 설정: appendonly yes
```

### 영속성 설정

```bash
# RDB + AOF 둘 다 사용
docker run \
  -d \
  --name redis-study \
  -p 6379:6379 \
  -v redis-data:/data \
  redis:alpine \
  redis-server --appendonly yes --save 900 1

# docker-compose.yml
services:
  redis:
    image: redis:alpine
    volumes:
      - redis-data:/data
    command: redis-server --appendonly yes
```

### 볼륨 확인

```bash
# 볼륨 목록
docker volume ls

# 볼륨 세부 정보
docker volume inspect redis-data

# 호스트 경로
# Linux: /var/lib/docker/volumes/redis-data/_data
# Mac: Docker Desktop VM 내부
# Windows: WSL2 VM 내부
```

---

## 검증 방법

### 동작 확인

```bash
# 컨테이너 실행
docker run -d --name redis-test -p 6379:6379 redis:alpine

# 데이터 저장
docker exec redis-test redis-cli SET hello world

# 데이터 확인
docker exec redis-test redis-cli GET hello
# 출력: "world"

# 성공!
docker stop redis-test
```

### 성능 테스트

```bash
# 벤치마크
docker exec redis-test redis-benchmark -n 1000 -c 10
# 10개 동시 연결로 1000개 요청

# 메모리 사용량 확인
docker exec redis-test redis-cli INFO memory

# 응답 예시:
# used_memory: 1045632 bytes (1MB)
# used_memory_human: 1.00M
```

### 연결 확인

```bash
# Redis CLI 접속
docker exec -it redis-test redis-cli

# 핑 테스트
PING                        # PONG
echo "Hello" | docker exec -i redis-test redis-cli
```

---

## 문제 해결

### 포트 충돌

```bash
# 포트 사용 중 에러
docker: Error response from daemon: Ports are not available

# 해결 방법
# 1) 다른 포트 사용
docker run -p 6380:6379 redis:alpine

# 2) 기존 컨테이너 중지
docker stop <container_id>

# 3) 포트 확인
netstat -tulpn | grep 6379
lsof -i :6379
```

### 데이터 접근 불가

```bash
# 호스트에서 접속 안 될 때
redis-cli ping
# 에러: Could not connect

# 확인
docker ps (컨테이너 실행 중?)
docker logs redis-test (에러 메시지?)

# 포트 매핑 확인
docker port redis-test
# 출력: 6379/tcp -> 0.0.0.0:6379

# localhost 대신 127.0.0.1 시도
redis-cli -h 127.0.0.1 ping
```

### 메모리 부족

```bash
# 메모리 제한 설정
docker run \
  -m 512m \              # 512MB 제한
  -p 6379:6379 \
  redis:alpine

# 확인
docker stats redis-study
```

---

## 다음 단계

### Redis Python 연결

```python
import redis

# 연결
r = redis.Redis(host='localhost', port=6379, decode_responses=True)

# 데이터 저장
r.set('name', 'john')
r.set('score', 100)

# 데이터 조회
print(r.get('name'))    # john
print(r.get('score'))   # 100

# 만료 시간
r.setex('session', 3600, 'abc123')

# List 사용
r.rpush('tasks', 'task1', 'task2')
print(r.lrange('tasks', 0, -1))
```

### 캐싱 사용

```python
def get_user(user_id):
    # 캐시에서 먼저 확인
    cached = r.get(f'user:{user_id}')
    if cached:
        return json.loads(cached)
    
    # DB에서 조회
    user = db.query(f"SELECT * FROM users WHERE id = {user_id}")
    
    # 캐시에 저장 (1시간)
    r.setex(f'user:{user_id}', 3600, json.dumps(user))
    
    return user
```

---

## 정리

| 개념 | 설명 |
|------|------|
| Redis | 메모리 기반 Key-Value 저장소 |
| Docker | 컨테이너로 Redis 실행 |
| Port Mapping | 호스트 포트와 컨테이너 포트 연결 |
| Volume | 데이터 영속성 보장 |
| RDB | 스냅샷 기반 영속성 |
| AOF | 명령 기반 영속성 |
| TTL | Key 만료 시간 |
