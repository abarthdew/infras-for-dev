# 데이터 센터와 클라우드 인프라

## 데이터 센터의 서버

### Headless Server

데이터 센터의 모든 서버는 **headless** 상태다.

- **Headless**: 모니터, 키보드, 마우스가 없음
- 물리적으로 접근할 수 없음
- **원격으로만 관리** (SSH를 통해)

```
일반 PC (모니터/키보드 있음)
    ↓
[모니터에서 작업]

데이터 센터 서버 (모니터 없음)
    ↓
[원격(SSH)에서만 작업]
```

### "Server" vs "PC"

| PC | Server |
|----|--------|
| 개인 사용 | 24/7 운영 |
| 가끔 켜고 끔 | 항상 실행 |
| 보통 1-2개 CPU | 다중 CPU (8, 16, 32+개) |
| 8-16GB 메모리 | 수십~수백 GB 메모리 |
| 높은 신뢰성 불필요 | 극도로 높은 신뢰성 필수 |

데이터 센터의 노드는 **"Headless PC"가 아니라 "Server"**입니다.

---

## SSH를 통한 원격 관리

### SSH란?

**SSH (Secure Shell)**는 네트워크를 통해 **원격 서버에 안전하게 접속하는 프로토콜**입니다.

```
노트북
    ↓ SSH 연결 (암호화)
[데이터 센터 서버]
    ├─ 앱 배포
    ├─ 패키지 설치
    ├─ 로그 확인
    ├─ 프로세스 재시작
    └─ 설정 파일 수정
```

### 실제 작업 예시

```bash
# 1. SSH로 원격 서버 접속
ssh ubuntu@192.168.0.10
# 패스워드 또는 키 입력

# 2. 서버에서 직접 명령 실행
cd /home/ubuntu/myapp
git pull origin main
npm install
npm run build

# 3. 서비스 재시작
systemctl restart myapp

# 4. 로그 확인
tail -f /var/log/myapp.log

# 5. 종료
exit
```

---

## AWS EC2와의 비교

### EC2는 무엇인가?

**EC2 (Elastic Compute Cloud)**는 AWS에서 제공하는 **클라우드 기반 가상 서버**다.

```
AWS 데이터 센터 (물리 서버)
    ↓
[Hypervisor - 여러 VM을 동시에 실행]
    ├─ EC2 Instance A (당신의 VM)
    ├─ EC2 Instance B (다른 사용자 VM)
    └─ EC2 Instance C (또 다른 사용자 VM)
```

### 일반 데이터 센터 vs AWS EC2

| 항목 | 일반 데이터 센터 | AWS EC2 |
|------|-----------------|---------|
| 서버 타입 | 물리 서버 (Bare Metal) | 가상 서버 (VM) |
| 인프라 구축 | 직접 구매, 설치, 관리 | AWS가 모두 관리 |
| 접속 방식 | SSH | SSH (동일) |
| 초기 비용 | 매우 높음 | 없음 (클릭으로 생성) |
| 운영 비용 | 고정비 | 사용량 기반 |
| 확장성 | 물리적 제약 | 즉시 확장 |
| 냉각, 전원 관리 | 직접 | AWS가 관리 |

---

## 관리 책임 범위 (Shared Responsibility)

### 일반 데이터 센터 (당신이 모두 관리)

```
당신의 책임
    ↓
[1. 물리 서버 구매]
[2. 데이터 센터 임차]
[3. 냉각, 전원 관리]
[4. 하드웨어 교체]
[5. 운영체제 설치]
[6. 보안 패치]
[7. 앱 배포]
    ↓
당신의 책임
```

### AWS EC2 (책임 분담)

```
AWS의 책임
    ↓
[1. 물리 서버 구매 및 관리]
[2. 데이터 센터 운영]
[3. 냉각, 전원 관리]
[4. 하드웨어 교체]
[5. 가상화 (Hypervisor)]
    ↓
당신의 책임
    ↓
[1. 운영체제 보안 패치]
[2. 앱 설정]
[3. 앱 배포]
[4. 방화벽 규칙]
    ↓
당신의 책임
```

**AWS는 물리 인프라를 관리해주므로, 당신은 OS와 앱만 신경 쓰면 된다.**

---

## 실제 사용 방식은 거의 동일

### 일반 데이터 센터에서 배포

```bash
ssh ubuntu@203.0.113.10

cd /opt/myapp
git pull origin main
npm install
npm run build
systemctl restart myapp
```

### AWS EC2에서 배포

```bash
ssh -i my-key.pem ubuntu@ec2-54-123-45-67.compute-1.amazonaws.com

cd /opt/myapp
git pull origin main
npm install
npm run build
systemctl restart myapp
```

**사용자 입장에서는 동일합니다.** 내부적으로 물리 인프라 vs 가상 인프라일 뿐입니다.

---

## Bare Metal vs Virtual Machine

### Virtual Machine (VM) - AWS EC2

```
장점:
- 즉시 생성/삭제 가능
- 격리됨 (다른 사용자와 안전)
- 비용 효율적 (사용량만 지불)
- 자동 백업, 모니터링 지원

단점:
- 약간의 성능 오버헤드
- 하드웨어 커스터마이징 불가
```

### Bare Metal Server

```
장점:
- 하드웨어를 직접 제어
- 최고의 성능
- 하이퍼바이저 오버헤드 없음

단점:
- 느린 생성/삭제
- 높은 비용
- 유지보수 책임 커짐
```

**AWS도 Bare Metal 옵션을 제공합니다** (EC2 Bare Metal Instance).

---

## 정리

| 질문 | 답변 |
|------|------|
| 데이터 센터 노드는 headless인가? | ✓ 맞음 |
| SSH로 접속해서 관리하나? | ✓ 맞음 (유일한 방식) |
| AWS EC2도 같은 방식인가? | ✓ 거의 동일 |
| 차이점은? | AWS가 물리 인프라를 관리해줌 |

---

## Related Notes

- [L4 vs L7 로드밸런싱](load-balancing.md)
- [API Gateway](api-gateway.md)
