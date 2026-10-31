# mission1 - 개발 워크스테이션 구축

## 1. 프로젝트 개요

터미널(리눅스 CLI), Docker, Git/GitHub를 직접 세팅하고 사용해 보며, 누구나 같은 방식으로 실행·배포·디버깅할 수 있는 개발 환경을 구성한다. 모든 수행 과정은 명령어와 출력 결과로 이 문서에 기록한다.

## 1-1. 저장소 구조

```text
.
├── README.md                  # 기술 문서 (전체 수행 기록)
├── app                        # Dockerfile 기반 웹 서버
│   ├── Dockerfile             # nginx:alpine 기반 커스텀 이미지
│   └── index.html             # 교체한 정적 콘텐츠
├── docs
│   └── images                 # README에 첨부한 증거 스크린샷
│       ├── github-vscode-login.png
│       └── port-mapping.png
└── practice                   # 터미널/권한 실습 파일
    ├── hello.txt
    └── mydir                  # 권한 실습용 (빈 디렉토리라 Git에는 올라가지 않음)
```

- 출력은 `tree -a -I '.git'` 결과이며, 주석은 설명을 위해 직접 추가했다.

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
- [x] Docker 기본 운영 명령
- [x] hello-world / ubuntu 컨테이너 실행
- [x] Dockerfile 기반 커스텀 이미지
- [x] 포트 매핑 접속 확인
- [x] 바인드 마운트
- [x] Docker 볼륨 영속성
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

- Docker: 29.9.2 (Docker Desktop 4.97.0, WSL2 백엔드)

### 7-1. 버전 및 데몬 점검
```bash
$ docker --version
Docker version 29.9.2, build a82e528
$ docker info
(Client/Server 출력 중 Server 부분 발췌)
Server:
 Containers: 0
 Images: 0
 Server Version: 29.9.2
 Operating System: Docker Desktop
 Kernel Version: 6.18.54.1-microsoft-standard-WSL2
```

### 7-2. hello-world 실행
```bash
$ docker run hello-world
(출력 전체 붙여넣기)
```
### 7-3. 이미지/컨테이너 조회
(docker images, docker ps, docker ps -a, docker logs 출력)

### 7-4. ubuntu 컨테이너 실행
(docker run -it ubuntu bash ~ exit 출력)

### 7-5. attach vs exec, docker stats
(docker run -dit ~ docker ps -a 출력)
- 결론: exec은 실행 중인 컨테이너에 새 프로세스를 추가하므로 exit해도 컨테이너가 유지된다. attach는 메인 프로세스(bash)에 연결하므로 exit하면 컨테이너도 종료된다.

## 8. Dockerfile / 포트 매핑 / 마운트 / 볼륨


### 8-1. 커스텀 이미지 (A안: NGINX 베이스)
- 베이스 이미지: `nginx:alpine`
- 커스텀 포인트: 기본 index.html을 내가 만든 `app/index.html`로 교체 (정적 콘텐츠 교체)

```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
```

### 8-2. 빌드/실행 및 포트 매핑 확인

```bash
$ docker build -t mission1-web .
(빌드 출력 붙여넣기)

$ docker run -d --name web -p 8080:80 mission1-web
(컨테이너 ID 출력 붙여넣기)

$ docker ps
(docker ps 출력 붙여넣기)

$ curl http://localhost:8080
(curl 출력 붙여넣기)
```

- `-p 8080:80`은 호스트 8080 포트를 컨테이너 80 포트에 연결한다는 뜻이다. 컨테이너는 격리된 네트워크를 쓰므로 포트를 연결해야 호스트 브라우저에서 접속할 수 있다.

### 8-3. 포트 매핑 증거
![포트 매핑 접속 화면 (localhost:8080)](docs/images/port-mapping.png)

- `-p 8080:80`: 호스트 8080 포트를 컨테이너 80 포트에 연결한다. 컨테이너는 격리된 네트워크를 쓰므로, 포트를 연결해야 호스트 브라우저에서 접속할 수 있다.

### 8-4. Docker 볼륨 영속성

**목적:** 컨테이너를 삭제해도 볼륨의 데이터가 남는지 확인한다.

#### 볼륨 생성 및 확인
```bash
$ docker volume create mydata
mydata
$ docker volume ls
(출력)
```

#### 컨테이너에 볼륨 연결 후 데이터 쓰기
```bash
$ docker run --name vol-test -v mydata:/data ubuntu bash -c "echo 'persistent data' > /data/test.txt && cat /data/test.txt"
persistent data
```

#### 컨테이너 삭제 전/후
```bash
$ docker ps -a        # 삭제 전: vol-test 존재
(출력)
$ docker rm vol-test
vol-test
$ docker ps -a        # 삭제 후: vol-test 없음
(출력)
```

#### 새 컨테이너로 데이터 확인
```bash
$ docker run --rm -v mydata:/data ubuntu cat /data/test.txt
persistent data
```

#### 볼륨 상세
```bash
$ docker volume inspect mydata
[
    {
        "CreatedAt": "2026-10-31T09:04:22Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/mydata/_data",
        "Name": "mydata",
        "Options": null,
        "Scope": "local"
    }
]
```

#### 정리
- 볼륨은 Docker가 관리하는 저장 공간으로, 컨테이너의 수명과 분리되어 있다. 컨테이너를 삭제해도 데이터가 남는다.
- 바인드 마운트는 호스트 경로를 직접 연결하고, 볼륨은 Docker가 위치를 관리한다. DB 데이터처럼 영속성이 필요한 경우에 적합하다.
- 컨테이너는 일회용으로 만들고, 유지해야 할 데이터는 볼륨에 두는 설계가 재현 가능한 환경의 기반이다.

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
