# API Gateway

## 목차

- [API Gateway의 역할](#api-gateway의-역할)
- [API Gateway vs Reverse Proxy](#api-gateway-vs-reverse-proxy)
- [주요 기능](#주요-기능)
- [성능 및 안정성](#성능-및-안정성)
- [마이크로서비스 아키텍처](#마이크로서비스-아키텍처)
- [대표 제품들](#대표-제품들)

---

## API Gateway의 역할

API Gateway는 **여러 내부 서비스 앞에서 외부 요청을 먼저 받는 진입점**이다. 클라이언트는 각 마이크로서비스를 직접 호출하지 않고 Gateway를 호출하며, Gateway가 인증, 라우팅, 제한, 로깅 같은 공통 기능을 처리한 뒤 내부 서비스로 전달한다.

```text
클라이언트
    ↓ HTTP 요청
[API Gateway]
    ├─ 인증 검증
    ├─ 요청 분석
    ├─ 레이트 제한 확인
    ├─ 로깅
    ↓
백엔드 서비스들
    ├─ /users → User Service
    ├─ /orders → Order Service
    └─ /products → Product Service
```

### 필요성

마이크로서비스 아키텍처에서 **API Gateway는 선택이 아닌 필수**다. 없으면 클라이언트가 모든 서비스의 위치를 알아야 하고, 각 서비스가 인증/로깅을 중복으로 구현해야 한다.

---

## API Gateway vs Reverse Proxy

**"API Gateway는 Reverse Proxy와 무엇이 다른가?"**

둘은 **기능이 매우 유사하고, 실제로는 경계가 모호**하다.

### 개념적 차이

**Reverse Proxy (리버스 프록시):**
- 클라이언트와 백엔드 사이에 위치
- **HTTP 레벨**에서 작동
- 캐싱, TLS 종료, 요청/응답 변환 등
- 주로 웹 서버 앞에 배치
- 예: Nginx, Apache

**API Gateway:**
- 클라이언트와 마이크로서비스들 사이에 위치
- **API 레벨**에서 작동
- 인증, 라우팅, Rate limiting, API 버전 관리 등
- 주로 마이크로서비스 아키텍처에서 사용
- 예: Kong, AWS API Gateway, Apigee

### 실제 사용

```text
웹 서버 구성:
클라이언트 → [Nginx: 리버스 프록시] → 웹 앱 서버

마이크로서비스 구성:
클라이언트 → [Kong: API Gateway] → 여러 마이크로서비스

통합:
클라이언트 → [Nginx: 리버스 프록시 역할] → 여러 서비스
  (실제로는 Nginx를 API Gateway처럼 사용)
```

### 정리

- **관점의 차이**: Reverse Proxy는 "웹 서버 기술", API Gateway는 "아키텍처 패턴"
- **기능은 겹침**: 둘 다 요청 라우팅, 캐싱, TLS 등을 지원
- **선택은 상황에 따라**: 마이크로서비스면 API Gateway, 단순 웹 애플리케이션이면 Reverse Proxy

API Gateway는 클라이언트와 백엔드 서비스 사이에 위치하는 **중간 계층(미들웨어)**이다. 모든 API 요청이 여기를 거쳐서 적절한 백엔드로 분배된다.

```
클라이언트 앱
    ↓ HTTP 요청
[API Gateway]
    ├─ 인증 검증
    ├─ 요청 분석
    ├─ 레이트 제한 확인
    ├─ 로깅
    ↓
백엔드 서비스들
    ├─ /users → User Service
    ├─ /orders → Order Service
    └─ /products → Product Service
```

---

## 주요 기능

### 1. 라우팅 (Routing)

URL 경로나 메서드에 따라 요청을 다른 백엔드로 분배한다.

```
GET /api/users → User Service
POST /api/orders → Order Service
GET /api/products → Product Service
```

### 2. 인증/인가 (Authentication & Authorization)

모든 요청의 JWT 토큰, API 키, OAuth를 검증한다.

```
API Gateway에서 한 번 검증하면,
백엔드 서비스들은 이미 검증된 요청만 받음
→ 백엔드 중복 개발 불필요
```

### 3. 요청/응답 변환

- 요청 바디 형식 변환 (JSON ↔ XML)
- 응답 압축 (gzip)
- 헤더 추가/제거

### 4. 레이트 제한 (Rate Limiting)

사용자당 초당 요청 수를 제한해서 DDoS 방어 및 공정한 리소스 사용을 보장한다.

```
일반 사용자: 초당 10개 요청
프리미엄 사용자: 초당 100개 요청
```

### 5. 캐싱

자주 사용하는 응답을 캐시해서 백엔드 부하를 줄인다.

```
GET /api/products (캐시 가능)
→ 첫 요청: 백엔드 호출
→ 같은 요청 반복: 캐시에서 반환
```

### 6. 로깅 & 모니터링

모든 API 요청을 기록해서 분석, 디버깅, 보안 감시에 활용한다.

```
- 요청 시간
- 클라이언트 IP
- 응답 시간
- 에러 코드
```

### 7. 요청 검증

- 필수 파라미터 확인
- 데이터 타입 검증
- URL 경로 유효성 확인

---

## 성능 및 안정성

**"Gateway가 장애나 병목이 되지 않게 하려면 무엇을 고려해야 하는가?"**

API Gateway는 모든 요청이 지나가는 **단일 진입점**이다. 만약 Gateway에 문제가 생기면 모든 서비스가 영향을 받는다. 따라서 다음을 고려해야 한다:

### 1. 높은 가용성

```text
클라이언트
    ↓
[Gateway 1] (Primary)
[Gateway 2] (Standby)
[Gateway 3] (Standby)
    ↓ (로드밸런싱)
백엔드 서비스들
```

- 여러 Gateway 인스턴스 운영 (redundancy)
- 로드밸런싱으로 분산
- Health check로 장애 자동 전환

### 2. 캐싱으로 백엔드 부하 감소

```text
자주 요청되는 데이터 (제품 목록, 설정 등)를 Gateway에 캐시
→ 백엔드 요청 감소
→ 응답 시간 단축
```

### 3. Rate Limiting으로 과부하 방지

```text
일반 사용자: 초당 10개 요청
프리미엄 사용자: 초당 100개 요청
한 IP가 1000개 이상 요청 시 차단
```

### 4. 비동기 처리

- 느린 백엔드 요청을 큐에 넣고 나중에 처리
- 클라이언트는 즉시 응답 받음

### 5. Circuit Breaker 패턴

```text
백엔드 서비스 에러 증가 감지
  → Circuit이 "열림" 상태로 변경
  → 그 서비스로의 요청 차단
  → 대신 캐시된 응답 또는 대체 응답 반환
```

---

## 마이크로서비스 아키텍처

**"내부 서비스 간 통신도 Gateway를 거쳐야 하는가?"**

**답**: 일반적으로 **아니다**. Gateway는 **외부 클라이언트만** 처리한다.

### 아키텍처 패턴

```text
패턴 1: Gateway는 외부 클라이언트만 처리
클라이언트 → [API Gateway] → User Service
User Service → (직접) → Order Service

패턴 2: Service Mesh (고급)
클라이언트 → [API Gateway] → User Service
User Service → [Sidecar Proxy] → Order Service
  (모든 서비스 간 통신이 proxy를 통함, Istio 등)
```

### API Gateway 없을 때 (문제점)

```
클라이언트 ─┬→ User Service (인증, 로깅, Rate limit)
           ├→ Order Service (인증, 로깅, Rate limit)
           └→ Product Service (인증, 로깅, Rate limit)

문제:
- 각 서비스에서 인증, 로깅, Rate limit을 모두 구현 (중복)
- 코드 중복으로 유지보수 어려움
- 클라이언트가 모든 서비스 URL을 알아야 함
- 보안 정책 일관성 없음
```

### API Gateway 있을 때 (해결)

```
클라이언트
    ↓
[API Gateway]
  ├─ 인증 (한 곳에서만!)
  ├─ Rate limiting (한 곳에서만!)
  ├─ 로깅 (한 곳에서만!)
  ├─ 요청 검증 (한 곳에서만!)
    ↓
클라이언트 ─┬→ User Service
           ├→ Order Service
           └→ Product Service

장점:
- 중복 제거
- 백엔드 서비스는 비즈니스 로직에만 집중
- 클라이언트는 Gateway URL만 알면 됨
- 서비스 추가/제거 시 Gateway만 수정
- 보안 정책을 한 곳에서 관리
```

### 인증/인가를 Gateway에서 처리하는 장단점

**장점:**
- 모든 서비스에서 중복 구현 불필요
- 보안 정책 일관성 보장
- 인증 로직 업데이트가 한 곳만 필요
- 느린 인증 처리가 백엔드에 영향 안 함

**단점:**
- Gateway가 모든 인증 요청을 처리 (병목 가능성)
- Gateway와 백엔드의 인증 방식이 다르면 복잡
- 내부 서비스 간 통신에는 별도 인증 필요 (Service-to-Service Auth)

**권장:**
- 외부 API용 인증: Gateway에서
- 내부 서비스 간: Mutual TLS, API Key, Service Mesh 등 별도로

| 용어 | 계층 | 위치 | 역할 |
|------|------|------|------|
| **API Gateway** | L7 | 클라이언트 ↔ 백엔드 | API 요청 라우팅 & 처리 |
| **NAT Gateway** | L3 | 프라이빗 네트워크 ↔ 공인 네트워크 | IP 주소 변환 |
| **Default Gateway** | L3 | 로컬 네트워크 ↔ 다른 네트워크 | 네트워크 경계 |
| **Payment Gateway** | Application | 클라이언트 ↔ 결제 시스템 | 결제 처리 |

API Gateway는 **가장 높은 추상화 계층(L7)**에 있으면서 **API 처리 전문**이다.

---

## 대표 제품들

### AWS API Gateway

클라우드 네이티브 환경에서 가장 많이 사용된다.

```
클라이언트 → API Gateway → Lambda 또는 EC2/ECS/Fargate

장점:
- AWS 서비스와 자동 통합
- 서버리스 환경에 최적화
- 관리형 서비스 (운영 부담 적음)
```

### Kong

오픈소스 API Gateway. 자체 서버에서 운영한다.

```
특징:
- 플러그인 시스템으로 확장 가능
- 마이크로서비스 아키텍처에 최적화
- 높은 커스터마이징 가능
```

### Nginx

원래는 웹 서버지만, Reverse Proxy로 API Gateway 역할도 한다.

```
특징:
- 매우 가볍고 빠름
- 고성능 요청 처리
- 낮은 메모리 사용량
```

### 기타

- Tyk: 엔터프라이즈급 API 관리
- Apigee: Google의 API 관리 플랫폼
- Envoy: 마이크로서비스 프록시

---

## 실전: 마이크로서비스 아키텍처

### API Gateway 없을 때 (문제점)

```
클라이언트 ─┬→ User Service (인증 로직, 속도제한, 로깅)
           ├→ Order Service (인증 로직, 속도제한, 로깅)
           └→ Product Service (인증 로직, 속도제한, 로깅)

문제:
- 각 서비스에서 인증, 로깅, 속도제한을 모두 구현
- 코드 중복
- 유지보수 어려움
- 클라이언트가 모든 서비스 URL을 알아야 함
```

### API Gateway 있을 때 (해결)

```
클라이언트
    ↓
[API Gateway]
  ├─ 인증 (한 곳에서만!)
  ├─ 속도제한 (한 곳에서만!)
  ├─ 로깅 (한 곳에서만!)
    ↓
클라이언트 ─┬→ User Service
           ├→ Order Service
           └→ Product Service

장점:
- 중복 제거
- 백엔드 서비스는 단순화
- 클라이언트는 Gateway URL만 알면 됨
- 서비스 추가/제거 시 Gateway만 수정
```

---

## 다른 "Gateway"들과의 구분

네트워킹에는 다양한 "Gateway"가 있어서 혼동할 수 있다:

| 용어 | 계층 | 위치 | 역할 |
|------|------|------|------|
| **API Gateway** | L7 | 클라이언트 ↔ 마이크로서비스 | API 요청 라우팅 & 처리 |
| **NAT Gateway** | L3 | 프라이빗 네트워크 ↔ 공인 네트워크 | IP 주소 변환 |
| **Default Gateway** | L3 | 로컬 네트워크 ↔ 다른 네트워크 | 네트워크 경계 |
| **Payment Gateway** | Application | 클라이언트 ↔ 결제 시스템 | 결제 처리 |

API Gateway는 **가장 높은 추상화 계층(L7)**에 있으면서 **API 처리 전문**이다.

---

## 대표 제품들

### AWS API Gateway

클라우드 네이티브 환경에서 가장 많이 사용된다.

```
클라이언트 → API Gateway → Lambda 또는 EC2/ECS/Fargate

장점:
- AWS 서비스와 자동 통합
- 서버리스 환경에 최적화
- 관리형 서비스 (운영 부담 적음)
```

### Kong

오픈소스 API Gateway. 자체 서버에서 운영한다.

```
특징:
- 플러그인 시스템으로 확장 가능
- 마이크로서비스 아키텍처에 최적화
- 높은 커스터마이징 가능
```

### Nginx

원래는 웹 서버지만, Reverse Proxy로 API Gateway 역할도 한다.

```
특징:
- 매우 가볍고 빠름
- 고성능 요청 처리
- 낮은 메모리 사용량
```

### 기타

- Tyk: 엔터프라이즈급 API 관리
- Apigee: Google의 API 관리 플랫폼
- Envoy: 마이크로서비스 프록시

---

## 참고

- API Gateway는 선택이 아닌 필수다. 마이크로서비스 아키텍처에서는 거의 반드시 필요하다.
- 단순 모놀리식 애플리케이션에서는 API Gateway가 불필요할 수도 있다.
- API Gateway는 L7(Application) 계층에서 동작하므로, L4 로드밸런서와는 역할이 다르다.
- 작은 팀은 Nginx로 시작, 복잡해지면 Kong이나 AWS API Gateway로 전환하는 경향

## Related Notes

- [L4 vs L7 로드밸런싱](../load-balancing/overview.md)
