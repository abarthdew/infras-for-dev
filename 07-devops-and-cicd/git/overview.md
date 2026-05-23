# Git

## 목차
- [Git의 역할](#git의-역할)
- [Repository](#repository)
- [Working Tree와 Index](#working-tree와-index)
- [Commit](#commit)
- [Branch](#branch)
- [Merge와 Rebase](#merge와-rebase)
- [Remote](#remote)
- [Conflict](#conflict)

---

## Git의 역할

Git은 **파일 변경 이력을 commit 단위로 저장하는 분산 버전 관리 시스템(VCS)**이다. 각 개발자가 전체 저장소 복사본을 가지고 있어서, 중앙 서버 없이도 협업이 가능하다.

```text
로컬 저장소 (완전한 히스토리)
    ↓
원격 저장소 (GitHub, GitLab 등)
    ↓
다른 개발자의 로컬 저장소
```

### Git이 필요한 이유

```text
Git 없이:
- 파일 버전 관리 불가 (file_v1.py, file_v2.py...)
- 누가 언제 뭘 바꿨는지 추적 불가
- 협업이 어려움 (파일 덮어쓰기)

Git 사용:
- 모든 변경 이력 자동 기록
- 누가 언제 뭘 바꿨는지 명확
- 여러 사람이 동시 작업 가능
- 이전 버전으로 롤백 가능
```

---

## Repository

### Repository란?

**Repository는 Git이 모든 변경 이력과 메타데이터를 저장하는 디렉토리**다.

```text
my-project/
├─ .git/              ← Git 메타데이터 (숨겨진 디렉토리)
│  ├─ objects/        ← 모든 commit, tree, blob 저장
│  ├─ refs/           ← branch 포인터
│  ├─ HEAD            ← 현재 branch 가리킴
│  └─ config          ← 저장소 설정
├─ src/
├─ README.md
└─ ...
```

### 로컬 vs 원격 Repository

```text
로컬 Repository (.git이 있는 곳)
- 로컬 머신에만 존재
- 모든 commit 히스토리 포함
- git log, git diff 등은 로컬에서 실행

원격 Repository (GitHub, GitLab)
- 서버에 호스팅됨
- 팀이 공유하는 중앙 저장소
- git push/pull로 동기화
```

### Repository 생성

```bash
# 새 저장소 초기화
git init my-project

# 기존 저장소 복제
git clone https://github.com/user/repo.git

# 저장소 설정 확인
git config --local --list  # 로컬 설정
git config --global --list # 전역 설정
```

---

## Working Tree와 Index

### "working tree와 index는 무엇이 다른가?"

**Working Tree, Index(Staging Area), Repository는 Git의 3가지 상태 영역**이다.

```text
┌─ Working Tree (작업 디렉토리)
│  └─ 실제 파일 (수정됨)
│
├─ Index / Staging Area
│  └─ commit할 파일 선택
│
└─ Repository (.git)
   └─ 영구 저장된 commit
```

### 각 영역의 역할

```text
Working Tree
- 개발자가 실제로 파일을 수정하는 곳
- git 추적 대상이 아닌 파일도 존재 가능
- 수정 내용이 즉시 반영됨

Index (Staging Area)
- "다음 commit에 포함할 파일" 선택하는 곳
- git add로 여기에 파일 추가
- 여러 번 add로 원하는 파일만 선택 가능

Repository
- 모든 commit 히스토리 저장
- 불변 (변경 불가)
- 각 commit은 고유한 hash로 식별
```

### 상태 전이

```bash
# 1. Working Tree에서 파일 수정
vi README.md

# 2. 현재 상태 확인
git status
# modified:   README.md (Working Tree에만 변경)

# 3. Index에 추가
git add README.md

# 4. 상태 다시 확인
git status
# Changes to be committed:
#   modified:   README.md (Index에 준비됨)

# 5. Repository에 commit
git commit -m "Update README"

# 6. 상태 최종 확인
git status
# nothing to commit, working tree clean
```

### 부분 커밋 (일부 파일만 선택)

```bash
# 여러 파일 수정
vi file1.py
vi file2.py
vi file3.py

# 특정 파일만 commit
git add file1.py file2.py
git commit -m "Fix bug in file1 and file2"

# file3.py는 Working Tree에만 남음
git status
# modified:   file3.py (untracked)
```

---

## Commit

### Commit이란?

**Commit은 특정 시점의 프로젝트 전체 상태를 스냅샷으로 저장**한다.

```text
Commit 구성:
├─ Hash (SHA-1): 고유 식별자
├─ Author: 작성자
├─ Date: 작성 시간
├─ Message: 커밋 메시지
├─ Parent: 이전 커밋 가리킴
└─ Tree: 전체 파일 구조 스냅샷
```

### "commit hash는 무엇을 식별하는가?"

**Commit hash는 그 commit의 모든 내용(파일, 메시지, 작성자, 시간 등)을 고유하게 식별**한다.

```text
commit 3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d
Author: John Doe <john@example.com>
Date:   Mon May 23 10:15:30 2026 +0900

    Fix bug in login flow

- Hash 변경 = 완전히 다른 commit (변경 불가능)
- 같은 파일 내용이라도 시간이 다르면 hash가 다름
```

### Commit 생성

```bash
# 기본 commit
git commit -m "Brief description"

# 상세 메시지
git commit -m "Brief description" -m "Detailed explanation
- bullet point 1
- bullet point 2"

# 모든 수정된 파일을 자동 추가 (새 파일 제외)
git commit -am "Update multiple files"

# 이전 commit 수정
git commit --amend

# 빈 commit (테스트 목적)
git commit --allow-empty -m "Trigger CI/CD"
```

### Commit 조회

```bash
# 전체 히스토리
git log

# 한 줄씩 간결하게
git log --oneline

# 그래프로 시각화 (branch 보임)
git log --oneline --graph --all

# 특정 파일의 히스토리
git log -- file.py

# 특정 범위
git log main..feature  # feature에만 있는 commit

# 특정 내용 검색
git log -S "bug fix"   # 이 텍스트를 포함하는 commit
```

---

## Branch

### Branch란?

**Branch는 독립적인 개발 흐름을 만드는 포인터**다. 같은 저장소에서 여러 버전을 동시에 개발할 수 있다.

```text
       feature1         feature2
         ↑               ↑
c1 ← c2 ← c3          c4
     ↑
    main (기본 branch)
```

### Branch의 용도

```text
main (master)
- 프로덕션에 배포되는 코드
- 항상 안정적이어야 함

develop
- 다음 버전 개발
- feature branch들의 기반

feature/...
- 새로운 기능 개발
- develop에서 생성, 완료 후 develop에 merge

hotfix/...
- 긴급 버그 수정
- main에서 생성, 수정 후 main과 develop에 merge
```

### Branch 관리

```bash
# 현재 branch 확인
git branch

# 새 branch 생성
git branch feature/new-feature

# branch 전환
git checkout feature/new-feature
# 또는 (최신 Git)
git switch feature/new-feature

# 생성과 전환을 한 번에
git checkout -b feature/new-feature
git switch -c feature/new-feature

# branch 삭제
git branch -d feature/new-feature  # 안전 삭제 (merge 확인)
git branch -D feature/new-feature  # 강제 삭제

# branch 이름 변경
git branch -m old-name new-name

# 모든 branch 보기 (원격 포함)
git branch -a
```

### Branch 전환

```bash
# feature branch로 전환
git switch feature/login

# 이제 이 branch에서 작업
vi login.py
git add login.py
git commit -m "Implement login form"

# main으로 돌아가기
git switch main

# main의 파일들로 작업 디렉토리 변경됨 (feature의 변경사항 보이지 않음)
```

---

## Merge와 Rebase

### "merge와 rebase는 무엇이 다른가?"

**Merge는 두 branch의 변경사항을 모두 유지하며 합치고, Rebase는 한 branch의 commit을 다른 branch 위에 순차적으로 재배치**한다.

### Merge

```text
feature branch:
c1 - c2 - c3

main branch:
m1 - m2

Merge 후:
       c3
       /  \
m1 - m2    merge commit
       \  /
       (feature branch의 모든 변경사항 유지)
```

```bash
# feature branch의 변경사항을 main에 merge
git switch main
git merge feature/login

# merge commit이 생성됨
# feature의 모든 변경사항이 main에 포함
```

### Rebase

```text
feature branch:
c1 - c2 - c3

main branch:
m1 - m2

Rebase 후:
m1 - m2 - c1' - c2' - c3'
(feature의 commit들이 main 위에 순차적으로 배치)
```

```bash
# feature branch의 commit들을 main 위에 배치
git switch feature/login
git rebase main

# 이제 feature는 main의 최신 위에 있음
# commit 히스토리가 선형적 (일직선)

# main으로 돌아가서 fast-forward merge
git switch main
git merge feature/login
```

### Merge vs Rebase 비교

| | Merge | Rebase |
|---|---|---|
| Commit 히스토리 | 병렬 구조 (보존) | 선형 구조 (재정렬) |
| Commit 수 | merge commit 추가 | 기존 commit 수 유지 |
| 장점 | 히스토리 보존, 안전 | 히스토리 깔끔, 읽기 쉬움 |
| 단점 | 히스토리 복잡 | 기존 commit hash 변경 |
| 사용처 | 공개 branch | 로컬 branch |

### 실전 가이드

```bash
# 로컬에서만 사용할 branch는 rebase (깔끔한 히스토리)
git switch feature/login
git rebase main
git switch main
git merge feature/login

# 공개 branch는 merge (안전성)
git switch main
git merge develop  # develop의 모든 변경사항 유지
```

---

## Remote

### Remote란?

**Remote는 원격 저장소에 대한 별칭**이다. 보통 `origin`이라 부른다.

```bash
# remote 확인
git remote -v
# origin  https://github.com/user/repo.git (fetch)
# origin  https://github.com/user/repo.git (push)

# remote 추가
git remote add origin https://github.com/user/repo.git

# remote 제거
git remote remove origin

# remote 이름 변경
git remote rename origin upstream
```

### Push

```bash
# 로컬 commit을 원격에 보내기
git push

# 특정 branch 지정
git push origin main

# 첫 push일 때 branch 생성
git push -u origin feature/login

# 강제 push (주의! 팀 협업 시 위험)
git push --force
```

### Pull

```bash
# 원격의 변경사항을 가져와서 merge
git pull

# 실제로는 fetch + merge
git fetch origin
git merge origin/main

# rebase로 merge (깔끔한 히스토리)
git pull --rebase
```

### Fetch vs Pull

```text
Fetch
- 원격의 변경사항을 로컬에 다운로드
- 로컬 branch 변경 안 함
- 안전 (손상 위험 없음)

Pull
- Fetch + Merge
- 로컬 branch 자동 업데이트
- 충돌 가능성 있음
```

### Tracking Branch

```bash
# feature/login을 원격과 동기화하도록 설정
git branch -u origin/feature/login

# 또는 push할 때
git push -u origin feature/login

# 확인
git branch -vv
# feature/login  abc1234 [origin/feature/login] Commit message
```

---

## Conflict

### "conflict는 왜 발생하는가?"

**Conflict는 같은 파일의 같은 부분을 두 branch에서 다르게 수정했을 때 발생**한다.

```text
main branch:
def login():
    return "success"  ← 메인에서 수정

feature branch:
def login():
    return "login success"  ← feature에서 수정

Merge 시도 → Conflict!
Git은 어느 것이 맞는지 알 수 없음
```

### Conflict 발생 상황

```bash
# main에서 작업
git checkout main
vi app.py  # 수정
git commit -m "Update app.py"

# feature에서 같은 파일 수정
git checkout feature/login
vi app.py  # 다른 부분 수정
git commit -m "Update app.py"

# merge 시도
git checkout main
git merge feature/login
# CONFLICT (content conflict): Merge conflict in app.py
```

### Conflict 해결

```bash
# 충돌 파일 확인
git status
# both modified:   app.py

# 파일 내용
cat app.py
# <<<<<<< HEAD
# def login():
#     return "success"
# =======
# def login():
#     return "login success"
# >>>>>>> feature/login

# 수동으로 수정
vi app.py
# 원하는 부분만 유지, 충돌 마커 제거

# 수정 후 완료
git add app.py
git commit -m "Resolve conflict in app.py"
```

### Conflict 예방

```bash
# 1. 자주 동기화
git pull --rebase origin main  # 주기적으로 최신 유지

# 2. 더 작은 단위의 commit
# 작은 변경 → 충돌 가능성 ↓

# 3. 팀 규칙 정하기
# 같은 파일을 동시에 수정하지 않기

# 4. merge tool 사용
git mergetool  # IDE 통합 툴로 해결
```

### Merge 취소

```bash
# merge 중단 (conflict 발생 시)
git merge --abort

# merge 후 취소
git revert -m 1 <merge-commit-hash>
```

---

## 실전 워크플로우

```bash
# 1. main에서 feature branch 생성
git switch main
git pull origin main
git switch -c feature/user-auth

# 2. 작업 진행
vi auth.py
git add auth.py
git commit -m "Implement user authentication"

vi test_auth.py
git add test_auth.py
git commit -m "Add tests for auth"

# 3. 원격에 push
git push -u origin feature/user-auth

# 4. Pull Request 생성 (GitHub)
# → Code Review 후 Approval

# 5. main에 merge
git switch main
git pull origin main
git merge feature/user-auth

# 6. 원격에 push
git push origin main

# 7. feature branch 정리
git branch -d feature/user-auth
git push origin -d feature/user-auth
```

---

## 정리

| 개념 | 설명 |
|------|------|
| Working Tree | 실제 작업 디렉토리 |
| Index | Commit할 파일 선택 영역 |
| Repository | 모든 commit 히스토리 저장소 |
| Commit | 특정 시점의 스냅샷 |
| Branch | 독립적인 개발 흐름 |
| Merge | 두 branch를 병렬 구조로 합침 |
| Rebase | 한 branch를 다른 위에 순차 배치 |
| Remote | 원격 저장소 참조 |
| Conflict | 같은 부분을 다르게 수정했을 때 |

---

## Related Notes

- [GitHub Actions](../github-actions/overview.md)
