# mission1 - 개발 워크스테이션 구축

## 1. 프로젝트 개요

터미널(리눅스 CLI), Docker, Git/GitHub를 직접 세팅하고 사용해 보며, 누구나 같은 방식으로 실행·배포·디버깅할 수 있는 개발 환경을 구성한다. 모든 수행 과정은 명령어와 출력 결과로 이 문서에 기록한다.

## 2. 실행 환경

- OS: Windows + WSL2 (Ubuntu 26.04.1 LTS)
- 쉘/터미널: bash (/bin/bash), VSCode 통합 터미널(WSL: Ubuntu)
- Git: 2.53.0
- Docker: (작성 예정)
- 참고: 안내문은 macOS의 OrbStack 기준이지만, Windows 환경이라 WSL2 + Docker Desktop으로 대체한다.

```bash
$ uname -a
Linux DESKTOP-GPNP5PR 6.18.54.1-microsoft-standard-WSL2 #1 SMP PREEMPT_DYNAMIC Tue Oct  6 19:13:07 UTC 2026 x86_64 GNU/Linux
$ echo $SHELL
/bin/bash
$ git --version
git version 2.53.0
```

## 3. 수행 항목 체크리스트

- [x] 터미널 기본 조작 (위치/목록/이동/생성/복사/이동/삭제/내용 확인)
- [x] 권한 실습 (파일 1개, 디렉토리 1개)
- [ ] Docker 설치 및 점검
- [ ] Docker 기본 운영 명령
- [ ] hello-world / ubuntu 컨테이너 실행
- [ ] Dockerfile 기반 커스텀 이미지
- [ ] 포트 매핑 접속 확인
- [ ] 바인드 마운트
- [ ] Docker 볼륨 영속성
- [x] Git 설정
- [x] GitHub/VSCode 연동 증거

## 4. 터미널 조작 로그

### 4-1. 기본 조작

```bash
$ mkdir practice
$ cd practice
$ pwd
/home/sujin/mission1/practice
$ touch empty.txt
$ echo "hello terminal" > hello.txt
$ cat hello.txt
hello terminal
$ mkdir mydir
```

### 4-2. 복사, 이동/이름변경, 삭제

```bash
$ cp hello.txt hello_copy.txt
$ mkdir backup
$ mv hello_copy.txt renamed.txt
$ mv renamed.txt backup/
$ ls -R
.:
backup  empty.txt  hello.txt  mydir

./backup:
renamed.txt

./mydir:
$ rm empty.txt
$ rm -r backup
$ ls
hello.txt  mydir
```

### 4-3. 목록 확인 (숨김 파일 포함)

```bash
$ ls -a
.  ..  hello.txt  mydir
$ ls -la
total 16
drwxr-xr-x 3 sujin sujin 4096 Oct 31 14:29 .
drwxr-xr-x 4 sujin sujin 4096 Oct 31 14:20 ..
-rw------- 1 sujin sujin   15 Oct 31 14:21 hello.txt
drwx------ 2 sujin sujin 4096 Oct 31 14:23 mydir
```

## 5. 권한 실습 (변경 전/후 비교)

```bash
# 파일
$ chmod 644 hello.txt
$ ls -l hello.txt
-rw-r--r-- 1 sujin sujin 15 Oct 31 14:21 hello.txt     # 변경 전
$ chmod 600 hello.txt
$ ls -l hello.txt
-rw------- 1 sujin sujin 15 Oct 31 14:21 hello.txt     # 변경 후

# 디렉토리
$ chmod 755 mydir
$ ls -ld mydir
drwxr-xr-x 2 sujin sujin 4096 Oct 31 14:23 mydir       # 변경 전
$ chmod 700 mydir
$ ls -ld mydir
drwx------ 2 sujin sujin 4096 Oct 31 14:23 mydir       # 변경 후
```

| 대상 | 변경 전 | 변경 후 |
|---|---|---|
| hello.txt | -rw-r--r-- (644) | -rw------- (600) |
| mydir | drwxr-xr-x (755) | drwx------ (700) |

### 해석

- 권한 문자열은 `유형 + 소유자 + 그룹 + 기타` 순서다. 맨 앞 `-`는 파일, `d`는 디렉토리다.
- r=4, w=2, x=1을 더한다. 644는 `rw-`(6) `r--`(4) `r--`(4), 755는 `rwx`(7) `r-x`(5) `r-x`(5)이다.
- 디렉토리의 x는 "그 안으로 들어갈 수 있는 권한"이라, 700이면 소유자만 접근할 수 있다.

## 6. 절대 경로와 상대 경로

- 절대 경로: 루트(`/`)부터 시작하는 전체 경로. 예: `/home/sujin/mission1/practice`
- 상대 경로: 현재 위치 기준 경로. 예: 현재 `/home/sujin/mission1`일 때 `practice`, 상위로는 `..`
- 어디서 실행해도 같은 곳을 가리키는 것이 절대 경로, 현재 위치에 따라 달라지는 것이 상대 경로다.

## 7. Docker 설치 및 운영

(작성 예정)

## 8. Dockerfile / 포트 매핑 / 마운트 / 볼륨

(작성 예정)

## 9. Git 설정 및 GitHub 연동

```bash
$ git config --list
user.name=kimdudin
user.email=(마스킹)
init.defaultbranch=main
core.repositoryformatversion=0
core.filemode=true
core.bare=false
core.logallrefupdates=true
remote.origin.url=https://github.com/kimdudin/mission1.git
remote.origin.fetch=+refs/heads/*:refs/remotes/origin/*
branch.main.remote=origin
```

![VSCode GitHub 로그인 및 push 성공](docs/images/github-vscode-login.png)

## 10. 트러블슈팅

### 10-1. 재부팅 후 Ubuntu가 자동 설치되지 않음
- 문제: `wsl --install` 후 재부팅했더니 Ubuntu가 설치되지 않고 ubuntu.com 웹사이트가 열림
- 원인 가설: WSL 기능만 활성화되고 배포판 설치는 이어지지 않음
- 확인: `wsl --list --verbose`로 배포판 목록 확인
- 해결: `wsl --install -d Ubuntu`로 직접 설치

### 10-2. VSCode 터미널이 PowerShell로 열림
- 문제: 프롬프트가 `PS Microsoft.PowerShell.Core\FileSystem::\\wsl.localhost\Ubuntu\...`로 표시됨
- 원인 가설: WSL 연결 없이 네트워크 경로로 폴더를 열었음
- 확인: VSCode 왼쪽 아래에 `WSL: Ubuntu` 표시 여부 확인
- 해결: Ubuntu 터미널에서 `cd ~/mission1` 후 `code .`로 실행

### 10-3. git commit 실패 (empty ident name)
- 문제: `fatal: empty ident name not allowed`
- 원인 가설: 새 컴퓨터라 Git 사용자 정보가 비어 있음
- 확인: 오류 메시지가 `user.name`, `user.email` 설정을 안내
- 해결: `git config --global user.name`, `user.email` 설정 후 재시도
