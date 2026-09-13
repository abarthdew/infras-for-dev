# Linux 권한 관리 (Permissions)

## 권한의 개념

리눅스는 **파일과 디렉토리에 대한 접근 권한을 엄격히 관리**한다. 각 파일/디렉토리는 소유자, 소유 그룹, 다른 사용자별로 **읽기(r), 쓰기(w), 실행(x)** 권한을 가진다.

---

## 권한 표기법

### ls -l로 권한 확인

```bash
ls -l /home/ubuntu/app/index.js
```

```
-rw-r--r-- 1 ubuntu ubuntu 1024 2026-05-23 10:30 index.js
```

각 부분의 의미:

```
-rw-r--r-- 1 ubuntu ubuntu 1024 2026-05-23 10:30 index.js
│││││││││ │ ││││││ ││││││ ││││ └─ 파일 이름
│││││││││ │ ││││││ ││││││ └────── 파일 크기
│││││││││ │ ││││││ └──────────── 수정 시간
│││││││││ │ └────────────────── 소유 그룹 (ubuntu)
│││││││││ └──────────────────── 소유자 (ubuntu)
│││││││││ └──────────────────── 하드링크 수
│││││││
││││││└─ 다른 사용자 권한 (r--: 읽기만 가능)
│││││└── 다른 사용자 권한
││││└─── 소유 그룹 권한 (r--: 읽기만 가능)
│││└──── 소유 그룹 권한
││└───── 소유 그룹 권한
│└────── 소유자 권한 (rw-: 읽기, 쓰기 가능)
└─────── 파일 타입 (-: 파일, d: 디렉토리, l: 심볼릭 링크)
```

### 권한 종류

```
r (read)    읽기 권한 (4)
w (write)   쓰기 권한 (2)
x (execute) 실행 권한 (1)
```

---

## chmod - 권한 변경

### 문자 표기법 (Symbolic)

```bash
# 소유자에게 실행 권한 추가
chmod u+x script.sh

# 모든 사용자에게 실행 권한 추가
chmod a+x script.sh

# 그룹에서 쓰기 권한 제거
chmod g-w config.txt

# 소유자는 rwx, 그룹은 rx, 다른 사용자는 rx
chmod u=rwx,g=rx,o=rx file.txt
```

**문자 표기법**

```
u (user/owner)    소유자
g (group)         소유 그룹
o (others)        다른 사용자
a (all)           모든 사용자

+ 권한 추가
- 권한 제거
= 권한 설정 (기존 권한 제거 후 설정)

r 읽기
w 쓰기
x 실행
```

### 숫자 표기법 (Octal)

각 권한을 숫자로 표현한다.

```
r (read)    = 4
w (write)   = 2
x (execute) = 1
```

**예시**

```bash
# 755 = rwxr-xr-x
# 7 (소유자: 4+2+1) 5 (그룹: 4+1) 5 (다른: 4+1)
chmod 755 script.sh

# 644 = rw-r--r--
# 6 (소유자: 4+2) 4 (그룹: 4) 4 (다른: 4)
chmod 644 config.txt

# 700 = rwx------
# 7 (소유자: 4+2+1) 0 (그룹: 없음) 0 (다른: 없음)
chmod 700 private_script.sh
```

**흔한 권한 설정**

```bash
755   # 실행 파일, 디렉토리 (소유자만 쓰기 가능)
644   # 일반 파일 (소유자만 쓰기 가능)
700   # 개인 파일/디렉토리 (소유자만 접근 가능)
777   # 모든 사용자가 모든 권한 (보안 위험!절대 금지)
```

### 재귀적 권한 변경

```bash
# 디렉토리 내 모든 파일과 하위 디렉토리 권한 변경
chmod -R 755 /home/ubuntu/app

# 숨김 파일도 포함해서 변경
chmod -R 755 /home/ubuntu/app/.*
```

---

## chown - 소유자/그룹 변경

파일의 소유자나 소유 그룹을 변경한다.

```bash
# 소유자만 변경
chown ubuntu file.txt

# 소유자와 그룹 변경
chown ubuntu:ubuntu file.txt

# 그룹만 변경
chown :www-data file.txt

# 디렉토리 내 모든 파일 재귀적 변경
chown -R ubuntu:ubuntu /home/ubuntu/app
```

**실제 사용 예**

```bash
# 웹 서버가 접근할 수 있도록 소유자 변경
sudo chown -R www-data:www-data /var/www/html

# 배포 디렉토리 권한 설정
sudo chown -R ubuntu:ubuntu /home/ubuntu/app
sudo chmod -R 755 /home/ubuntu/app
```

---

## 특수 권한 (Special Permissions)

### setuid (Set User ID)

실행 파일에 setuid가 설정되면, **파일을 실행한 사용자가 아니라 파일 소유자 권한으로 실행**된다.

```bash
# 4755 = setuid + rwxr-xr-x
chmod 4755 /usr/bin/sudo
```

**예시**: `sudo` 명령어는 root가 소유하고 setuid가 설정되어 있어서, 일반 사용자가 실행해도 root 권한으로 작동한다.

### setgid (Set Group ID)

실행 파일에 setgid가 설정되면, **파일을 실행한 사용자가 아니라 파일 소유 그룹 권한으로 실행**된다.

```bash
# 2755 = setgid + rwxr-xr-x
chmod 2755 /path/to/script
```

**디렉토리에 setgid 설정**

```bash
# 이 디렉토리에 만든 파일들이 자동으로 그룹 상속
chmod 2755 /home/shared_dir
```

### sticky bit

**디렉토리에 sticky bit가 설정되면, 파일 소유자와 root만 파일을 삭제**할 수 있다. (다른 사용자는 삭제 불가)

```bash
# 1777 = sticky bit + rwxrwxrwx
chmod 1777 /tmp
```

**예시**: `/tmp` 디렉토리는 모든 사용자가 파일을 생성할 수 있지만, 자신이 만든 파일만 삭제할 수 있다.

---

## umask - 기본 권한 설정

**umask**는 새 파일이나 디렉토리 생성 시 기본 권한을 결정한다.

```bash
# 현재 umask 확인
umask

# 출력 예
0022
```

**계산 방식**

```
파일의 기본 권한 = 666 (rw-rw-rw-) - umask
디렉토리 기본 권한 = 777 (rwxrwxrwx) - umask

umask가 0022라면:
파일:      666 - 022 = 644 (rw-r--r--)
디렉토리:  777 - 022 = 755 (rwxr-xr-x)
```

**umask 설정**

```bash
# 임시 설정 (현재 셸에서만)
umask 0022

# 영구 설정 (~/.bashrc에 추가)
echo "umask 0022" >> ~/.bashrc
source ~/.bashrc
```

---

## 실제 사용 사례

### 웹 서버 디렉토리 권한

```bash
# 웹 서버가 읽기 권한만 필요
sudo chown -R www-data:www-data /var/www/html
sudo chmod -R 755 /var/www/html

# 업로드 디렉토리는 쓰기도 필요
sudo chmod 775 /var/www/html/uploads
```

### SSH 키 파일 권한

```bash
# SSH 키는 소유자만 읽을 수 있어야 함 (매우 중요!)
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub

# .ssh 디렉토리는 소유자만 접근 가능
chmod 700 ~/.ssh
```

### 배포 스크립트 권한

```bash
# 배포 스크립트는 실행 가능해야 함
chmod +x /home/ubuntu/deploy.sh

# 민감한 설정 파일은 소유자만 읽을 수 있어야 함
chmod 600 /home/ubuntu/.env
```

### 공유 디렉토리 권한

```bash
# 팀원들과 공유하는 디렉토리
mkdir /shared
chmod 2775 /shared     # setgid + rwxrwxr-x
chown :team /shared    # 그룹: team

# 이제 team 그룹 멤버들은 자유롭게 파일 생성/삭제 가능
```

---

## 권한 확인하기

```bash
# 특정 사용자가 파일을 읽을 수 있는지 확인
test -r file.txt && echo "읽기 권한 있음" || echo "읽기 권한 없음"

# 특정 사용자가 쓸 수 있는지 확인
test -w file.txt && echo "쓰기 권한 있음" || echo "쓰기 권한 없음"

# 실행 권한 확인
test -x script.sh && echo "실행 권한 있음" || echo "실행 권한 없음"
```

---

## 참고

- **보안 원칙**: 필요한 최소 권한만 부여 (Principle of Least Privilege)
- setuid, setgid, sticky bit는 자주 사용되지 않으므로 필요할 때만 사용
- SSH 키 파일의 권한은 매우 중요함 (항상 600)
- 웹 서버 디렉토리는 보통 755, 파일은 644

## Related Notes

- [Linux Filesystem and Users](linux-filesystem-and-users.md)
- [Linux Remote Access](linux-remote-access.md)
