# Infrastructure Networking

## 목차
- [VPC와 Subnet](#vpc와-subnet)
- [Routing Table](#routing-table)
- [Security Group과 Firewall](#security-group과-firewall)
- [NAT Gateway](#nat-gateway)
- [아키텍처 설계](#아키텍처-설계)

---

## VPC와 Subnet

**"public subnet과 private subnet은 무엇이 다른가?"**

### VPC (Virtual Private Cloud)

AWS에서 사용자가 정의하는 **격리된 네트워크 공간**이다. 물리 데이터 센터의 LAN을 가상화한 것이다.

```text
AWS Region (지역)
    ↓
[VPC: 172.31.0.0/16] ← 사용자의 격리된 네트워크
    ├─ Subnet A (public, 10.0.1.0/24)
    ├─ Subnet B (private, 10.0.2.0/24)
    └─ Subnet C (private, 10.0.3.0/24)
```

### Public Subnet

**인터넷과 직접 통신 가능한 서브넷**. Internet Gateway를 통해 인터넷으로 나갈 수 있다.

```text
사용자
    ↓ (인터넷)
[Internet Gateway]
    ↓
[Public Subnet]
    ├─ Load Balancer
    ├─ Bastion Host (점프 서버)
    └─ NAT Gateway
```

**사용처:**
- 로드 밸런서
- 웹 서버 (외부 접근 필요)
- Bastion host (다른 서버로의 진입점)

### Private Subnet

**인터넷으로 직접 나갈 수 없는 서브넷**. 내부 통신만 가능하거나, NAT Gateway를 통해서만 외부로 나갈 수 있다.

```text
[Private Subnet]
    ├─ Application Server
    ├─ Database
    └─ Cache Server

    ↓ (아웃바운드)
[NAT Gateway in Public Subnet]
    ↓ (인터넷)
Internet
```

**사용처:**
- 애플리케이션 서버
- 데이터베이스
- 캐시 서버 (Redis)
- 내부 시스템

### 보안상 이점

```text
인터넷 (해킹 위험)
    ↓
[Internet Gateway]
    ↓
[Public Subnet - 공격 대상] (Load Balancer만)
    ↓ (내부 통신)
[Private Subnet - 보호됨] (DB, App)
```

데이터베이스를 private subnet에 두면 인터넷에서 직접 접근 불가능하다.

---

## Routing Table

**Routing Table은 "네트워크 트래픽을 어디로 보낼지" 정의하는 규칙 모음**이다.

```text
목적지                      → 다음 홉
────────────────────          ──────────
10.0.0.0/16 (VPC 내부)       Local
0.0.0.0/0 (모든 외부)        Internet Gateway
```

### 예시

```text
앱 서버 (10.0.2.5)에서 google.com에 접근하려 할 때:

1. 목적지: 142.251.41.14 (google.com IP)
2. Routing Table 확인
   - 10.0.0.0/16? 아니다
   - 0.0.0.0/0? 맞다! → Internet Gateway로 보냄
3. Private Subnet이지만 NAT Gateway를 거쳐 인터넷으로 나감
```

---

## Security Group과 Firewall

**"security group과 firewall은 어떤 역할을 하는가?"**

### Security Group (AWS 방화벽)

각 인스턴스 레벨의 **상태 추적 방화벽**이다.

```yaml
Inbound Rules (들어오는 트래픽):
  - 프로토콜: TCP
  - 포트: 80 (HTTP)
  - 소스: 0.0.0.0/0 (모두 허용)

  - 프로토콜: TCP
  - 포트: 443 (HTTPS)
  - 소스: 0.0.0.0/0

  - 프로토콜: TCP
  - 포트: 22 (SSH)
  - 소스: 203.0.113.0/24 (특정 IP만)

Outbound Rules (나가는 트래픽):
  - 모든 프로토콜, 모든 포트, 모든 목적지 (기본값)
```

### Network ACL (네트워크 방화벽)

Subnet 레벨의 **stateless 방화벽**이다.

```text
Security Group     vs     Network ACL
─────────────────         ──────────────
인스턴스별                Subnet별
상태 추적 (Stateful)      상태 미추적 (Stateless)
허용 규칙만               허용/거부 규칙
기본: 모두 거부           기본: 모두 허용
작은 범위                 큰 범위
```

**실제로는 Security Group만 사용해도 대부분 충분하다.**

---

## NAT Gateway

**"NAT gateway는 왜 필요한가?"**

Private Subnet의 서버가 **아웃바운드** (나가는 요청)만 필요할 때 사용한다.

```text
[Private Subnet App Server]
    ↓ (https://api.github.com/user)
    아웃바운드 요청

[NAT Gateway in Public Subnet]
    ↓ (출발지 IP를 Public Subnet IP로 변환)
GitHub
```

### 역할

1. **아웃바운드 인터넷 접근**: npm install, apt update 등
2. **인바운드 차단**: 외부에서 직접 접근 불가
3. **IP 마스킹**: 내부 IP 보호

### 비용

```text
NAT Gateway 시간당 비용: $0.045 (약 한 달 $32)
NAT Gateway 데이터 전송 비용: $0.045 per GB

→ 규모가 크면 비용이 많이 들 수 있다
```

---

## 아키텍처 설계

**"load balancer는 어느 계층에서 동작하는가?"**

→ L4 로드 밸런서: EC2 네트워크 레벨에서 동작
→ L7 로드 밸런서: Application 레벨에서 동작

### 완전한 아키텍처 예시

```text
┌─ Internet
│
└─ Internet Gateway
   │
   └─ Public Subnet (10.0.1.0/24)
      ├─ Load Balancer (L7 ALB)
      └─ NAT Gateway (아웃바운드용)
      │
      └─ Private Subnet (10.0.2.0/24) [AZ-A]
         ├─ App Server 1 (10.0.2.10)
         ├─ App Server 2 (10.0.2.11)
         └─ App Server 3 (10.0.2.12)
      │
      └─ Private Subnet (10.0.3.0/24) [AZ-B]
         ├─ Database 1 (primary)
         ├─ Database 2 (read replica)
         └─ Cache (Redis)
```

### 흐름

```text
클라이언트
    ↓
인터넷 (HTTP 요청)
    ↓
Internet Gateway
    ↓
Public Subnet의 Load Balancer (L7 라우팅)
    ↓
Private Subnet의 App Server
    ├─ 데이터 조회 → Private Subnet의 DB
    └─ 외부 API 호출 → NAT Gateway → 인터넷
```

---

## 정리

| 질문 | 답변 |
|------|------|
| Public/Private Subnet 차이? | 인터넷 직접 접근 가능 여부 |
| Security Group 역할? | 인스턴스 레벨 방화벽 |
| NAT Gateway 역할? | Private에서 아웃바운드 인터넷 접근 |
| Load Balancer 위치? | Public Subnet (클라이언트 직접 접근) |

---

## Related Notes

- [L4 vs L7 로드밸런싱](../load-balancing/overview.md)
- [데이터 센터와 클라우드 인프라](../data-center-and-cloud/overview.md)
