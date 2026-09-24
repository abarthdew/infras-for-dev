# Deployment

## 목차
- [Deployment의 역할](#deployment의-역할)
- [Build와 Release](#build와-release)
- [Artifact](#artifact)
- [Environment](#environment)
- [Rollback](#rollback)
- [Blue-Green Deployment](#blue-green-deployment)
- [Canary Deployment](#canary-deployment)

---

## Deployment의 역할

**Deployment는 개발된 애플리케이션을 사용자가 접근 가능한 환경에 안전하게 반영하는 과정**이다. 핵심은 **재현 가능성**, **점진적 반영**, **빠른 복구**이다.

```text
Source Code
    ↓ (Build)
Artifact (빌드 결과물)
    ↓ (배포)
Dev Environment
    ↓ (테스트)
Staging Environment
    ↓ (최종 검증)
Production Environment
    ↓ (모니터링)
사용자
```

### 배포의 위험성

```text
배포 시 문제점:
1. 서비스 중단 가능 (사용자 영향)
2. 롤백이 어려움 (이전 상태 복구 불가)
3. 무결성 깨짐 (일부만 배포되는 경우)
4. 환경 차이 (dev에선 작동하는데 prod에선 안 됨)

→ 배포 전략과 자동화로 해결
```

---

## Build와 Release

### Build란?

**Build는 소스 코드를 실행 가능한 형태로 변환하는 과정**이다.

```text
Source Code (사람이 읽을 수 있는 형태)
    ↓ (컴파일, 테스트, 최적화)
Compiled Binary / Docker Image
    ↓
실행 가능한 Artifact
```

### Build 과정

```bash
# 예: Java 프로젝트
mvn clean package
# 1. 소스 컴파일 (Java → .class)
# 2. 의존성 다운로드
# 3. 테스트 실행
# 4. JAR 생성

# 예: Docker 프로젝트
docker build -t app:1.0 .
# 1. Dockerfile 읽음
# 2. 레이어별 실행
# 3. 이미지 생성
# 4. 레지스트리에 push
```

### Release란?

**Release는 빌드된 artifact를 특정 버전으로 표시하고 배포 가능한 상태로 준비하는 과정**이다.

```bash
# Version Tagging
git tag v1.0.0
git push origin v1.0.0

# 또는 Release Notes와 함께
gh release create v1.0.0 -t "Version 1.0.0" -n "Release notes here"
```

---

## Artifact

### "build artifact는 왜 필요한가?"

**Artifact는 빌드된 결과물로, 언제든 같은 결과를 재현할 수 있도록 보장**한다.

```text
문제 (Artifact 없이):
소스 코드 → 매번 빌드 → 매번 다를 수 있음
(의존성 버전 차이, 빌드 환경 차이 등)

해결 (Artifact 사용):
소스 코드 1회 빌드 → Artifact 생성
Artifact → Dev 배포 ✓ (동일)
Artifact → Staging 배포 ✓ (동일)
Artifact → Prod 배포 ✓ (동일)
```

### Artifact의 종류

```text
언어별 Artifact:
- Java: JAR, WAR
- Python: Wheel (.whl)
- Node.js: tar.gz 패키지
- Go: 바이너리 파일

컨테이너 환경:
- Docker Image (가장 일반적)
- Kubernetes Deployment YAML
```

### Artifact 저장소

```bash
# Docker Image Registry
docker build -t my-app:1.0 .
docker tag my-app:1.0 registry.example.com/my-app:1.0
docker push registry.example.com/my-app:1.0

# Artifact Repository (Nexus, Artifactory)
# JAR, Wheel 등 바이너리 저장

# Git Tag (소스 코드 버전)
git tag v1.0.0
git push origin v1.0.0
```

### Artifact 검증

```bash
# 빌드한 Docker Image 테스트
docker run --rm my-app:1.0 python -m pytest

# JAR 실행 테스트
java -jar app-1.0.jar --version

# 바이너리 체크섬으로 무결성 검증
sha256sum app-binary > app-binary.sha256
# 나중에 검증
sha256sum -c app-binary.sha256
```

---

## Environment

### "환경별 설정은 어떻게 분리할 것인가?"

**환경별로 데이터베이스, API 엔드포인트, 로그 레벨 등이 다르므로 설정을 분리**해야 한다.

```text
개발 환경 (Dev)
- 로컬 머신 또는 개발 서버
- 테스트 데이터
- 상세한 로깅
- 느림 (최적화 불필요)

스테이징 환경 (Staging)
- 프로덕션과 동일한 인프라
- 테스트 데이터
- 중간 로깅
- 배포 전 최종 검증

프로덕션 환경 (Production)
- 실제 사용자 접근
- 실제 데이터
- 최소한의 로깅 (성능 중요)
- 고가용성 필수
```

### 설정 분리 방식

```bash
# 1. 환경 변수 사용
export APP_ENV=production
export DB_HOST=prod-db.example.com
export LOG_LEVEL=warn

# 애플리케이션에서
db_host = os.environ.get('DB_HOST')

# 2. 설정 파일 분리
config/
├─ dev.yaml
├─ staging.yaml
└─ prod.yaml

# 3. Docker 환경
docker run \
  -e APP_ENV=production \
  -e DB_HOST=prod-db.example.com \
  my-app:1.0

# 4. Kubernetes ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config-prod
data:
  APP_ENV: "production"
  DB_HOST: "prod-db.example.com"
  LOG_LEVEL: "warn"
```

### 환경별 검증

```bash
# Dev에서 테스트
APP_ENV=dev pytest

# Staging에서 테스트
APP_ENV=staging pytest -m "integration"

# Prod 배포 전 체크리스트
- 로그 레벨: warn 이상만 ✓
- DB: 프로덕션 DB 연결 ✓
- API: 프로덕션 엔드포인트 ✓
- 캐싱: 활성화됨 ✓
```

---

## Rollback

### "rollback은 어떤 조건에서 실행해야 하는가?"

**Rollback은 배포 후 문제가 감지되면 이전 버전으로 복구하는 것**이다.

```text
배포 후 모니터링:
- 에러율 증가? → Rollback
- 응답 시간 악화? → Rollback
- 특정 기능 동작 안 함? → Rollback
- 데이터 손상? → Rollback (더 신속)
```

### Rollback 전략

```bash
# 1. 이전 버전 Docker Image로 즉시 재배포
kubectl set image deployment/app app=my-app:v1.0.0

# 2. 이전 Kubernetes Deployment로 복원
kubectl rollout undo deployment/app

# 3. Git으로 이전 commit으로 롤백
git revert <bad-commit-hash>
git push origin main

# 4. 데이터베이스 스냅샷 복원 (필요 시)
# 백업에서 복구
```

### Rollback 시간

```text
Rollback 가능 시간에 따른 영향:

1분 이내 (대부분 안전)
- 사용자 경험 큰 영향 없음
- 손상된 데이터 최소화

5분 이상 (중간 정도)
- 일부 사용자 영향
- 데이터 관계 검증 필요

30분 이상 (위험)
- 광범위한 영향
- 데이터 복구 고려
```

### 자동 Rollback

```yaml
# Kubernetes에서 자동 Rollback
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  progressDeadlineSeconds: 600  # 10분 내 ready 아니면 rollback
  replicas: 3
  selector:
    matchLabels:
      app: app
  template:
    spec:
      containers:
      - name: app
        image: my-app:1.0
```

---

## Blue-Green Deployment

### Blue-Green Deployment란?

**"무중단 배포는 어떤 문제를 해결하는가?"**

**Blue-Green Deployment는 동일한 환경 두 개(Blue, Green)에서 한쪽은 현재 서비스, 다른 한쪽은 새 버전 준비 후 한 번에 전환하는 전략**이다.

```text
현재 상태 (Blue):
- 실제 트래픽 처리
- v1.0 실행 중

새 버전 준비 (Green):
- v2.0 배포
- 테스트 완료
- 트래픽 0

전환:
Load Balancer → Green으로 라우팅 변경
    ↓
Blue (v1.0): 롤백 준비 상태로 유지
Green (v2.0): 실제 트래픽 처리

문제 발생 시:
Load Balancer → Blue로 복구 (즉시)
```

### 구현 예시

```bash
# 1. 현재 버전 확인
kubectl get deployment app
# NAME  READY  IMAGE
# app   3/3    my-app:1.0 (Blue)

# 2. Green 환경에 새 버전 배포
kubectl create deployment app-green --image=my-app:2.0 --replicas=3

# 3. Green 검증
kubectl exec -it pod/app-green-xxx -- /healthcheck

# 4. 라우팅 전환
kubectl patch service app -p '{"spec":{"selector":{"deployment":"app-green"}}}'

# 5. 모니터링
# 에러 없으면 Blue 삭제
kubectl delete deployment app

# 에러 발생하면 즉시 복구
kubectl patch service app -p '{"spec":{"selector":{"deployment":"app-blue"}}}'
```

### 장단점

```text
장점:
- 무중단 (사용자 체감 0초)
- 즉각적 롤백 가능
- 완전한 환경 테스트 가능

단점:
- 2배 인프라 필요
- 디스크 공간 2배 필요
- 상태 동기화 필요 (캐시, 세션)
```

---

## Canary Deployment

### Canary Deployment란?

**Canary Deployment는 새 버전을 소수의 사용자에게 먼저 배포하고, 문제 없으면 점진적으로 확대하는 전략**이다.

```text
현재 (v1.0): 100%
새 버전 (v2.0): 0%
    ↓
v1.0: 95%
v2.0: 5% (카나리 배포)
    ↓ (메트릭 모니터링 OK)
v1.0: 50%
v2.0: 50% (절반씩)
    ↓ (메트릭 모니터링 OK)
v1.0: 0%
v2.0: 100% (완전 전환)
```

### 구현 예시

```yaml
# Istio 사용
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: app
spec:
  hosts:
  - app.example.com
  http:
  - match:
    - headers:
        user-id:
          regex: "canary-.*"
    route:
    - destination:
        host: app
        subset: v2
  - route:
    - destination:
        host: app
        subset: v1
      weight: 95
    - destination:
        host: app
        subset: v2
      weight: 5
```

```bash
# 카나리 그룹 정의 (5%)
# user_id가 "canary-"로 시작하는 사용자는 v2 사용

# 메트릭 모니터링
- 에러율 비교 (v1 vs v2)
- 응답 시간 비교
- 특정 기능 로그 분석

# 이상 없으면 가중치 조정
# 95:5 → 50:50 → 0:100
```

### Canary 모니터링 지표

```text
다음 지표들을 모니터링해서 롤백 결정:

에러율:
- v1 에러율: 0.1%
- v2 에러율: 0.5% → 문제! Rollback

응답 시간:
- v1 p99: 100ms
- v2 p99: 500ms → 문제! Rollback

사용자 만족도:
- v1 thumbs-up: 95%
- v2 thumbs-up: 80% → 문제! Rollback
```

### Canary vs Blue-Green

| | Canary | Blue-Green |
|---|---|---|
| 위험도 | 낮음 (일부 영향) | 높음 (전체 영향) |
| 롤백 속도 | 느림 (점진적) | 빠름 (즉시) |
| 메트릭 수집 | 필수 | 선택 |
| 인프라 필요량 | 1.05배 | 2배 |
| 사용 사례 | 대규모 변경 | 긴급 배포 |

---

## 실전 배포 절차

```bash
# 1. 로컬에서 테스트
npm test
npm run build

# 2. Staging에 배포
git tag v1.1.0
git push origin v1.1.0

# CI/CD가 자동으로:
# - 빌드
# - 테스트
# - Docker Image 생성
# - Staging에 배포

# 3. Staging 검증
# - Smoke test 실행
# - 성능 테스트
# - E2E 테스트

# 4. 승인 및 Prod 배포
# Blue-Green 또는 Canary 전략 선택
kubectl apply -f prod-deployment.yaml

# 5. 모니터링
# 에러율, 응답 시간, 로그 확인

# 6. Rollback 준비
# 문제 발생 시 즉시 실행
kubectl rollout undo deployment/app
```

---

## 정리

| 개념 | 설명 |
|------|------|
| Build | 소스 코드를 실행 가능한 형태로 변환 |
| Artifact | 빌드된 결과물 (Docker Image, JAR 등) |
| Environment | Dev, Staging, Production 환경 분리 |
| Rollback | 문제 발생 시 이전 버전으로 복구 |
| Blue-Green | 두 환경 전환으로 무중단 배포 |
| Canary | 일부 사용자부터 점진적 배포 |

---

## Related Notes

- [GitHub Actions](../github-actions/overview.md)
- [Git](../git/overview.md)
