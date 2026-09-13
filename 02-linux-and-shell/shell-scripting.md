# 쉘 스크립팅 (Shell Scripting)

## 개요

쉘 스크립트는 **쉘 명령어를 파일에 저장해서 자동으로 실행하는 방식**이다. 배포, 백업, 모니터링 등 반복되는 작업을 자동화할 때 가장 많이 사용된다.

```bash
# 수동으로 매번 입력하는 방식
mkdir -p /backup
cp -r /home/ubuntu/app /backup/app-$(date +%Y%m%d)
tar -czf /backup/backup.tar.gz /backup/app

# 스크립트로 자동화 (backup.sh)
#!/bin/bash
mkdir -p /backup
cp -r /home/ubuntu/app /backup/app-$(date +%Y%m%d)
tar -czf /backup/backup.tar.gz /backup/app
```

---

## 쉘 스크립트 작성 기초

### 파일 생성 및 권한

```bash
# 1. 스크립트 파일 생성
cat > backup.sh << 'EOF'
#!/bin/bash
echo "백업 시작..."
EOF

# 2. 실행 권한 추가
chmod +x backup.sh

# 3. 실행
./backup.sh
```

### Shebang (#!)

```bash
#!/bin/bash
```

파일의 첫 줄인 **shebang**은 "이 파일을 어떤 인터프리터로 실행할지"를 지정한다.

```bash
#!/bin/bash       # Bash로 실행
#!/bin/sh         # 표준 sh로 실행
#!/usr/bin/python # Python으로 실행
```

---

## 변수 (Variables)

### 변수 선언 및 사용

```bash
#!/bin/bash

# 변수 선언 (공백 없음!)
NAME="John"
AGE=30
PATH_TO_APP="/home/ubuntu/app"

# 변수 사용 ($로 참조)
echo "이름: $NAME"
echo "나이: $AGE"
echo "경로: $PATH_TO_APP"
```

**주의**: 공백이 있으면 오류 발생

```bash
# 잘못된 방법
NAME = "John"  # 오류!

# 올바른 방법
NAME="John"    # OK
```

### 명령어 결과를 변수에 저장

```bash
#!/bin/bash

# 방법 1: $(...)
CURRENT_DATE=$(date +%Y%m%d)
echo "오늘 날짜: $CURRENT_DATE"

# 방법 2: `...` (이전 방식, 가독성 낮음)
CURRENT_DATE=`date +%Y%m%d`
```

---

## 조건문 (if / else)

### 기본 구조

```bash
#!/bin/bash

AGE=25

if [ $AGE -ge 18 ]; then
    echo "성인입니다"
else
    echo "미성년자입니다"
fi
```

### 비교 연산자

```bash
# 숫자 비교
[ $A -eq $B ]   # 같음 (equal)
[ $A -ne $B ]   # 다름 (not equal)
[ $A -lt $B ]   # 작음 (less than)
[ $A -le $B ]   # 작거나 같음
[ $A -gt $B ]   # 큼 (greater than)
[ $A -ge $B ]   # 크거나 같음

# 문자열 비교
[ "$A" = "$B" ]   # 같음
[ "$A" != "$B" ]  # 다름
[ -z "$A" ]       # 빈 문자열인가?
[ -n "$A" ]       # 비어있지 않은가?

# 파일 확인
[ -f $FILE ]      # 파일이 존재하는가?
[ -d $DIR ]       # 디렉토리가 존재하는가?
[ -e $PATH ]      # 경로가 존재하는가?
[ -r $FILE ]      # 읽기 권한이 있는가?
[ -w $FILE ]      # 쓰기 권한이 있는가?
[ -x $FILE ]      # 실행 권한이 있는가?
```

### 실제 예시

```bash
#!/bin/bash

FILE="/home/ubuntu/app/config.sh"

if [ -f $FILE ]; then
    echo "파일이 존재합니다"
    source $FILE  # 파일 실행
else
    echo "파일이 없습니다"
    exit 1        # 오류로 종료
fi
```

### if / elif / else

```bash
#!/bin/bash

HOUR=$(date +%H)

if [ $HOUR -lt 12 ]; then
    echo "좋은 아침입니다"
elif [ $HOUR -lt 18 ]; then
    echo "좋은 오후입니다"
else
    echo "좋은 밤입니다"
fi
```

---

## 반복문 (Loop)

### for 루프

```bash
#!/bin/bash

# 1. 범위 지정
for i in 1 2 3 4 5; do
    echo "번호: $i"
done

# 2. 시퀀스 생성
for i in {1..5}; do
    echo "번호: $i"
done

# 3. 배열 순회
FILES=("file1.txt" "file2.txt" "file3.txt")
for FILE in "${FILES[@]}"; do
    echo "처리 중: $FILE"
done

# 4. 명령어 결과 순회
for FILE in $(ls *.txt); do
    echo "파일: $FILE"
done
```

### while 루프

```bash
#!/bin/bash

COUNT=1
while [ $COUNT -le 5 ]; do
    echo "카운트: $COUNT"
    COUNT=$((COUNT + 1))  # 수식 계산
done
```

### 반복 제어

```bash
#!/bin/bash

for i in {1..10}; do
    if [ $i -eq 5 ]; then
        continue  # 다음 반복으로 (5는 건너뜀)
    fi
    
    if [ $i -eq 8 ]; then
        break     # 루프 탈출
    fi
    
    echo "번호: $i"
done
```

---

## 함수 (Functions)

### 함수 정의 및 호출

```bash
#!/bin/bash

# 함수 정의
backup_files() {
    echo "백업 중..."
    cp -r /home/ubuntu/app /backup/
    echo "백업 완료"
}

# 함수 호출
backup_files
backup_files
```

### 매개변수와 반환값

```bash
#!/bin/bash

# 매개변수 받기
greet() {
    local NAME=$1      # 첫 번째 매개변수
    local AGE=$2       # 두 번째 매개변수
    
    echo "안녕하세요, $NAME님! 나이: $AGE"
    
    return 0  # 반환값 (0 = 성공, 1 = 실패)
}

# 함수 호출
greet "John" 30
greet "Jane" 25
```

### 함수의 반환값 확인

```bash
#!/bin/bash

copy_file() {
    if cp $1 $2; then
        return 0  # 성공
    else
        return 1  # 실패
    fi
}

if copy_file "source.txt" "dest.txt"; then
    echo "복사 성공"
else
    echo "복사 실패"
fi
```

---

## 실제 예시: 배포 스크립트

```bash
#!/bin/bash

# 에러 발생 시 즉시 종료
set -e

# 변수 선언
APP_DIR="/home/ubuntu/app"
BACKUP_DIR="/backup"
DATE=$(date +%Y%m%d_%H%M%S)

# 함수: 백업
backup() {
    echo "백업 중... ($DATE)"
    cp -r $APP_DIR $BACKUP_DIR/app-$DATE
    echo "백업 완료"
}

# 함수: 배포
deploy() {
    echo "배포 중..."
    cd $APP_DIR
    git pull origin main
    npm install
    npm run build
    systemctl restart app
    echo "배포 완료"
}

# 함수: 검증
validate() {
    if curl -f http://localhost:3000 > /dev/null; then
        echo "서비스 정상 작동"
        return 0
    else
        echo "서비스 오류"
        return 1
    fi
}

# 메인 로직
echo "배포 시작: $DATE"
backup
deploy

if validate; then
    echo "배포 성공"
else
    echo "검증 실패, 이전 버전으로 복구"
    cp -r $BACKUP_DIR/app-$DATE $APP_DIR
    systemctl restart app
    exit 1
fi
```

---

## 특수 변수

```bash
#!/bin/bash

echo $0      # 스크립트 이름
echo $1      # 첫 번째 인자
echo $2      # 두 번째 인자
echo $@      # 모든 인자 (배열)
echo $#      # 인자 개수
echo $?      # 이전 명령어의 반환값 (0 = 성공)
echo $$      # 현재 프로세스 ID
```

### 실제 사용

```bash
#!/bin/bash

# 스크립트 실행: ./script.sh apple banana cherry

echo "스크립트: $0"
echo "첫 번째: $1"  # apple
echo "두 번째: $2"  # banana
echo "세 번째: $3"  # cherry
echo "모두: $@"     # apple banana cherry
echo "개수: $#"     # 3
```

---

## 주의사항

### set -e (에러 발생 시 즉시 종료)

```bash
#!/bin/bash
set -e

cd /nonexistent  # 오류 발생
echo "이 줄은 실행되지 않음"
```

### set -x (디버그 모드)

```bash
#!/bin/bash
set -x  # 실행할 명령어 출력

mkdir -p /backup
cp -r /app /backup/
```

```
+ mkdir -p /backup
+ cp -r /app /backup/
```

---

## 참고

- 스크립트는 간단한 작업 자동화부터 복잡한 배포 파이프라인까지 다양하게 사용된다.
- 복잡한 로직은 Python이나 Go 같은 언어가 더 적합할 수 있다.
- 본 노트의 예시는 `#!/bin/bash` 기준이다.

## Related Notes

- [Linux Shell I/O and Command Flow](linux-shell-io-and-command-flow.md)
- [Linux System Operations](linux-system-operations.md)
