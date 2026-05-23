# API Gateway

## 목차

- API Gateway의 역할
- Gateway와 Reverse Proxy
- 인증, 라우팅, Rate Limit
- 마이크로서비스에서의 위치

## 기초 개념

API Gateway는 여러 내부 서비스 앞에서 외부 요청을 먼저 받는 진입점이다. 클라이언트는 각 마이크로서비스를 직접 호출하지 않고 Gateway를 호출하며, Gateway가 인증, 라우팅, 제한, 로깅 같은 공통 기능을 처리한 뒤 내부 서비스로 전달한다.

```text
client
-> API Gateway
-> service A / service B / service C
```

## 간단한 예시

```text
GET /users/1
-> API Gateway
-> user-service /users/1
```

## 반드시 알아야 할 질문

- API Gateway는 Reverse Proxy와 무엇이 다른가?
- 인증과 인가를 Gateway에서 처리하면 어떤 장단점이 있는가?
- Gateway가 장애나 병목이 되지 않게 하려면 무엇을 고려해야 하는가?
- 내부 서비스 간 통신도 Gateway를 거쳐야 하는가?

## 세부 노트

## 개념

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

## "Gateway"와의 혼동

네트워킹에는 다양한 "Gateway"가 있어서 헷갈릴 수 있다:

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

## 참고

- API Gateway는 선택이 아닌 필수다. 마이크로서비스 아키텍처에서는 거의 반드시 필요하다.
- 단순 모놀리식 애플리케이션에서는 API Gateway가 불필요할 수도 있다.
- API Gateway는 l7(Application) 계층에서 동작하므로, L4 로드밸런서와는 역할이 다르다.

## Related Notes

- [L4 vs L7 로드밸런싱](../load-balancing/overview.md)
