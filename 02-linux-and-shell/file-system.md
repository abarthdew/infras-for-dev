# Linux 파일 시스템 구조

## 파일 시스템의 계층 구조

리눅스의 모든 파일과 디렉토리는 **루트 디렉토리 (`/`)**부터 시작해서 계층적으로 구성된다.

```
/
├── bin              # 기본 실행 파일 (ls, cp, grep 등)
├── sbin             # 관리자 실행 파일 (ifconfig, fdisk 등)
├── boot             # 부팅 관련 파일
├── dev              # 장치 파일 (/dev/null, /dev/sda 등)
├── etc              # 시스템 설정 파일
├── home             # 사용자 홈 디렉토리
├── lib              # 라이브러리 파일
├── media            # 외장 디스크 마운트 지점
├── mnt              # 일시적 파일 시스템 마운트
├── opt              # 선택적 소프트웨어
├── proc             # 프로세스 정보 (가상 파일시스템)
├── root             # root 사용자 홈 디렉토리
├── run              # 런타임 데이터
├── srv              # 서비스 데이터
├── sys              # 시스템 정보 (가상 파일시스템)
├── tmp              # 임시 파일
├── usr              # 사용자 프로그램과 데이터
└── var              # 변경되는 데이터 (로그, 캐시 등)
```

---

## 주요 디렉토리 설명

### `/bin` - 기본 실행 파일

모든 사용자가 사용할 수 있는 기본 명령어들이 저장된다.

```bash
ls /bin
# bash cat cp grep ls mkdir mv rm touch ...
```

### `/sbin` - 관리자 실행 파일

시스템 관리에 필요한 명령어들. 주로 root만 사용한다.

```bash
ls /sbin
# ifconfig shutdown reboot fdisk iptables ...
```

### `/etc` - 시스템 설정 파일

거의 모든 시스템 설정이 여기에 저장된다. **가장 중요한 디렉토리 중 하나.**

```bash
/etc/
├── passwd            # 사용자 계정 정보
├── shadow            # 사용자 비밀번호 (암호화)
├── group             # 그룹 정보
├── sudoers           # sudo 권한 설정
├── hostname          # 호스트 이름
├── fstab             # 파일 시스템 마운트 정보
├── ssh/              # SSH 설정
│   └── sshd_config
├── nginx/            # Nginx 설정
├── mysql/            # MySQL 설정
├── systemd/          # systemd 설정
└── cron.d/           # cron 작업 설정
```

**예시: SSH 설정 파일 수정**

```bash
sudo vi /etc/ssh/sshd_config
# SSH 포트 변경, 루트 접속 제한 등 설정
systemctl restart ssh  # 변경 적용
```

### `/home` - 사용자 홈 디렉토리

각 사용자의 개인 디렉토리.

```bash
/home/
├── john/    # john 사용자의 홈 디렉토리
│   ├── .bashrc          # bash 설정
│   ├── .ssh/            # SSH 키 저장
│   ├── Desktop/
│   ├── Documents/
│   └── Downloads/
└── jane/    # jane 사용자의 홈 디렉토리
```

`~`는 현재 사용자의 홈 디렉토리를 가리킨다.

```bash
cd ~       # /home/john으로 이동
cd ~/app   # /home/john/app으로 이동
```

### `/root` - root 사용자 홈

일반 사용자와 달리 root의 홈 디렉토리는 `/root`다.

```bash
# root 로그인
sudo su
cd ~  # /root으로 이동
```

### `/var` - 변경되는 데이터

시스템이 실행되면서 계속 변경되는 데이터를 저장한다.

```bash
/var/
├── log/          # 시스템 로그
│   ├── syslog
│   ├── auth.log
│   ├── nginx/
│   └── mysql/
├── cache/        # 캐시 데이터
├── spool/        # 메일, 프린터 작업 대기열
├── tmp/          # 임시 파일
└── run/          # 런타임 데이터 (PID 파일 등)
```

**로그 확인**

```bash
# 시스템 로그 확인
tail -f /var/log/syslog

# Nginx 로그
tail -f /var/log/nginx/access.log
tail -f /var/log/nginx/error.log

# 애플리케이션 로그
tail -f /var/log/myapp.log
```

### `/opt` - 선택적 소프트웨어

사용자가 직접 설치한 서드파티 소프트웨어를 저장한다.

```bash
/opt/
├── nodejs/
├── mongodb/
├── custom-app/
└── tools/
```

**패키지 매니저로 설치되지 않은 프로그램**

```bash
# 자체 컴파일한 프로그램 설치
./configure
make
sudo make install --prefix=/opt/myapp
```

### `/usr` - 사용자 프로그램

패키지 매니저로 설치된 프로그램들의 대부분이 여기에 저장된다.

```bash
/usr/
├── bin/          # 사용자 실행 파일 (python, node, git 등)
├── sbin/         # 관리자 실행 파일
├── local/        # 로컬 설치 프로그램
│   ├── bin/
│   ├── lib/
│   └── share/
├── share/        # 공유 데이터
│   ├── doc/      # 문서
│   ├── man/      # man 페이지
│   └── icons/
└── lib/          # 라이브러리
```

### `/tmp` - 임시 파일

시스템이 자동으로 오래된 파일들을 삭제한다.

```bash
/tmp/
├── temp-script-12345
├── mysql-socket-xyz
└── ... (자동 정리됨)
```

### `/dev` - 장치 파일

물리 장치나 가상 장치를 파일처럼 접근하게 해준다.

```bash
/dev/
├── null          # 모든 입력을 버림
├── zero          # 무한 0 바이트 스트림
├── random        # 난수 생성
├── sda           # 첫 번째 하드 디스크
├── sda1          # 첫 번째 파티션
├── tty           # 터미널
└── pts/          # 가상 터미널
```

**흔한 사용**

```bash
# 출력 무시
echo "hello" > /dev/null

# 무한 0 생성 (테스트용)
head -c 1000 /dev/zero > /tmp/largefile

# 난수 생성
head -c 16 /dev/random | od -An -tx1
```

### `/proc` - 프로세스 정보 (가상 파일시스템)

실행 중인 프로세스와 시스템 정보를 파일로 표현한다.

```bash
/proc/
├── cpuinfo       # CPU 정보
├── meminfo       # 메모리 정보
├── loadavg       # 로드 평균
├── uptime        # 시스템 부팅 후 경과 시간
├── [PID]/        # 프로세스별 정보
│   ├── cmdline   # 실행 명령어
│   ├── status    # 프로세스 상태
│   └── fd/       # 열린 파일 디스크립터
└── net/          # 네트워크 정보
```

**시스템 정보 확인**

```bash
# CPU 개수 확인
cat /proc/cpuinfo | grep processor | wc -l

# 메모리 정보
cat /proc/meminfo

# 시스템 부팅 후 경과 시간
cat /proc/uptime

# 특정 프로세스의 명령어 확인
cat /proc/[PID]/cmdline
```

### `/sys` - 시스템 정보 (가상 파일시스템)

하드웨어와 커널의 상태를 조회하고 설정한다.

```bash
/sys/
├── devices/      # 하드웨어 장치
├── class/        # 장치 클래스
├── block/        # 블록 장치
└── power/        # 전원 관리
```

---

## 경로 지정 방식

### 절대 경로 (Absolute Path)

루트 `/`부터 시작하는 완전한 경로.

```bash
/home/ubuntu/app
/etc/nginx/nginx.conf
/var/log/syslog
```

### 상대 경로 (Relative Path)

현재 디렉토리를 기준으로 한 경로.

```bash
# 현재 디렉토리: /home/ubuntu/app
cd config         # /home/ubuntu/app/config로 이동
cd ..             # /home/ubuntu로 이동
cd ../../         # /home으로 이동
cd ../other       # /home/ubuntu/other로 이동
```

### 축약 표현

```bash
~       # 현재 사용자의 홈 디렉토리 (/home/username)
.       # 현재 디렉토리
..      # 부모 디렉토리
-       # 이전 디렉토리
```

---

## 파일 시스템 마운트

디스크 파티션이나 외장 드라이브를 특정 디렉토리에 연결하는 과정을 **마운트**라고 한다.

```bash
# 현재 마운트 상태 확인
mount

# 마운트된 파일 시스템 용량 확인
df -h

# 특정 파일의 디스크 사용량 확인
du -sh /var/log

# USB 드라이브 마운트
sudo mount /dev/sdb1 /mnt/usb

# 마운트 해제
sudo umount /mnt/usb
```

---

## 참고

- 파일 시스템 구조는 대부분의 리눅스 배포판에서 동일하다 (Filesystem Hierarchy Standard).
- `/etc`와 `/var/log`는 시스템 관리할 때 가장 자주 방문하는 디렉토리다.
- 서버 운영할 때는 각 디렉토리의 디스크 사용량을 주기적으로 모니터링해야 한다.

## Related Notes

- [Linux Filesystem and Users](linux-filesystem-and-users.md)
- [Linux System Operations](linux-system-operations.md)
