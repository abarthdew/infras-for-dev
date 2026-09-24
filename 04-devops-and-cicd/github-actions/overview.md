# GitHub Actions

## 목차
- [GitHub Actions의 역할](#github-actions의-역할)
- [Workflow](#workflow)
- [Event](#event)
- [Job](#job)
- [Step](#step)
- [Runner](#runner)
- [Artifact](#artifact)
- [Secret](#secret)
- [CI와 CD](#ci와-cd)

---

## GitHub Actions의 역할

**GitHub Actions는 GitHub 이벤트를 기반으로 테스트, 빌드, 배포 같은 자동화 작업을 실행하는 CI/CD 도구**다. 개발자가 코드를 push할 때마다 자동으로 테스트하고, 통과하면 배포하는 식으로 진행된다.

```text
개발자
    ↓ git push
GitHub Repository
    ↓ (push event 감지)
GitHub Actions (자동 실행)
    ├─ 테스트
    ├─ 빌드
    └─ 배포
    ↓
프로덕션 환경
```

### GitHub Actions 없이는

```bash
# 수동으로 하는 작업들:
$ git clone https://github.com/user/repo
$ cd repo
$ npm install
$ npm test
$ npm run build
$ docker build -t app:1.0 .
$ docker push registry.example.com/app:1.0
$ kubectl set image deployment/app app=app:1.0
# 모든 팀원이 손으로 반복하거나, 한 사람이 매번 수동 실행
```

---

## Workflow

### "workflow와 job은 무엇이 다른가?"

**Workflow는 자동화 전체 프로세스를 정의하는 YAML 파일**이고, **Job은 workflow 내에서 실행되는 작업 단위**다.

```text
Workflow (자동화 전체 프로세스)
├─ Job 1 (테스트) - 실행 환경: Ubuntu
│  ├─ Step 1 (checkout)
│  ├─ Step 2 (npm install)
│  └─ Step 3 (npm test)
├─ Job 2 (빌드) - 실행 환경: Ubuntu
│  ├─ Step 1 (checkout)
│  ├─ Step 2 (npm build)
│  └─ Step 3 (docker build)
└─ Job 3 (배포) - 실행 환경: Ubuntu
   ├─ Step 1 (checkout)
   └─ Step 2 (kubectl deploy)
```

### Workflow 파일

```yaml
# .github/workflows/ci-cd.yml

name: CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm install
      - run: npm test

  build:
    needs: test  # test job이 완료된 후 실행
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm run build
      - run: docker build -t app:${{ github.sha }} .

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: kubectl set image deployment/app app=app:${{ github.sha }}
```

### Workflow 상태

```text
Triggered (실행 시작)
    ↓
In Progress (실행 중)
    ├─ Job 1 실행
    ├─ Job 2 대기 (Job 1 완료 후)
    └─ Job 3 대기
    ↓
Completed (완료)
    ├─ All passed ✓
    ├─ Some failed ✗
    └─ Cancelled
```

---

## Event

### Event란?

**Event는 Workflow를 실행하는 trigger**다.

```yaml
on:
  push:
    branches: [main, develop]
    paths: ['src/**']
  
  pull_request:
    types: [opened, synchronize, reopened]
  
  schedule:
    - cron: '0 0 * * *'  # 매일 자정
  
  workflow_dispatch:  # 수동 실행
  
  release:
    types: [published]
```

### 주요 Event

```text
push
- 코드를 main에 push 시 실행
- 가장 자주 사용됨

pull_request
- PR 생성/업데이트 시 실행
- 코드 리뷰 전 테스트

schedule
- Cron 표현식으로 주기적 실행
- 정기적 backup, 보안 스캔

workflow_dispatch
- 수동으로 GitHub UI에서 실행
- 긴급 배포 등

repository_dispatch
- 외부 이벤트 (webhook)로 실행
- 다른 시스템에서 트리거
```

### Event 필터링

```yaml
on:
  push:
    # main branch에만 실행
    branches: [main]
    
    # src 디렉토리 변경 시만 실행
    paths: ['src/**']
    
    # 제외 (이 경로 변경 시 실행 안 함)
    paths-ignore: ['docs/**', '*.md']
```

---

## Job

### Job이란?

**Job은 같은 Runner에서 실행되는 Step들의 모음**이다.

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - run: npm test
  
  build:
    runs-on: ubuntu-latest
    needs: test  # test job 완료 후 실행
    steps:
      - run: npm run build
```

### Job 의존성

```text
기본 (병렬 실행):
test  ┐
build ├─ 동시 실행
deploy┘

의존성 지정:
test
  ↓
build (test 완료 후)
  ↓
deploy (build 완료 후)
```

### Job 조건

```yaml
jobs:
  deploy:
    # 이전 job들이 모두 성공했을 때만 실행 (기본값)
    if: success()
    
    # 이전 job들 중 하나라도 실패했을 때만 실행 (cleanup 용도)
    # if: failure()
    
    # 항상 실행 (job 성공/실패 무관)
    # if: always()
    
    steps:
      - run: echo "Deploying..."
```

---

## Step

### Step이란?

**Step은 Job 내에서 실행되는 개별 명령 또는 action**이다.

```yaml
steps:
  # GitHub 제공 action 사용
  - uses: actions/checkout@v4
  
  # 명령어 실행
  - run: npm install
  - run: npm test
  
  # 다중 라인 명령어
  - run: |
      npm install
      npm run build
      npm test
  
  # 조건부 실행
  - run: npm run deploy
    if: github.ref == 'refs/heads/main'
  
  # 환경 변수 사용
  - run: echo "Deploying to ${{ env.ENVIRONMENT }}"
    env:
      ENVIRONMENT: production
```

### Step 출력

```yaml
- name: Run tests
  id: test
  run: npm test

- name: Report results
  run: echo "Tests passed: ${{ steps.test.outcome }}"
```

---

## Runner

### "runner는 어디서 실행되는가?"

**Runner는 Workflow의 Step을 실행하는 머신**이다. GitHub에서 제공하는 호스팅 Runner와 자체 Runner를 사용할 수 있다.

```text
GitHub 호스팅 Runner (클라우드)
- ubuntu-latest
- windows-latest
- macos-latest
- 무료 사용 가능 (월 2000분)

자체 Runner (Self-hosted)
- 자신의 머신에 설치
- 무제한 사용
- 보안 제어 가능
```

### Runner 지정

```yaml
jobs:
  test:
    # GitHub 호스팅 (가장 일반적)
    runs-on: ubuntu-latest
    steps:
      - run: npm test
  
  deploy:
    # 자체 Runner (온프레미스)
    runs-on: [self-hosted, linux]
    steps:
      - run: kubectl deploy ...
```

### Self-hosted Runner 설정

```bash
# 1. Runner 다운로드
mkdir actions-runner
cd actions-runner
curl -o actions-runner-linux-x64-2.310.0.tar.gz \
  https://github.com/actions/runner/releases/download/v2.310.0/actions-runner-linux-x64-2.310.0.tar.gz
tar xzf actions-runner-linux-x64-2.310.0.tar.gz

# 2. Runner 등록
./config.sh --url https://github.com/owner/repo --token TOKEN

# 3. 서비스로 등록
./svc.sh install
./svc.sh start
```

---

## Artifact

### Artifact란?

**Artifact는 Job에서 생성되는 산출물(로그, 빌드 결과, 테스트 보고서 등)**이다.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: npm run build
      
      # Artifact 생성
      - uses: actions/upload-artifact@v3
        with:
          name: build-output
          path: dist/
          retention-days: 7

  test:
    runs-on: ubuntu-latest
    steps:
      - run: npm test -- --coverage
      
      # 테스트 보고서 저장
      - uses: actions/upload-artifact@v3
        if: always()
        with:
          name: test-coverage
          path: coverage/
```

### Artifact 다운로드

```yaml
jobs:
  deploy:
    needs: build
    steps:
      # 이전 job의 artifact 다운로드
      - uses: actions/download-artifact@v3
        with:
          name: build-output
          path: ./dist
      
      - run: |
          ls -la dist/
          npm run deploy
```

---

## Secret

### "secret은 왜 필요하고 어떻게 보호되는가?"

**Secret은 API 키, 데이터베이스 비밀번호 같은 민감한 정보를 안전하게 저장**한다.

```text
문제:
DB_PASSWORD="secret123" # workflow 파일에 저장 → GitHub에 노출!

해결:
github.secrets.DB_PASSWORD # 암호화되어 저장, 로그에 표시 안 됨
```

### Secret 설정

```bash
# GitHub UI에서 설정
Settings → Secrets and variables → Actions → New repository secret

변수명: DB_PASSWORD
값: supersecret123
```

### Secret 사용

```yaml
jobs:
  deploy:
    steps:
      - run: |
          export DB_PASSWORD=${{ secrets.DB_PASSWORD }}
          export API_KEY=${{ secrets.API_KEY }}
          npm run deploy
        env:
          DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
          API_KEY: ${{ secrets.API_KEY }}
```

### Secret 보호 규칙

```bash
# Secret이 로그에 노출되지 않도록 자동 마스킹
- run: echo "Password is ${{ secrets.DB_PASSWORD }}"
# 로그 출력: Password is ***

# 실수로 echo한 경우도 감지됨
- run: echo "supersecret123"
# 로그 출력: echo "***"
```

---

## CI와 CD

### "CI와 CD는 무엇이 다른가?"

**CI (Continuous Integration)는 코드 변경을 자동으로 테스트**하고, **CD (Continuous Deployment)는 테스트를 통과한 코드를 자동으로 배포**한다.

```text
CI (Continuous Integration)
- 개발자가 main에 push
- 자동으로 테스트 실행
- 테스트 실패 → 빨리 알림
- 목표: 코드 품질 보증

CD (Continuous Delivery)
- 테스트 통과 후 배포 준비 (수동 승인)
- Production에 배포 직전 상태
- 목표: 배포 가능한 상태 유지

CD (Continuous Deployment)
- 테스트 통과 후 자동 배포 (수동 승인 없음)
- 바로 사용자에게 전달
- 목표: 빠른 릴리스 주기
```

### CI 파이프라인

```yaml
name: CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm install
      - run: npm run lint
      - run: npm test
      - run: npm run type-check
```

### CD 파이프라인

```yaml
name: CI/CD

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm install && npm test

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: docker build -t app:${{ github.sha }} .
      - run: docker push registry.example.com/app:${{ github.sha }}

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: |
          kubectl set image deployment/app \
            app=registry.example.com/app:${{ github.sha }}
```

---

## 실전 Workflow 예시

```yaml
name: Full CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

env:
  REGISTRY: registry.example.com
  IMAGE_NAME: my-app

jobs:
  # 단계 1: 테스트
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run linter
        run: npm run lint
      
      - name: Run tests
        run: npm run test -- --coverage
      
      - name: Upload coverage
        uses: actions/upload-artifact@v3
        if: always()
        with:
          name: coverage
          path: coverage/

  # 단계 2: 빌드 (main에만)
  build:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      
      - name: Build Docker image
        run: |
          docker build -t ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }} .
      
      - name: Push to registry
        run: |
          echo ${{ secrets.REGISTRY_PASSWORD }} | docker login -u ${{ secrets.REGISTRY_USER }} --password-stdin ${{ env.REGISTRY }}
          docker push ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}

  # 단계 3: 배포 (main에만)
  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Deploy to production
        run: |
          mkdir -p ~/.kube
          echo "${{ secrets.KUBE_CONFIG }}" > ~/.kube/config
          kubectl set image deployment/my-app \
            my-app=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
      
      - name: Verify deployment
        run: |
          kubectl rollout status deployment/my-app
```

---

## 정리

| 개념 | 설명 |
|------|------|
| Workflow | 자동화 전체 프로세스 (YAML 파일) |
| Event | Workflow를 실행하는 트리거 |
| Job | Workflow 내 작업 단위 |
| Step | Job 내 개별 명령 |
| Runner | Workflow를 실행하는 머신 |
| Artifact | Job의 산출물 |
| Secret | 암호화된 민감한 정보 |
| CI | 자동 테스트 |
| CD | 자동 배포 |

---

## Related Notes

- [Git](../git/overview.md)
- [Deployment](../deployment/overview.md)
