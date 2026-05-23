# 데이터 센터와 클라우드 인프라

## 목차

- [데이터 센터와 서버](#데이터-센터와-서버)
- [SSH를 통한 원격 관리](#ssh를-통한-원격-관리)
- [Bare Metal vs Virtual Machine](#bare-metal-vs-virtual-machine)
- [관리 책임 범위](#관리-책임-범위)
- [자동화 운영](#자동화-운영)

---

## 데이터 센터와 서버

데이터 센터는 **서버, 네트워크, 스토리지, 전력, 냉각 설비가 모여 있는 물리적 인프라 공간**이다. 클라우드는 이런 인프라를 사용자가 직접 소유하지 않고 API와 콘솔로 빌려 쓰게 만든 **서비스 모델**이다.

```text
사용자
    ↓ API/콘솔으로 조작
클라우드 (AWS, GCP, Azure)
    ↓ 가상 서버, 네트워크, 스토리지 할당
데이터 센터 (물리 인프라)
    ↓ 실제 하드웨어
```

### Headless Server

**"데이터 센터 서버는 왜 보통 headless로 운영되는가?"**

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

**"Bare Metal과 Virtual Machine은 무엇이 다른가?"**

### Virtual Machine (VM) - AWS EC2

일반 데이터 센터의 물리 서버 위에 Hypervisor를 깔고, 그 위에서 여러 VM을 실행한다.

```
장점:
- 즉시 생성/삭제 가능 (분 단위)
- 격리됨 (다른 사용자와 안전)
- 비용 효율적 (사용량만 지불)
- 자동 백업, 모니터링 지원
- 확장 용이

단점:
- 약간의 성능 오버헤드 (5-15%)
- 하드웨어 커스터마이징 불가
```

### Bare Metal Server

물리 서버를 직접 전유한다. (다른 사용자와 공유 안 함)

```
장점:
- 하드웨어를 직접 제어
- 최고의 성능 (VM 오버헤드 없음)
- 하이퍼바이저 오버헤드 없음
- 특수 하드웨어 사용 가능 (GPU, 고속 네트워크)

단점:
- 느린 생성/삭제 (시간 ~ 하루)
- 높은 비용
- 유지보수 책임 커짐
```

**AWS도 Bare Metal 옵션을 제공합니다** (EC2 Bare Metal Instance). 금융, 머신러닝, 데이터베이스 등 극도의 성능이 필요한 경우 선택한다.

---

## 관리 책임 범위

**"클라우드에서 사용자가 책임지는 범위는 어디까지인가?"**

### 일반 데이터 센터 (당신이 모두 관리)

```
당신의 책임
    ↓
[1. 물리 서버 구매 및 설치]
[2. 데이터 센터 임차]
[3. 냉각, 전원 관리]
[4. 하드웨어 교체 및 업그레이드]
[5. 운영체제 설치 및 관리]
[6. 보안 패치 적용]
[7. 애플리케이션 배포]
    ↓
당신의 책임 100%
```

### AWS EC2 (책임 분담 - Shared Responsibility Model)

```
AWS의 책임                당신의 책임
─────────────────────    ───────────
[물리 서버 구매]          [운영체제 보안 패치]
[데이터 센터 운영]        [애플리케이션 설정]
[냉각, 전원 관리]         [애플리케이션 배포]
[하드웨어 교체]           [방화벽/보안 그룹]
[가상화 (Hypervisor)]     [IAM 권한 관리]
[네트워크 인프라]         [데이터 암호화]
```

**AWS는 물리 인프라를 관리해주므로, 당신은 OS 레벨과 애플리케이션만 신경 쓰면 된다.**

### 서비스별 책임 범위

```
On-Premises (직접 운영)
────────────────────
당신: 모든 것 (100%)

IaaS (AWS EC2, GCP Compute Engine)
───────────────────────────────────
AWS: 물리, 네트워크, 하이퍼바이저
당신: OS, 미들웨어, 애플리케이션

PaaS (AWS Elastic Beanstalk, Heroku)
────────────────────────────────────
플랫폼: 물리, 네트워크, OS, 미들웨어
당신: 애플리케이션만

SaaS (Salesforce, Google Workspace)
──────────────────────────────────
서비스: 모든 것
당신: 데이터와 사용자만
```

---

## 자동화 운영

**"SSH 기반 운영과 자동화 운영은 어떻게 이어지는가?"**

초기에는 SSH로 수동 접속해서 배포하지만, 서버가 많아지면 **자동화 도구**를 사용한다.

### 1. 수동 운영 (초기)

```bash
# 매번 SSH 접속해서 배포
ssh ubuntu@server1.com
cd /app
git pull && npm install && npm run build
systemctl restart app
# server2, server3도 반복...
```

### 2. Shell 스크립트 자동화

```bash
# deploy.sh로 여러 서버에 배포
#!/bin/bash
for server in server1 server2 server3; do
  ssh ubuntu@$server << 'EOF'
    cd /app
    git pull && npm install && npm run build
    systemctl restart app
  EOF
done
```

### 3. 설정 관리 도구 (Ansible, Chef, Puppet)

```yaml
# Ansible playbook으로 선언적 관리
- hosts: all
  tasks:
    - name: Git pull
      shell: cd /app && git pull origin main
    
    - name: Install dependencies
      shell: cd /app && npm install
    
    - name: Build and restart
      shell: cd /app && npm run build && systemctl restart app
```

### 4. 컨테이너 + 오케스트레이션 (Docker + Kubernetes)

```yaml
# Kubernetes Deployment로 자동 배포, 확장, 롤백
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: app
        image: myapp:1.0
        ports:
        - containerPort: 8080
```

### 자동화의 이점

```text
수동 운영                자동화 운영
─────────────          ───────────────
시간 오래 걸림          몇 초 (매우 빠름)
휴먼 에러 가능          일관성 보장
확장 어려움             쉽게 확장 가능 (100대 서버도 동일)
배포 기록 없음          모든 변경사항 추적 가능
야간 긴급 배포 힘듦     언제든지 배포 가능
```

### 흐름도

```text
수동 (SSH) 운영 → Shell 스크립트 → Ansible/Chef → Docker → Kubernetes
(1대)            (2-5대)         (5-20대)      (20-100대) (100+대)
     ↑ 서버 개수 증가 →
```

---

## 정리

| 질문 | 답변 |
|------|------|
| 데이터 센터 노드는 headless인가? | ✓ 맞음 (모니터/키보드 없음) |
| SSH로 접속해서 관리하나? | ✓ 맞음 (유일한 방식) |
| AWS EC2도 같은 방식인가? | ✓ 거의 동일 |
| Bare Metal과 VM의 차이는? | 성능 vs 비용 트레이드오프 |
| 클라우드에서 사용자 책임은? | OS, 미들웨어, 애플리케이션 |
| 자동화는 언제 필요? | 서버가 5대 이상일 때 |

---

## 참고

- **Shared Responsibility Model**: 클라우드 제공사가 어디까지 책임지는지 항상 확인
- **자동화는 선택이 아닌 필수**: 서버가 여러 대면 수동은 불가능
- **인프라 as Code (IaC)**: Terraform, CloudFormation으로 인프라 버전 관리
- **재해 복구**: 클라우드는 리전 분산, 백업으로 높은 가용성 보장

## Related Notes

- [L4 vs L7 로드밸런싱](../load-balancing/overview.md)
- [API Gateway](../api-gateway/overview.md)
