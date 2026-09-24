# 로드밸런싱 (L4 vs L7)

## 목차

- [로드밸런싱의 역할](#로드밸런싱의-역할)
- [L4 로드밸런서](#l4-로드밸런서)
- [L7 로드밸런서](#l7-로드밸런서)
- [Health Check와 Sticky Session](#health-check와-sticky-session)
- [로드밸런서와 리버스 프록시](#로드밸런서와-리버스-프록시)
- [실전 사용 기준](#실전-사용-기준)

---

## 로드밸런싱의 역할

로드밸런싱은 **여러 서버로 요청을 나누어 보내 가용성과 처리량을 높이는 인프라 기술**이다. 서버 한 대에 모든 요청이 몰리면 장애와 병목이 생기기 쉬우므로 앞단에서 요청을 분산한다.

```text
client
-> load balancer
-> server 1 / server 2 / server 3
```

### OSI 7계층과 로드밸런싱

네트워크 통신은 OSI(Open Systems Interconnection) 7계층 모델로 표현된다. 로드밸런서는 어느 계층에서 동작하는지에 따라 **L4 로드밸런서**와 **L7 로드밸런서**로 나뉜다.

| 계층 | 이름 | 주요 역할 | 프로토콜 예시 |
|------|------|----------|-------------|
| L7 | Application | 사용자 애플리케이션 | HTTP, HTTPS, DNS |
| L6 | Presentation | 데이터 형식, 암호화 | TLS/SSL |
| L5 | Session | 세션 관리 | - |
| **L4** | **Transport** | **포트 기반 전송** | **TCP, UDP** |
| L3 | Network | IP 주소 라우팅 | IP |
| L2 | Data Link | MAC 주소 | Ethernet |
| L1 | Physical | 실제 신호/케이블 | - |

**"L4와 L7은 어떤 계층 정보를 보고 분산하는가?"**
- **L4**: IP 주소와 포트 번호만 봄 (Transport 계층)
- **L7**: URL, HTTP 헤더, 쿠키 등 애플리케이션 정보까지 분석 (Application 계층)

네트워크 통신은 OSI(Open Systems Interconnection) 7계층 모델로 표현된다. 각 계층은 추상화 수준이 다르며, 상위 계층일수록 더 많은 정보를 다룬다.

| 계층 | 이름 | 주요 역할 | 프로토콜 예시 |
|------|------|----------|-------------|
| L7 | Application | 사용자 애플리케이션 | HTTP, HTTPS, DNS |
| L6 | Presentation | 데이터 형식, 암호화 | TLS/SSL |
| L5 | Session | 세션 관리 | - |
| **L4** | **Transport** | **포트 기반 전송** | **TCP, UDP** |
| L3 | Network | IP 주소 라우팅 | IP |
| L2 | Data Link | MAC 주소 | Ethernet |
| L1 | Physical | 실제 신호/케이블 | - |

---

## L4 로드밸런서

### 작동 원리

L4는 Transport 계층에서 동작하므로, **IP 주소와 포트 번호**만을 기준으로 트래픽을 분산한다. 패킷의 실제 내용(HTTP body, URL 등)은 열어보지 않는다.

```
클라이언트 (192.168.1.100)
    ↓ TCP 연결 요청 (IP:Port 정보만 확인)
[L4 로드밸런서]
    ├→ 192.168.0.10:8080 (서버 A)
    ├→ 192.168.0.11:8080 (서버 B)
    └→ 192.168.0.12:8080 (서버 C)
```

### 특징

- **빠름**: 패킷 내용을 분석하지 않으므로 지연 시간 최소
- **가벼움**: CPU 사용량 적음
- **높은 처리량**: 많은 연결 동시 처리 가능
- **제한된 라우팅**: IP/Port 기준의 단순 분산만 가능

### 분산 알고리즘

- Round Robin: 순차적으로 서버에 할당
- Least Connections: 활성 연결이 가장 적은 서버로
- IP Hash: 클라이언트 IP 기반으로 항상 같은 서버로

### 대표 제품

- AWS NLB (Network Load Balancer)
- HAProxy (TCP mode)
- Linux IPVS (IP Virtual Server)

---

## L7 로드밸런서

### 작동 원리

L7은 Application 계층에서 동작하므로, **HTTP 헤더, URL, 쿠키** 등 애플리케이션 수준의 정보를 분석해서 라우팅할 수 있다.

```
[L7 로드밸런서]
    ├─ GET /api/* → API 서버
    ├─ GET /images/* → 이미지 서버
    ├─ Host: admin.example.com → 관리자 서버
    └─ Cookie: user_type=premium → 프리미엄 서버
```

### 특징

- **지능형 라우팅**: URL, 헤더, 쿠키 기반의 세밀한 분산
- **콘텐츠 기반 라우팅**: 요청 내용에 따라 다른 서버로 보냄
- **SSL/TLS 처리**: HTTPS를 복호화해서 분석 가능
- **상대적으로 느림**: 패킷 내용을 분석하므로 지연 발생
- **높은 CPU 사용**: 더 많은 계산 필요

### 라우팅 규칙 예시

```
조건: URL 경로로 분산
/api/ → 백엔드 API 서버
/static/ → CDN 또는 정적 파일 서버

조건: 호스트명으로 분산
api.example.com → API 서버
admin.example.com → 관리자 서버

조건: HTTP 헤더로 분산
X-User-Type: premium → 프리미엄 서버
```

### 대표 제품

- AWS ALB (Application Load Balancer)
- Nginx (reverse proxy)
- Apache HTTP Server
- HAProxy (HTTP mode)

---

## L4 vs L7 비교

| 항목 | L4 | L7 |
|------|-----|-----|
| 기준 | IP + Port | URL, 헤더, 쿠키 |
| 속도 | 빠름 | 상대적으로 느림 |
| 처리량 | 매우 높음 | 중간~높음 |
| 라우팅 정교성 | 낮음 | 높음 |
| CPU 사용량 | 낮음 | 높음 |
| 사용 사례 | 게임, 실시간 통신 | 웹 애플리케이션 |

---

## Health Check와 Sticky Session

### Health Check (헬스 체크)

**"Health check는 왜 필요한가?"**

로드밸런서는 백엔드 서버의 상태를 주기적으로 점검해서 **정상 서버에만 요청을 보낸다**. 만약 한 서버가 장애나면, 로드밸런서가 자동으로 그 서버로의 요청을 멈춘다.

```text
로드밸런서
  ↓ (주기적으로 점검)
서버 A: 200 OK (정상)
서버 B: Connection Refused (장애)
서버 C: 200 OK (정상)
  ↓ (A, C에만 요청 분산)
```

**구현 방식:**

```
TCP Health Check:  포트 연결 확인 (빠름, 기본적)
HTTP Health Check: GET /health → 200 응답 확인 (정확함)
Custom Health Check: 애플리케이션 정의 엔드포인트
```

Health Check가 없으면 장애난 서버로 계속 요청을 보내 고객 경험이 악화된다.

### Sticky Session

**"Sticky session은 언제 문제가 되는가?"**

Sticky session은 같은 클라이언트의 모든 요청을 **같은 서버로 보내는 기능**이다.

```text
클라이언트 A
  → 요청 1 → 서버 1
  → 요청 2 → 서버 1 (같은 서버)
  → 요청 3 → 서버 1
```

**필요한 경우:**
- 서버 메모리에 세션 데이터 저장 (로그인 정보 등)
- 클라이언트별 상태 유지가 필요

**문제가 되는 경우:**
- 로드 분산이 제대로 안 됨 (특정 서버에 요청이 몰림)
- 서버 1이 장애나면 세션 데이터 유실 (Sticky session이 도움이 안 됨)
- 서버 증설/감축이 어려움

**대안:**
- 세션을 Redis 같은 별도 저장소에 저장 → Sticky session 필요 없음
- Session affinity 대신 **Session sharing** 사용

---

## 로드밸런서와 리버스 프록시

**"로드밸런서와 리버스 프록시는 어떻게 겹치고 다른가?"**

이 둘은 **기능이 겹치는 부분이 많고, 실제로는 구분이 모호**하다.

### 정의

**Reverse Proxy (리버스 프록시):**
- 클라이언트와 백엔드 서버 사이에 위치
- 캐싱, TLS 종료, 요청/응답 변환 등 **HTTP 레벨**에서 작동
- 주로 **Nginx, Apache**

**Load Balancer (로드밸런서):**
- 여러 서버에 **요청을 분산**하는 것이 주 목적
- L4, L7 모두 가능

### 실제는?

```text
Nginx (Reverse Proxy)
  ├─ HTTP 레벨에서 요청 처리
  ├─ 캐싱
  ├─ TLS 종료
  └─ 로드 밸런싱 기능도 있음

AWS ALB (Load Balancer)
  ├─ L7에서 요청 분산 (로드밸런싱이 주)
  ├─ 캐싱은 없음
  └─ 리버스 프록시 기능도 제공

HAProxy (Load Balancer & Reverse Proxy)
  ├─ L4, L7 모두 지원
  ├─ 로드 밸런싱
  └─ 리버스 프록시 기능도 함
```

### 구분 기준

| | Reverse Proxy | Load Balancer |
|---|---|---|
| 주 목적 | 요청 처리/캐싱 | 요청 분산 |
| 사용처 | 웹 서버 앞 | 여러 백엔드 앞 |
| 기능 | HTTP 레벨 | L4, L7 모두 가능 |
| 예시 | Nginx, Apache | AWS NLB, ALB, HAProxy |

**실제 사용:**
- Nginx를 리버스 프록시 + 간단한 로드 밸런서로 사용
- AWS ALB/NLB를 로드 밸런서로 사용하고, 각 서버에 Nginx를 리버스 프록시로 둔다
- 둘의 기능이 겹치므로 상황에 맞게 선택

---

## 실전 사용 기준

### L4를 선택해야 할 때

- 극도로 높은 처리량이 필요한 경우 (금융 거래, 게임)
- TCP/UDP 기반 실시간 통신 (VoIP, 온라인 게임)
- 요청 내용 분석이 필요 없는 경우
- 비용이 중요한 경우 (L4가 저렴)

### L7을 선택해야 할 때

- 웹 애플리케이션 (일반적인 웹 서비스)
- 마이크로서비스 아키텍처 (경로 기반 라우팅)
- A/B 테스트 (특정 사용자 그룹을 다른 서버로)
- 캐싱 및 압축 (응답 최적화)
- 최근 추세 (클라우드 네이티브, 컨테이너)

---

## 참고

- OSI 모델은 이론적 모델이며, 실제 로드밸런서는 여러 계층을 결합해서 동작한다.
- 최근 클라우드 환경에서는 L7(ALB)이 더 많이 사용되는 추세다 (마이크로서비스, 컨테이너 기반).
- 극단적인 성능이 필요한 경우에만 L4를 고려한다.
- 로드 밸런서 선택은 성능, 비용, 운영 편의성을 함께 고려해야 한다.
