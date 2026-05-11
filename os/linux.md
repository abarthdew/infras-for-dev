# 리눅스

## 명령어
### 파일/디렉토리
```bash
pwd
ls
cd
mkdir
rm
cp
mv
find
```

### 파일 보기
```bash
cat
less
head
tail
grep
```

### 권한
```bash
chmod
chown
sudo
```

### 네트워크
```bash
curl
ping
ssh
scp
```

### 패키지 관리
```bash
apt
apt install
apt update
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
