# 환경 변수 (Environment Variables)

## 개념

**환경 변수**는 셸과 그 자식 프로세스들이 공유하는 **이름-값 쌍**이다. 프로그램이 시작될 때 필요한 설정을 환경 변수로 전달한다.

```bash
# 환경 변수 사용 예
echo $HOME     # /home/ubuntu
echo $PATH     # /usr/local/bin:/usr/bin:/bin:...
echo $USER     # ubuntu
```

---

## 주요 환경 변수

### 시스템 환경 변수

```bash
# 홈 디렉토리
echo $HOME     # /home/ubuntu

# 현재 사용자 이름
echo $USER     # ubuntu

# 호스트 이름
echo $HOSTNAME # my-server

# 셸의 종류
echo $SHELL    # /bin/bash

# 현재 작업 디렉토리
echo $PWD      # /home/ubuntu/app

# 이전 작업 디렉토리
echo $OLDPWD   # /home/ubuntu

# 시스템 언어
echo $LANG     # en_US.UTF-8
```

### PATH - 명령어 검색 경로

**PATH**는 `ls`, `python`, `node` 같은 명령어를 찾을 때 검색하는 디렉토리 목록이다.

```bash
echo $PATH
# /usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
```

콜론(`:`)으로 구분된 경로들을 순서대로 검색해서 명령어를 찾는다.

```bash
# python 명령어를 찾을 때
which python
# /usr/bin/python
```

### PATH에 경로 추가

```bash
# 현재 셸에서만 (임시)
export PATH=$PATH:/home/ubuntu/.local/bin

# 영구 설정 (홈 디렉토리에 .bashrc 편집)
echo 'export PATH=$PATH:/home/ubuntu/.local/bin' >> ~/.bashrc
source ~/.bashrc
```

---

## 환경 변수 설정하기

### 임시 설정 (현재 셸에서만)

```bash
# 설정
export MY_VAR="hello"

# 사용
echo $MY_VAR  # hello

# 셸 종료하면 사라짐
```

### 영구 설정 (재부팅해도 유지)

```bash
# 1. ~/.bashrc 파일 편집 (현재 사용자용)
vi ~/.bashrc

# 파일 끝에 추가
export APP_ENV="production"
export DATABASE_URL="postgres://localhost/mydb"
export API_KEY="sk-1234567890"

# 2. 저장하고 적용
source ~/.bashrc

# 3. 확인
echo $APP_ENV    # production
```

### 전역 설정 (모든 사용자)

```bash
# /etc/environment 파일 편집 (root만)
sudo vi /etc/environment

# 파일 내용
PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
LANG="en_US.UTF-8"
APP_ENV="production"
```

---

## .bashrc vs .bash_profile vs .profile

### .bashrc - 인터랙티브 셸 설정

```bash
# ~/.bashrc
# 대화형 셸이 시작될 때마다 실행됨

export PATH=$PATH:/home/ubuntu/.local/bin
export PS1="\u@\h:\w$ "  # 프롬프트 형태
alias ll='ls -lah'
alias grep='grep --color=auto'
```

**언제 실행**: 터미널을 열 때마다

### .bash_profile - 로그인 셸 설정

```bash
# ~/.bash_profile
# 로그인할 때만 실행됨 (SSH 접속 시)

# 보통 .bashrc를 실행
if [ -f ~/.bashrc ]; then
    source ~/.bashrc
fi
```

**언제 실행**: SSH로 접속할 때

### .profile - POSIX 호환 설정

```bash
# ~/.profile
# sh 및 호환 셸에서 실행

export PATH=$PATH:/home/ubuntu/.local/bin
```

**관계**

```
로그인 셸
  ↓
.bash_profile 또는 .profile 실행
  ↓
(보통 .bashrc도 함께 실행)
  ↓

인터랙티브 셸 (이미 로그인한 상태)
  ↓
.bashrc 실행
```

---

## 환경 변수 확인 및 관리

### 모든 환경 변수 확인

```bash
# 환경 변수만 확인
env

# 셸 변수 + 환경 변수 확인 (함수도 포함)
set

# 특정 변수 확인
echo $PATH
echo $HOME
```

### 변수 설정 해제

```bash
# 변수 삭제
unset MY_VAR

# 확인
echo $MY_VAR  # (아무것도 출력 안 됨)
```

### 읽기 전용 변수

```bash
# 한 번 설정하면 변경 불가
readonly DATABASE_URL="postgres://localhost"

# 변경 시도
export DATABASE_URL="postgres://other"
# bash: DATABASE_URL: readonly variable

# 읽기 전용 변수 확인
readonly -p
```

---

## 실제 사용 사례

### 배포 환경 설정

```bash
# ~/.bashrc
export APP_ENV="production"
export NODE_ENV="production"
export DATABASE_URL="postgres://user:pass@db.example.com/mydb"
export REDIS_URL="redis://localhost:6379"
export API_PORT="3000"
export LOG_LEVEL="info"
```

### 개발 환경 설정

```bash
# ~/.bashrc (개발 머신)
export APP_ENV="development"
export NODE_ENV="development"
export DATABASE_URL="postgres://localhost/mydb_dev"
export DEBUG="app:*"
```

### 프로젝트별 환경 변수

```bash
# ~/app/.env 파일 생성
DATABASE_URL="postgres://localhost/mydb"
API_KEY="sk-1234567890"
SECRET_KEY="secret-key-xyz"

# 스크립트에서 로드
#!/bin/bash
set -a
source .env
set +a

echo "Database: $DATABASE_URL"
```

### Python/Node.js 애플리케이션

```bash
# Node.js (환경 변수 사용)
#!/bin/bash
export NODE_ENV="production"
export PORT="3000"
export DATABASE_URL="postgres://localhost/mydb"

node app.js
```

```python
# Python (환경 변수 읽기)
import os

app_env = os.getenv('APP_ENV', 'development')
database_url = os.getenv('DATABASE_URL', 'sqlite:///db.sqlite')
debug = os.getenv('DEBUG', 'false').lower() == 'true'

print(f"환경: {app_env}")
print(f"DB: {database_url}")
```

---

## 환경 변수 Best Practices

### 1. 보안 정보는 환경 변수로

```bash
# ❌ 나쁜 예 (코드에 직접 쓰기)
const API_KEY = "sk-1234567890"

# ✅ 좋은 예 (환경 변수)
const API_KEY = process.env.API_KEY
```

### 2. .env 파일 .gitignore에 추가

```bash
# .gitignore
.env
.env.local
.env.*.local
```

```bash
# .env 파일 예
DATABASE_URL="..."
API_KEY="..."
SECRET_KEY="..."
```

### 3. 환경별로 다른 설정

```bash
# .env.development
DATABASE_URL="postgres://localhost/mydb_dev"
DEBUG="true"

# .env.production
DATABASE_URL="postgres://db.example.com/mydb"
DEBUG="false"

# 로드 스크립트
#!/bin/bash
ENV=${APP_ENV:-development}
set -a
source .env.$ENV
set +a
```

### 4. 환경 변수 필수 확인

```bash
#!/bin/bash

# 필수 환경 변수 확인
required_vars=("DATABASE_URL" "API_KEY" "SECRET_KEY")

for var in "${required_vars[@]}"; do
    if [ -z "${!var}" ]; then
        echo "Error: $var is not set"
        exit 1
    fi
done

echo "모든 필수 환경 변수가 설정됨"
```

---

## 프로세스에 전달하기

### 명령어 실행 시 환경 변수 설정

```bash
# 한 번만 적용
DATABASE_URL="postgres://localhost" python app.py

# 여러 개 설정
NODE_ENV="production" PORT="8080" DEBUG="true" node server.js
```

### 서비스 시작 시 환경 변수

```bash
# systemd 서비스 파일 (/etc/systemd/system/myapp.service)
[Service]
Type=simple
User=ubuntu
WorkingDirectory=/home/ubuntu/app
ExecStart=/usr/bin/node server.js
Environment="NODE_ENV=production"
Environment="PORT=3000"
EnvironmentFile=/home/ubuntu/app/.env

Restart=always
RestartSec=10
```

---

## 참고

- 환경 변수는 프로세스 격리 (각 프로세스가 독립적인 복사본 유지)
- `export`하지 않으면 자식 프로세스가 상속받지 못함
- 민감한 정보(API 키, 비밀번호)는 절대 코드에 하드코딩하지 말 것
- 배포 시 환경 변수 설정을 자동화하는 것이 중요함

## Related Notes

- [Shell Scripting](shell-scripting.md)
- [Linux System Operations](linux-system-operations.md)
