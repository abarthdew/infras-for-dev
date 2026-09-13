# 텍스트 처리 (Text Processing)

## Regular Expression (정규 표현식)

**정규 표현식**은 텍스트 패턴을 표현하는 방식이다. 로그 파일 검색, 데이터 변환 등에 자주 사용된다.

### 기본 문법

```
.      아무 문자 (공백 포함)
*      직전 문자가 0개 이상
+      직전 문자가 1개 이상
?      직전 문자가 0개 또는 1개
^      줄의 시작
$      줄의 끝
[]     괄호 안 문자 중 하나
[^]    괄호 안 문자를 제외한 나머지
|      또는 (pipe, 대안)
()     그룹 (캡처)
\      이스케이프 (특수 문자 해석 해제)
```

### 문자 클래스

```
[a-z]      소문자
[A-Z]      대문자
[0-9]      숫자
[a-zA-Z0-9]  알파벳과 숫자
[^0-9]     숫자가 아닌 문자
\w         알파벳, 숫자, _ (word character)
\d         숫자
\s         공백 (space, tab, newline)
```

### 예시

```regex
hello          "hello" 정확히 일치
h.llo          h + 아무 문자 + llo (hello, hallo, h1llo 등)
hel+o          hel + 1개 이상의 l + o (helo, hello, helllo 등)
colou?r        color 또는 colour (u가 0개 또는 1개)
^error         줄의 시작이 "error"
error$         줄의 끝이 "error"
\d{3}-\d{4}    숫자 3개-숫자 4개 (전화번호: 123-4567)
[0-9]{2,4}     숫자 2-4개 (포트 번호 등)
```

---

## grep - 패턴으로 줄 검색

**grep**은 텍스트 파일에서 패턴과 일치하는 줄을 검색한다.

### 기본 사용

```bash
# 파일에서 문자열 검색
grep "ERROR" /var/log/app.log

# 표준 입력에서 검색
echo -e "hello\nworld\nerror" | grep "error"

# 현재 디렉토리의 모든 파일 검색
grep "pattern" *

# 재귀적으로 모든 하위 디렉토리 검색 (-r)
grep -r "TODO" ./src
```

### 주요 옵션

```bash
-i      대소문자 구분 안 함
-v      패턴과 일치하지 않는 줄 출력 (역)
-n      줄 번호 출력
-c      일치하는 줄의 개수만 출력
-o      일치하는 부분만 출력
-l      파일명만 출력
-L      파일명 (일치하지 않는 파일)
-E      정규 표현식 사용 (grep -E = egrep)
-P      Perl 정규 표현식
-A 3    일치 후 3줄 추가 출력 (after)
-B 3    일치 전 3줄 추가 출력 (before)
-C 3    전후 3줄 추가 출력 (context)
```

### 실제 사용

```bash
# 1. 로그 파일에서 에러만 추출
grep "ERROR" app.log

# 2. 특정 IP 주소를 포함한 줄 찾기
grep "192.168.1" /var/log/nginx/access.log

# 3. 패턴과 일치하지 않는 줄 (주석 제거)
grep -v "^#" /etc/nginx/nginx.conf

# 4. 파일명 포함해서 검색
grep -n "function" index.js
# index.js:5: function hello() {

# 5. 대소문자 구분 안 하고 검색
grep -i "error" app.log

# 6. 정규 표현식으로 검색 (5자리 숫자)
grep -E "[0-9]{5}" /var/log/syslog

# 7. 전후 3줄 함께 출력
grep -C 3 "CRITICAL" app.log

# 8. 특정 파일에서만 검색
grep "pattern" *.js
grep -r "pattern" src/  # 재귀

# 9. 일치하는 줄의 개수
grep -c "ERROR" app.log
```

---

## sed - 텍스트 변환 및 필터

**sed (Stream Editor)**는 텍스트를 검색하고 치환하거나 삭제하는 도구다.

### 기본 문법

```bash
sed 's/찾을_패턴/바꿀_패턴/' file.txt
```

### 치환 (Substitution)

```bash
# 1. 첫 번째 일치만 치환
sed 's/hello/goodbye/' file.txt

# 2. 모든 일치를 치환 (g 플래그)
sed 's/hello/goodbye/g' file.txt

# 3. N번째부터 치환
sed 's/hello/goodbye/2g' file.txt  # 2번째부터 모두

# 4. 대소문자 무시 (i 플래그)
sed 's/HELLO/goodbye/i' file.txt

# 5. 파일에 직접 쓰기 (-i)
sed -i 's/old/new/g' file.txt
```

### 줄 삭제

```bash
# 1. 특정 줄 삭제
sed '5d' file.txt          # 5번 줄 삭제
sed '2,4d' file.txt        # 2-4줄 삭제
sed '1d' file.txt          # 첫 줄 삭제

# 2. 패턴과 일치하는 줄 삭제
sed '/^#/d' file.txt       # 주석(#)으로 시작하는 줄 삭제
sed '/^$/d' file.txt       # 빈 줄 삭제
```

### 선택적 실행

```bash
# 1. 특정 줄에서만 실행
sed '5s/hello/goodbye/' file.txt    # 5번 줄에서만
sed '2,4s/old/new/g' file.txt       # 2-4줄에서만

# 2. 패턴과 일치하는 줄에서만
sed '/error/s/old/new/g' file.txt   # "error"를 포함한 줄에서만
```

### 실제 사용

```bash
# 1. 설정 파일에서 주석과 빈 줄 제거
sed -e '/^#/d' -e '/^$/d' /etc/nginx/nginx.conf

# 2. 호스트명 변경
sed 's/localhost/example.com/g' config.txt

# 3. URL 변경
sed 's|http://old.com|https://new.com|g' links.txt

# 4. 숫자 형식 변경 (콤마 추가)
sed 's/\([0-9]\)\([0-9][0-9][0-9]\)$/\1,\2/' numbers.txt
# 1000 -> 1,000

# 5. JSON 정렬 (들여쓰기)
sed 's/,/,\n/g' data.json

# 6. 파일 백업하면서 수정
sed -i.bak 's/old/new/g' file.txt
# file.txt (수정됨), file.txt.bak (원본)
```

---

## awk - 데이터 추출 및 가공

**awk**는 텍스트 파일을 필드 단위로 처리하는 프로그래밍 언어다. 로그 분석, 데이터 변환에 매우 유용하다.

### 기본 문법

```bash
awk 'pattern { action }' file.txt
```

### 필드 (Field)

awk는 기본적으로 공백으로 필드를 구분한다.

```bash
echo "john 30 developer" | awk '{ print $1, $3 }'
# 출력: john developer

# $1: john (첫 번째 필드)
# $2: 30 (두 번째 필드)
# $3: developer (세 번째 필드)
# $NF: developer (마지막 필드)
# NF: 3 (필드 개수)
```

### 구분자 변경

```bash
# CSV 파일 처리 (쉼표로 구분)
awk -F, '{ print $1, $3 }' data.csv

# 콜론으로 구분 (passwd 파일)
awk -F: '{ print $1, $5 }' /etc/passwd
```

### 주요 사용

```bash
# 1. 특정 열만 출력
awk '{ print $1, $3 }' file.txt

# 2. 행 번호 출력
awk '{ print NR, $0 }' file.txt
# 1 line 1
# 2 line 2

# 3. 조건부 처리
awk '$3 > 30 { print $1, $3 }' data.txt   # 3번 필드가 30 초과

# 4. 패턴 매칭
awk '/error/ { print NR, $0 }' app.log    # "error" 포함 줄

# 5. 누적 계산
awk '{ sum += $2 } END { print sum }' numbers.txt

# 6. 라인 단위 조작
awk 'NR % 2 == 0 { print }' file.txt      # 짝수 줄만 출력

# 7. 변수 사용
awk 'BEGIN { total = 0 } { total += $2 } END { print "합계:", total }' sales.txt
```

### 내장 변수

```
NR      현재 줄 번호
NF      현재 줄의 필드 개수
FS      필드 구분자 (기본값: 공백)
OFS     출력 필드 구분자
RS      레코드 구분자
ORS     출력 레코드 구분자
FILENAME 현재 파일명
```

### 실제 사용

```bash
# 1. Nginx 접근 로그 분석
awk '{ print $1 }' /var/log/nginx/access.log | sort | uniq -c | sort -rn
# IP별 접근 횟수 (가장 많은 순)

# 2. CSV 파일 처리
awk -F, 'NR > 1 { sum += $3 } END { print "총액:", sum }' sales.csv

# 3. passwd 파일에서 사용자명과 홈 디렉토리
awk -F: '{ print $1, $6 }' /etc/passwd

# 4. 메모리 사용량 계산 (ps 결과)
ps aux | awk '{ sum += $6 } END { print "총 메모리:", sum, "KB" }'

# 5. 특정 라인 추출 (5번 줄부터 10번 줄까지)
awk 'NR >= 5 && NR <= 10' file.txt

# 6. 형식 변환 (탭으로 정렬)
awk '{ printf "%-15s %10s\n", $1, $3 }' data.txt

# 7. 조건부 필터링 (특정 필드 값 기준)
awk '$2 >= 1000000 { print $1 }' data.txt  # 2번 필드가 100만 이상

# 8. 다중 파일 처리
awk 'FNR == 1 { print "파일:", FILENAME }' *.txt
```

---

## 조합 예시

### 로그 분석

```bash
# 에러 로그 추출 및 시간별로 정렬
grep "ERROR" app.log | awk '{ print $1 }' | sort | uniq -c

# IP와 요청 수 (상위 10)
grep "GET" /var/log/nginx/access.log | awk '{ print $1 }' | sort | uniq -c | sort -rn | head -10

# 시간대별 요청 분석
awk '{ print $4 }' /var/log/nginx/access.log | cut -d: -f1-2 | sort | uniq -c
```

### 데이터 변환

```bash
# CSV를 탭 분리로 변환
sed 's/,/\t/g' input.csv > output.tsv

# 숫자 형식 변경
sed 's/\([0-9]\)\([0-9][0-9][0-9]\)$/\1,\2/' numbers.txt

# JSON 줄 나누기
sed 's/,/,\n/g' data.json
```

---

## 참고

- 정규 표현식은 tool마다 문법이 조금씩 다름 (grep -E, sed, awk 등)
- sed는 한 줄씩, awk는 필드 단위로 처리
- 복잡한 데이터 처리는 Python/Perl이 더 효율적일 수 있음
- 로그 분석이나 간단한 데이터 변환에는 이 도구들이 강력함

## Related Notes

- [Shell Scripting](shell-scripting.md)
- [Linux System Operations](linux-system-operations.md)
