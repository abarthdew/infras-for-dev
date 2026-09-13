# Linux Filesystem and Users

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


## Q&A: `sudo -i`, root 사용자, `/` 루트 디렉토리의 차이

### 질문
`sudo -i`를 입력하면 user name이 `root`로 바뀌는데, 이것은 무엇을 뜻하는가? `/` 경로의 root라는 뜻인가?

### 답변
`sudo -i`를 입력했을 때 사용자 이름이 `root`로 바뀌는 것은 **root 사용자 계정으로 로그인 셸을 연 것**을 뜻한다. 이것은 `/` 경로의 root 디렉토리와는 다른 개념이다.

먼저 두 가지를 분리해야 한다.

```text
root 사용자  = 리눅스의 최고 관리자 계정
/            = 파일 시스템의 최상위 디렉토리
/root        = root 사용자의 홈 디렉토리
```

`root` 사용자는 리눅스 시스템에서 가장 강한 권한을 가진 관리자 계정이다. 파일 삭제, 패키지 설치, 시스템 설정 변경, 서비스 제어 같은 거의 모든 작업을 할 수 있다.

```bash
sudo -i
```

이 명령은 현재 사용자에게 sudo 권한이 있을 때, root 사용자의 로그인 셸을 시작한다. 그래서 프롬프트가 보통 다음처럼 바뀐다.

```bash
root@hostname:~#
```

여기서 `root`는 현재 사용자가 root 계정이라는 뜻이다. 마지막 기호도 `$`에서 `#`로 바뀌는 경우가 많다.

```text
$  일반 사용자 셸
#  root 사용자 셸
```

하지만 이것이 `/` 디렉토리로 이동했다는 뜻은 아니다. `sudo -i`를 하면 보통 root 사용자의 홈 디렉토리인 `/root`로 이동한다.

```bash
pwd
```

결과는 보통 다음과 비슷하다.

```text
/root
```

즉 다음 셋은 다르다.

```text
root      사용자 이름
/root     root 사용자의 홈 디렉토리
/         파일 시스템의 최상위 디렉토리
```

`/`는 모든 파일과 디렉토리가 시작되는 최상위 경로다.

```bash
cd /
pwd
```

결과는 다음과 같다.

```text
/
```

`/root`는 그 `/` 아래에 있는 디렉토리 중 하나이며, root 사용자의 개인 홈 디렉토리다.

```bash
cd /root
pwd
```

결과는 다음과 같다.

```text
/root
```

일반 사용자의 홈 디렉토리가 `/home/shin` 같은 곳이라면, root 사용자의 홈 디렉토리는 보통 `/root`다.

```text
shin 사용자의 홈   /home/shin
root 사용자의 홈   /root
파일 시스템 루트   /
```

`sudo -i`와 `sudo 명령어`도 구분해야 한다.

```bash
sudo apt update
```

이 명령은 `apt update` 하나만 root 권한으로 실행한다.

반면 다음 명령은 root 셸 자체를 연다.

```bash
sudo -i
```

이후 입력하는 명령들은 root 권한으로 실행된다.

```bash
whoami
```

결과는 다음과 같다.

```text
root
```

root 셸에서 빠져나오려면 다음 명령을 입력한다.

```bash
exit
```

정리하면 다음과 같다.

```text
sudo command  특정 명령 하나를 root 권한으로 실행
sudo -i       root 사용자의 로그인 셸로 전환
root          최고 관리자 사용자
/root         root 사용자의 홈 디렉토리
/             파일 시스템 최상위 디렉토리
```

따라서 `sudo -i` 후에 보이는 `root`는 `/` 경로를 뜻하는 것이 아니라, **현재 셸이 root 사용자 권한으로 실행 중**이라는 뜻이다. `/`는 경로이고, `root`는 사용자 계정이며, `/root`는 root 사용자의 홈 디렉토리다.

---

## Q&A: 사용자 및 그룹 관리

### 질문
리눅스에서 새로운 사용자를 추가하거나 관리하려면 어떻게 해야 할까?

### 답변

**사용자 관리 파일**

리눅스의 사용자 정보는 다음 파일들에 저장된다:

```bash
/etc/passwd      # 사용자 계정 정보 (암호화되지 않은)
/etc/shadow      # 사용자 암호 (암호화됨)
/etc/group       # 그룹 정보
/etc/gshadow     # 그룹 암호 (거의 사용 안 함)
```

**/etc/passwd 형식**

```bash
cat /etc/passwd
```

```
root:x:0:0:root:/root:/bin/bash
ubuntu:x:1000:1000:ubuntu:/home/ubuntu:/bin/bash
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
```

각 필드:

```
사용자명:암호:UID:GID:설명:홈디렉토리:셸
```

- **UID**: User ID (0 = root, 1-999 = 시스템 사용자, 1000+ = 일반 사용자)
- **GID**: 기본 Group ID
- **홈디렉토리**: 로그인 후 시작 위치
- **셸**: 로그인 셸 (nologin이면 로그인 불가)

**/etc/shadow 형식**

```bash
# root만 읽을 수 있음
sudo cat /etc/shadow
```

```
root:$6$...(암호화):18000:0:99999:7:::
ubuntu:$6$...(암호화):18001:0:99999:7:::
```

**useradd** - 사용자 추가

```bash
# 기본 사용자 추가
sudo useradd john

# 홈 디렉토리와 함께 추가 (-m)
sudo useradd -m john

# 특정 그룹에 속하는 사용자 추가 (-g)
sudo useradd -m -g developers john

# 여러 그룹에 속함 (-G)
sudo useradd -m -G developers,sudo john

# 셸 지정 (-s)
sudo useradd -m -s /bin/bash john

# 홈 디렉토리 경로 지정 (-d)
sudo useradd -m -d /custom/home/john john

# 한 번에 지정
sudo useradd -m -s /bin/bash -G sudo,developers -c "John Doe" john
```

**passwd** - 비밀번호 설정/변경

```bash
# 현재 사용자 비밀번호 변경
passwd

# 다른 사용자의 비밀번호 설정 (root만)
sudo passwd john

# 비밀번호 지우기
sudo passwd -d john

# 비밀번호 잠금
sudo passwd -l john

# 비밀번호 해제
sudo passwd -u john
```

**usermod** - 사용자 정보 수정

```bash
# 그룹 변경
sudo usermod -g developers john

# 추가 그룹 지정 (-a: append)
sudo usermod -aG sudo john

# 홈 디렉토리 변경
sudo usermod -d /new/home/john john

# 셸 변경
sudo usermod -s /bin/zsh john

# 사용자명 변경
sudo usermod -l jane john  # john -> jane

# 계정 비활성화
sudo usermod -L john  # 로그인 불가
sudo usermod -U john  # 로그인 가능
```

**userdel** - 사용자 삭제

```bash
# 사용자만 삭제 (홈 디렉토리 유지)
sudo userdel john

# 사용자와 홈 디렉토리 함께 삭제 (-r)
sudo userdel -r john
```

**그룹 관리**

```bash
# 그룹 생성
sudo groupadd developers

# 그룹에 사용자 추가
sudo usermod -aG developers john

# 그룹 확인
cat /etc/group | grep developers
# developers:x:1001:john,jane,bob

# 그룹 삭제
sudo groupdel developers

# 그룹명 변경
sudo groupmod -n backend developers
```

**id** - 사용자 정보 확인

```bash
# 현재 사용자
id
# uid=1000(ubuntu) gid=1000(ubuntu) groups=1000(ubuntu),4(adm),27(sudo)

# 특정 사용자
id john
# uid=1001(john) gid=1001(john) groups=1001(john),1002(developers)
```

**groups** - 속한 그룹 확인

```bash
# 현재 사용자
groups
# ubuntu adm sudo

# 특정 사용자
groups john
# john developers sudo
```

**실제 사용 예**

```bash
# 1. 개발자 그룹 만들고 사용자 추가
sudo groupadd developers
sudo useradd -m -s /bin/bash -G developers alice
sudo useradd -m -s /bin/bash -G developers bob

# 2. 개발 프로젝트 디렉토리 권한 설정
sudo chown -R :developers /home/shared/project
sudo chmod -R 775 /home/shared/project
# 이제 developers 그룹 멤버들이 자유롭게 수정 가능

# 3. 웹 서버 사용자 추가
sudo useradd -m -s /usr/sbin/nologin www-app
sudo usermod -aG www-data www-app
# nologin 셸이므로 SSH 접속 불가능 (자동 스크립트만 실행)

# 4. 사용자 확인
grep "john\|jane" /etc/passwd
id john
groups john
```

**sudo 권한 설정**

```bash
# /etc/sudoers 파일 편집 (visudo 사용!)
sudo visudo

# 파일 내용
john ALL=(ALL) NOPASSWD: /usr/bin/systemctl
# john은 systemctl을 비밀번호 없이 실행 가능

bob ALL=(ALL) ALL
# bob은 모든 명령어를 sudo로 실행 가능 (비밀번호 필요)
```

