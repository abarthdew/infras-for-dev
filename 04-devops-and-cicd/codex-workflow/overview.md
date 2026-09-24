# Codex Workflow

## 목차
- [Codex Workflow의 역할](#codex-workflow의-역할)
- [학습 질문 누적](#학습-질문-누적)
- [파일 분류](#파일-분류)
- [Commit과 Push](#commit과-push)
- [Agent Identity](#agent-identity)
- [병렬 작업 주의](#병렬-작업-주의)
- [실전 워크플로우](#실전-워크플로우)

---

## Codex Workflow의 역할

**Codex Workflow는 사용자의 학습 질문을 받아 자동으로 관련 문서를 찾거나 생성하고, 구조화된 답변으로 누적하는 패턴**이다. 사용자는 질문만 하고, AI Agent가 문서화, 분류, 커밋을 자동 처리한다.

```text
사용자 질문
    ↓
AI Agent
    ├─ 관련 문서 찾기
    ├─ 답변 작성/수정
    ├─ 파일 분류
    └─ Git commit/push
    ↓
문서 축적 (자동)
```

### 수동 방식 vs Codex 방식

```text
수동:
사용자 → 직접 파일 생성 → 내용 작성 → git 커밋 → 푸시
(시간 소모, 일관성 부족)

Codex:
사용자 → 질문만 제시 → Agent가 자동 처리
(빠름, 일관성 높음, 구조화됨)
```

---

## 학습 질문 누적

### 질문이 오면

```text
사용자: "Linux에서 파일 권한은 어떻게 관리하는가?"

Agent 작업:
1. 질문 분류: 02-linux-and-shell 범주
2. 관련 파일 검색: permissions.md
3. 파일 구조 분석: 어디에 추가할지 결정
4. 답변 작성: 상세한 설명 (예제 포함)
5. 파일 업데이트: 구조 유지하면서 내용 추가
6. git commit: 변경사항 기록
7. git push: 원격 저장소 동기화
```

### 질문 수집 기준

```text
개념적 질문 (문서화 가치 높음)
- "Linux에서 package manager는 어떻게 동작하는가?"
- "Docker와 VM은 무엇이 다른가?"
- "Kubernetes의 Pod는 Container와 어떻게 다른가?"
→ 즉시 문서화

구현 관련 질문 (코드 예제 필요)
- "Python에서 async/await는 어떻게 사용하는가?"
- "JavaScript의 closure는 무엇인가?"
→ 코드 예제와 함께 문서화

순간적 이슈 (메모 정도)
- "이 에러는 어떻게 해결하는가?"
- "이 명령어 플래그는?"
→ 관련 파일에 간단히 추가
```

---

## 파일 분류

### "언제 새 파일을 만들고 언제 기존 파일에 append할 것인가?"

**새로운 큰 주제는 새 파일, 기존 주제의 세부 내용은 append**한다.

```text
새 파일을 만드는 경우:
- 완전히 새로운 주제 (예: Prometheus 모니터링)
- 디렉토리 내 새 하위 주제 (예: database/redis)
- 기존 파일보다 큼직한 범위

기존 파일에 append:
- 이미 관련 섹션이 있음
- 같은 주제의 세부 내용
- 파일이 일관된 맥락 유지
```

### 파일 구조 예시

```
02-linux-and-shell/
├─ overview.md (개요)
├─ shell-scripting.md (쉘 스크립팅)
├─ file-system.md (파일시스템)
├─ permissions.md (권한 관리)
├─ environment-variables.md (환경변수)
└─ ...
```

```text
질문: "파일 permission에서 setuid는 뭐야?"
→ permissions.md 파일의 Setuid 섹션에 추가

질문: "Kubernetes operator pattern은?"
→ 03-infrastructure/kubernetes/overview.md에 operator 섹션 추가
```

### 파일 정확도

```text
파일명은 정확해야 함:
✓ shell-scripting.md (구체적)
✗ shell.md (모호함)

✓ data-modeling.md (명확함)
✗ db.md (축약어)

구조 유지:
- 각 파일은 독립적으로 읽혀야 함
- 목차(TOC)는 앵커 링크 포함
- 관련 파일은 문서 하단에 링크
```

---

## Commit과 Push

### "push 전 pull/rebase는 왜 필요한가?"

**Pull/rebase는 원격의 최신 변경사항을 로컬에 반영하고 충돌을 미리 해결**한다.

```text
원격 저장소:        로컬 저장소:
c1 ← c2 (main)     c1 ← c3 (내 변경)

⬇️ Pull 없이 push 시도
REJECTED: 먼저 pull!

⬇️ Pull/Rebase:
c1 ← c2 ← c3' (rebase 후)

⬇️ 이제 push 가능
✓ c1 ← c2 ← c3'
```

### Commit 메시지 규칙

```bash
# 좋은 예시:
git commit -m "Add explanation of Linux file permissions with chmod and chown"
git commit -m "Expand Docker overview with volume persistence examples"

# 나쁜 예시:
git commit -m "Update"
git commit -m "Fix"
git commit -m "Changes"

# 포맷:
- 동작 (Add, Expand, Fix, Update, Refactor)
- 대상 (파일명 또는 주제)
- 상세 설명 (무엇을 했는지)
```

### Push 전 체크리스트

```bash
# 1. 상태 확인
git status  # 아무것도 staging되지 않았으면 OK

# 2. 최신 변경 받기
git pull --rebase origin main

# 3. 충돌 해결 (필요시)
# 충돌 파일 수정 후
git add .
git rebase --continue

# 4. 로그 확인
git log --oneline -5

# 5. 푸시
git push origin main
```

---

## Agent Identity

### "agent identity는 왜 사용자 identity와 분리하는가?"

**Agent와 사용자의 commit을 분리하면 누가 어떤 변경을 했는지 명확하게 추적**할 수 있다.

```text
문제 (분리 안 함):
모든 commit이 user 이름으로 기록
→ 사용자 직접 변경과 AI 생성 변경이 구분 안 됨
→ 누가 코드를 작성했는지 불명확

해결 (분리함):
사용자: John Doe
AI Agent: Claude Sonnet

commit log:
- John Doe: 새 파일 생성
- Claude: 구조 개선
- John Doe: 기존 내용 수정
→ 변경 이력이 명확함
```

### Git Config 설정

```bash
# 기존 user identity (사용자)
git config --global user.name "AbarthDew"
git config --global user.email "user@example.com"

# Agent commit 시 임시 오버라이드
git -c user.name="Claude" \
    -c user.email="claude@anthropic.com" \
    commit -m "Expand documentation"

# 또는 스크립트에서
export GIT_AUTHOR_NAME="Claude"
export GIT_AUTHOR_EMAIL="claude@anthropic.com"
git commit -m "..."
```

### Commit 메타데이터

```bash
# 상세 로그로 작성자 확인
git log --format='%h %an <%ae> %s'

# 출력 예시:
3a4b5c6 Claude <claude@anthropic.com> Add Docker explanation
f7e8d9c AbarthDew <user@example.com> Add new learning goal
```

---

## 병렬 작업 주의

### 동시 작업 시 문제

```text
시나리오:
사용자가 로컬에서 파일 수정 중
    ↓
동시에 AI Agent가 원격에서 같은 파일 수정
    ↓
user가 push 시도
    ↓
Rejected: 충돌!
```

### 해결 방법

```bash
# 방법 1: Pull before push
# 사용자가 작업 전에
git pull origin main

# 방법 2: Rebase
# 작업 완료 후
git pull --rebase origin main
git push origin main

# 방법 3: Stash
# 충돌 시 임시 저장
git stash
git pull
git stash pop
```

### 문제 피하기

```text
좋은 실무:
1. 자주 pull (매 작업 시작 전)
2. 작은 단위 commit
3. 작은 범위 파일 수정
4. 정기적 push (하루 1-2회)

나쁜 실무:
1. 여러 날간 pull 안 함
2. 큰 파일에 많은 변경
3. 여러 주제 한 번에 수정
4. 주말에 한꺼번에 push
```

---

## 실전 워크플로우

### 하루의 학습 사이클

```bash
# 1. 시작 (매일 아침)
git pull origin main  # 최신 변경사항 받기

# 2. 학습 및 질문
# 사용자: "Kubernetes StatefulSet은 뭐야?"
# → Agent가 자동으로 kubernetes/overview.md에 추가

# 3. Agent의 자동 처리
# - 파일 찾기
# - 섹션 추가
# - 예제 작성
# - commit: "Expand Kubernetes with StatefulSet explanation"
# - push

# 4. 사용자가 로컬에서 수정
# git pull --rebase origin main
# dbms-for-dev 저장소에서:
# vi 03-databases/redis/overview.md  # 수정
# git add 03-databases/redis/overview.md
# git commit -m "Add Redis pub/sub pattern explanation"
# git push origin main

# 5. 계속 학습 또는 종료
# 더 배울 게 있으면 반복
```

### 주간 정리

```bash
# 주 1회: 전체 상태 확인
git log --oneline --all -20  # 최근 변경 20개

git log --format='%an' --all | sort | uniq -c  # 작성자별 통계

# 필요시 파일 구조 정리
# - 너무 큰 파일 분할
# - 고아 파일 정리
# - 목차 갱신
```

### 학습 속도 조절

```text
빠른 학습 (매일 여러 질문):
- 자동화 덕에 가능
- Agent가 문서화 처리
- 사용자는 질문만 집중

깊은 학습 (주 1-2 질문):
- 각 주제를 깊이 있게
- 사용자가 직접 수정 가능
- 더 많은 예제와 설명

혼합형:
- 빠른 질문: Agent 자동 처리
- 깊은 수정: 사용자가 직접
```

---

## "학습 노트는 어느 정도 깊이로 작성할 것인가?"

### 깊이 기준

```text
Level 1 (기본 이해)
- 개념 정의
- 간단한 예제 1-2개
- 비유나 그림

Level 2 (실무 활용)
- 동작 원리 상세 설명
- 실제 코드 예제
- 최적 사용 사례

Level 3 (깊이 있는 이해)
- 내부 구현 원리
- 성능 특성
- 트레이드오프 분석
- 심화 예제
```

### 예시: "Docker는 뭐야?"

```text
Level 1:
- Docker는 컨테이너 기술
- VM보다 가볍고 빠름
- 이미지 → 컨테이너

Level 2:
- Dockerfile 문법
- 이미지 빌드 과정
- Port mapping, Volume
- Docker Compose

Level 3:
- Cgroup, Namespace 내부 구조
- 레이어 기반 이미지 원리
- 성능 오버헤드 분석
- Kubernetes와의 통합
```

### 파일별 깊이 가이드

```text
overview.md: Level 1-2
- 기본 개념과 실무 활용

심화 파일 (별도 생성): Level 2-3
- 예: dbms-for-dev/03-databases/redis/advanced-data-structures.md
- 예: 03-infrastructure/kubernetes/advanced-scheduling.md
```

---

## 정리

| 단계 | 내용 | 담당 |
|------|------|------|
| 1. 질문 | 사용자가 학습 질문 제시 | 사용자 |
| 2. 분류 | 관련 파일 찾기 | Agent |
| 3. 작성 | 답변 작성 및 구조화 | Agent |
| 4. 커밋 | Git commit (별도 identity) | Agent |
| 5. 푸시 | 원격 저장소 동기화 | Agent |
| 6. 검증 | 사용자가 결과 확인 및 수정 | 사용자 |

---

## Related Notes

- [Git](../git/overview.md)
- [GitHub Actions](../github-actions/overview.md)
- [Deployment](../deployment/overview.md)
