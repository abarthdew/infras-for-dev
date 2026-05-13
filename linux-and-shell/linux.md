# Linux

## 명령어
아래 예시는 다음과 같은 연습 디렉토리를 전제한다.

```bash
~/study-linux/
├── notes/
│   ├── linux.md
│   └── shell.md
├── logs/
│   ├── app.log
│   └── error.log
└── scripts/
    └── backup.sh
```

### 터미널 열기
```bash
Ctrl + Alt + T
```

### 파일/디렉토리
```bash
# 현재 내가 어느 디렉토리에 있는지 확인
pwd

# 숨김 파일과 자세한 정보를 함께 보기
ls -la ~/study-linux

# 작업 디렉토리 이동
cd ~/study-linux/notes

# 중간 디렉토리까지 한 번에 만들기
mkdir -p ~/study-linux/archive/2026

# 파일 삭제: rm은 되돌리기 어렵기 때문에 먼저 ls로 대상을 확인하는 습관이 좋다
ls ~/study-linux/logs/*.log
rm ~/study-linux/logs/error.log

# 파일 복사
cp ~/study-linux/notes/linux.md ~/study-linux/archive/linux-2026.md

# 파일 이름 변경 또는 위치 이동
mv ~/study-linux/archive/linux-2026.md ~/study-linux/archive/linux-basic.md

# 특정 이름의 파일 찾기
find ~/study-linux -name "*.md"
```

### 파일 보기
```bash
# 짧은 파일 전체 출력
cat ~/study-linux/notes/linux.md

# 긴 파일을 페이지 단위로 읽기
less ~/study-linux/logs/app.log

# 파일 앞부분만 확인
head -n 20 ~/study-linux/logs/app.log

# 파일 끝부분만 확인
tail -n 20 ~/study-linux/logs/app.log

# 로그를 실시간으로 따라가며 보기
tail -f ~/study-linux/logs/app.log

# 특정 단어가 들어간 줄 검색
grep "ERROR" ~/study-linux/logs/app.log
```

### 권한
```bash
# 스크립트 실행 권한 추가
chmod +x ~/study-linux/scripts/backup.sh

# 파일 소유자 변경
sudo chown $USER:$USER ~/study-linux/scripts/backup.sh

# 관리자 권한으로 명령 실행
sudo apt update
```

### 네트워크
```bash
# HTTP 응답 헤더 확인
curl -I https://example.com

# 네트워크 연결 확인
ping -c 4 google.com

# 원격 서버 접속
ssh ubuntu@192.168.0.10

# 로컬 파일을 원격 서버로 복사
scp ~/study-linux/notes/linux.md ubuntu@192.168.0.10:/home/ubuntu/
```

### 패키지 관리
```bash
# 패키지 목록 갱신
sudo apt update

# 설치 가능한 패키지 검색
apt search tree

# 패키지 설치
sudo apt install tree

# 설치된 패키지 제거
sudo apt remove tree
```

## Shell
- GNU bash
```text
GNU Bash(배시, Bourne Again Shell)은 자유 소프트웨어 재단(FSF)의 GNU 프로젝트에서 개발된 유닉스 셸이자 명령어 해석기다. POSIX 표준을 준수하면서도 기존 Bourne Shell(sh)을 확장하여 대화형 사용과 스크립팅 모두에 적합한 기능을 제공한다.

핵심 정보
개발 주체: GNU Project / Free Software Foundation
최초 공개: 1989년
주요 언어: C
라이선스: GNU General Public License v3 또는 그 이후 버전
유지 관리자: Chet Ramey
최신 안정판: 5.2 (2022 년 9 월 출시)

설계와 표준
Bash는 Stephen Bourne이 설계한 전통적 sh 셸의 직계 후손으로, 이름 자체가 “Bourne Again Shell”이라는 말장난에서 유래했다. Korn Shell(ksh)과 C Shell(csh)의 편의 기능을 통합했으며 IEEE POSIX 1003.1 Shell and Tools 규격을 준수하도록 설계되었다. 이러한 호환성 덕분에 대부분의 sh 스크립트가 Bash 에서 수정 없이 실행된다.

주요 기능
Bash는 명령행 편집, 무제한 크기의 명령 이력, 작업 제어(job control), 셸 함수와 별칭(alias), 정수 산술 연산 등을 제공한다. 또한 파이프와 리디렉션, 조건문과 반복문을 통한 스크립트 프로그래밍을 지원해 운영 체제 작업 자동화에 널리 활용된다.

활용과 이식성
GNU 시스템의 기본 셸로 채택되어 있으며, GNU/Linux 를 포함한 대부분의 유닉스 계열 시스템에서 기본 또는 표준 셸로 사용된다. 또한 MS-DOS, OS/2, Windows 등으로 포팅된 버전도 존재해 플랫폼 간 호환성이 높다.

최근 동향
2025 년 기준 자유 소프트웨어 재단은 Bash 5.3 릴리스 후보판을 발표했으며, 계속해서 POSIX 호환성 향상과 성능 개선을 진행 중이다. Bash 는 시스템 관리, 소프트웨어 개발, 네트워크 운영 등 다양한 환경에서 가장 널리 사용되는 CLI 셸 중 하나로 자리 잡고 있다.
```
- Z shell
```text
Z shell(Zsh)는 1990년 Paul Falstad가 개발한 고급 유닉스 셸이자 명령 인터프리터다. Bourne shell의 확장판으로, Bash·KornShell·C shell의 기능을 결합해 자동 완성, 오타 교정, 플러그인 시스템 등 다양한 고급 기능을 제공한다. 2019년 macOS Catalina부터 macOS의 기본 셸로 채택되며 널리 알려졌다.

핵심 정보
개발자: Paul Falstad (1990)
라이선스: MIT 및 BSD 계열 오픈소스
기반: Bourne shell 확장판
기본 셸 채택: macOS Catalina (2019)
언어: C

기능과 특징
Zsh는 상호작용형 셸과 스크립트 실행 환경을 모두 지원한다. 문맥 인식 탭 자동 완성, 철자 교정, 고급 글로빙(**/*.log), 다중 스레드 명령 실행, 그리고 강력한 프롬프트 커스터마이징 기능을 제공한다. 사용자는 ~/.zshrc, ~/.zprofile 등의 설정 파일을 통해 셸 동작을 세밀하게 제어할 수 있다.

플러그인 및 테마 생태계
Zsh의 인기는 풍부한 플러그인과 테마 지원에서 비롯된다. 대표적인 프레임워크인 Oh My Zsh, Prezto, Zinit(구 zplugin)은 수천 개의 플러그인을 통해 Git·Docker·Python 등 다양한 도구와 통합된다. 또한 Powerlevel10k 같은 테마는 시스템 상태와 브랜치 정보를 실시간으로 표시해준다.

Bash와의 비교
Zsh는 Bash와 호환성을 유지하면서도 생산성을 높인다. 배열 인덱스가 1부터 시작하고, 확장 글로빙과 철자 교정 기능이 내장돼 있으며, 우측 프롬프트와 부동소수점 연산을 기본 지원한다. 개발자와 시스템 관리자는 이러한 개선점을 활용해 보다 효율적인 CLI 환경을 구축한다.

현재 활용과 영향
Zsh는 macOS, Linux, BSD 계열 운영체제에서 기본 혹은 선택 셸로 사용된다. 특히 개발자, DevOps 엔지니어, 시스템 관리자에게 인기 있는 환경으로 자리 잡았으며, 오픈소스 커뮤니티를 통해 지속적으로 확장·유지보수되고 있다.
```

## 공식 문서
- man 페이지 읽는 습관 들이기
```bash
man ls
man grep
```

## Q&A: 터미널 프롬프트, `~`, `/`

### 질문
리눅스 터미널에 나오는 `shin@DESKTOP-LBOABJ3:~$` 는 뭐라고 지칭하는가? 처음 경로가 `~`로 되어 있는데, `cd /` 명령어를 입력하는 것과 무엇이 다른가? `~`와 `/`의 차이는 무엇인가?

### 답변
`shin@DESKTOP-LBOABJ3:~$` 는 보통 **프롬프트(prompt)** 라고 부른다. 셸이 사용자에게 "명령어를 입력할 준비가 되었다"고 보여주는 표시다.

구성은 보통 다음처럼 해석할 수 있다.

```bash
shin@DESKTOP-LBOABJ3:~$
```

- `shin`: 현재 로그인한 사용자 이름
- `DESKTOP-LBOABJ3`: 현재 접속한 컴퓨터 또는 호스트 이름
- `~`: 현재 작업 디렉토리가 사용자의 홈 디렉토리라는 뜻
- `$`: 일반 사용자 권한의 셸이라는 뜻

여기서 중요한 차이는 `~`와 `/`다.

`~`는 **현재 사용자의 홈 디렉토리**를 뜻하는 축약 표현이다. 예를 들어 사용자가 `shin`이라면 보통 다음 경로를 의미한다.

```bash
/home/shin
```

그래서 아래 두 명령은 거의 같은 의미다.

```bash
cd ~
cd /home/shin
```

반면 `/`는 **리눅스 파일 시스템의 최상위 디렉토리**, 즉 루트 디렉토리다. 모든 디렉토리와 파일은 `/` 아래에서 시작한다.

```bash
cd /
```

이 명령을 입력하면 홈 디렉토리가 아니라 파일 시스템의 가장 위로 이동한다. 예를 들어 `/` 아래에는 보통 이런 디렉토리들이 있다.

```bash
bin  boot  dev  etc  home  usr  var
```

정리하면 다음과 같다.

```bash
~  = 내 홈 디렉토리, 예: /home/shin
/  = 파일 시스템 전체의 시작점
```

따라서 `shin@DESKTOP-LBOABJ3:~$` 에서 `~`가 보인다는 것은 "현재 나는 내 홈 디렉토리에 있다"는 뜻이고, `cd /`를 입력하면 "홈 디렉토리를 벗어나 파일 시스템의 최상위로 이동한다"는 뜻이다.

## Q&A: 리눅스 기본 디렉토리와 `/dev/null`

### 질문
리눅스 기본 폴더의 역할은 각각 무엇인가? `home`, `usr`, `bin`, `etc`, `dev`, `lib` 등이 있는데, 이것이 명령어에 붙는 `/dev/null`과 관련이 있는가?

### 답변
리눅스의 기본 디렉토리들은 아무렇게나 만들어진 폴더가 아니라, **파일 시스템에서 각 종류의 파일을 어디에 둘지 정한 약속**에 가깝다. 리눅스에서는 프로그램, 설정, 장치, 사용자 파일까지 모두 하나의 디렉토리 트리 안에 배치된다.

자주 보는 디렉토리의 역할은 다음과 같다.

```text
/        파일 시스템의 최상위 디렉토리
/home    일반 사용자의 홈 디렉토리
/bin     기본 명령어 실행 파일
/usr     사용자용 프로그램, 라이브러리, 문서 등
/etc     시스템 설정 파일
/dev     장치 파일
/lib     시스템 실행에 필요한 기본 라이브러리
/var     자주 변하는 데이터
/tmp     임시 파일
/root    root 사용자의 홈 디렉토리
```

조금 더 구체적으로 보면 다음과 같다.

`/home`은 일반 사용자의 개인 공간이다. 예를 들어 사용자 이름이 `shin`이면 보통 홈 디렉토리는 `/home/shin`이다.

```bash
/home/shin
```

`/bin`은 `ls`, `cp`, `mv`, `cat` 같은 기본 명령어 실행 파일이 있는 곳이다. 사용자가 `ls`를 입력하면 셸은 실제로는 이런 실행 파일을 찾아 실행한다.

```bash
/bin/ls
/bin/cp
```

`/usr`는 많은 사용자용 프로그램과 라이브러리, 문서가 들어가는 큰 영역이다. 요즘 리눅스에서는 `/usr/bin`에 대부분의 명령어가 들어 있는 경우가 많다.

```bash
/usr/bin/python3
/usr/bin/git
```

`/etc`는 시스템 설정 파일이 있는 곳이다. 프로그램 실행 파일이 아니라, 프로그램이나 시스템이 어떤 방식으로 동작할지 정하는 설정 파일들이 들어간다.

```bash
/etc/hosts
/etc/passwd
/etc/ssh/sshd_config
```

`/dev`는 장치 파일이 있는 곳이다. 여기서 중요한 점은 리눅스가 하드디스크, 터미널, 랜덤 생성기 같은 장치도 파일처럼 다룬다는 것이다.

```bash
/dev/sda
/dev/tty
/dev/null
/dev/random
```

따라서 `/dev/null`은 `/dev` 디렉토리와 직접 관련이 있다. `/dev/null`은 실제 일반 파일이 아니라 **특수 장치 파일**이다.

`/dev/null`의 역할은 "버리는 곳"이다. 여기에 출력하면 내용이 저장되지 않고 사라진다.

```bash
echo "hello" > /dev/null
```

위 명령은 `hello`를 출력하지만, 그 출력이 화면이나 파일에 남지 않고 `/dev/null`로 버려진다.

실무에서는 보통 명령어의 출력을 숨기고 싶을 때 사용한다.

```bash
some-command > /dev/null
```

에러 메시지까지 숨기고 싶다면 표준 에러도 함께 보낸다.

```bash
some-command > /dev/null 2>&1
```

여기서 `>`는 표준 출력을 리다이렉션한다는 뜻이고, `2>&1`은 표준 에러를 표준 출력과 같은 곳으로 보낸다는 뜻이다. 결과적으로 일반 출력과 에러 출력이 모두 `/dev/null`로 가서 사라진다.

`/lib`는 시스템과 프로그램 실행에 필요한 라이브러리가 들어간다. 라이브러리는 프로그램들이 공통으로 사용하는 기능 묶음이다.

```bash
/lib
/lib64
/usr/lib
```

정리하면 다음과 같다.

```text
/home      사용자 개인 공간
/bin       기본 명령어
/usr       사용자용 프로그램과 라이브러리
/etc       설정 파일
/dev       장치 파일
/lib       실행에 필요한 라이브러리
/dev/null  출력을 버리는 특수 장치 파일
```

즉 `/dev/null`은 이름만 우연히 그런 것이 아니라, **장치 파일을 담는 `/dev` 디렉토리 아래에 있는 특수한 장치 파일**이다.

## Q&A: `/dev/null`에서 "출력을 버린다"는 의미

### 질문
`/dev/null`과 관련해서, "출력을 버린다"는 게 무슨 뜻인가? `/dev/null`을 붙이지 않는 것과 어떻게 다른가?

### 답변
명령어를 실행하면 결과가 보통 터미널 화면에 출력된다. 예를 들어 다음 명령을 실행하면 `hello`가 화면에 보인다.

```bash
echo "hello"
```

결과는 다음과 같다.

```text
hello
```

그런데 출력을 `/dev/null`로 보내면 화면에 아무것도 보이지 않는다.

```bash
echo "hello" > /dev/null
```

여기서 `>`는 출력 방향을 바꾸는 **리다이렉션(redirection)** 이다. 원래는 터미널 화면으로 가야 할 출력이 `/dev/null`로 간다.

`/dev/null`은 받은 내용을 저장하지 않고 즉시 버리는 특수 파일이다. 그래서 "출력을 버린다"는 말은 **명령어의 실행 결과를 화면에도 보여주지 않고, 파일에도 저장하지 않고, 그냥 없애 버린다**는 뜻이다.

차이를 비교하면 다음과 같다.

```bash
echo "hello"
```

```text
hello
```

```bash
echo "hello" > /dev/null
```

```text
# 아무것도 출력되지 않음
```

중요한 점은 `/dev/null`을 붙인다고 명령어가 실행되지 않는 것은 아니라는 점이다. 명령어는 정상적으로 실행된다. 다만 그 명령어가 만들어낸 출력만 보이지 않게 된다.

예를 들어 다음 명령은 `ls`의 결과를 화면에 보여준다.

```bash
ls
```

하지만 다음 명령은 `ls`를 실행하되, 목록 출력은 버린다.

```bash
ls > /dev/null
```

이런 방식은 주로 "결과 내용은 필요 없고, 성공했는지 실패했는지만 알고 싶을 때" 사용한다.

```bash
ls /home/shin > /dev/null
```

이 명령은 `/home/shin` 디렉토리 목록을 화면에 보여주지 않는다. 하지만 명령이 성공했는지는 종료 상태(exit status)로 확인할 수 있다.

```bash
ls /home/shin > /dev/null
echo $?
```

`0`이 나오면 성공, `0`이 아닌 값이 나오면 실패를 의미한다.

또 하나 중요한 구분이 있다. 기본적으로 `>`는 **표준 출력(stdout)** 만 `/dev/null`로 보낸다. 에러 메시지인 **표준 에러(stderr)** 는 그대로 화면에 보일 수 있다.

```bash
ls /not-exist > /dev/null
```

위 명령은 일반 출력은 버리지만, `/not-exist`가 없다는 에러 메시지는 화면에 보인다.

에러까지 모두 숨기려면 다음처럼 쓴다.

```bash
ls /not-exist > /dev/null 2>&1
```

정리하면 다음과 같다.

```text
명령어만 실행              결과가 화면에 보인다
명령어 > /dev/null         일반 출력(stdout)을 버린다
명령어 > /dev/null 2>&1    일반 출력(stdout)과 에러(stderr)를 모두 버린다
```

따라서 `/dev/null`을 붙이지 않는 것과 붙이는 것의 차이는 **명령어 실행 여부가 아니라, 출력 결과를 사용자에게 보여줄지 버릴지**의 차이다.
