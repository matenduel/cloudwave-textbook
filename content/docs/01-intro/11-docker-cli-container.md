---
title: "CLI - Container"
weight: 41
draft: false
---

## 1.2. Docker 컨테이너 관리

### 시작하기에 앞서

- 컨테이너는 `Docker` 이미지로부터 생성된 **하나의 실행 중인 프로세스**입니다.
- 프로세스(`PID=1`)가 종료되면 컨테이너도 함께 종료됩니다.



### 컨테이너 실행하기

> https://docs.docker.com/engine/reference/commandline/run/

```shell
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```



**주요 옵션**

| Option          | Short | Default | Description                                                  |
| --------------- | ----- | ------- | ------------------------------------------------------------ |
| `--name`        |       |         | 컨테이너의 이름을 지정합니다.                                |
| `--detach`      | `-d`  |         | 컨테이너를 백그라운드에서 실행합니다.                        |
| `--env`         | `-e`  |         | 환경 변수를 설정합니다.                                      |
| `--env-file`    |       |         | 환경 변수를 저장한 파일을 설정합니다.                        |
| `--expose`      |       |         | 포트 또는 포트 범위를 노출합니다.                            |
| `--publish`     | `-p`  |         | 컨테이너의 포트를 공개합니다. (`-p HOST_PORT:CONTAINER_PORT`) |
| `--rm`          |       |         | 컨테이너가 종료되면 자동으로 삭제합니다.                     |
| `--interactive` | `-i`  |         | `STDIN`을 활성화합니다.                                      |
| `--tty`         | `-t`  |         | `pseudo-TTY`를 할당합니다.                                   |
| `--volume`      | `-v`  |         | 볼륨을 설정합니다.                                           |



#### Expose와 Publish 차이점

- `Expose`는 실제로 포트를 열지 않습니다. 일종의 문서 또는 가이드 역할로써 사용됩니다.
- `Publish`는 `host`의 포트와 바인드하여, `host` 또는 외부에서도 접근이 가능합니다.



**주의사항**

- `COMMAND`는 `Entrypoint`의 추가 인자로 활용됩니다.



#### [예시] `Ubuntu`에서 명령어 실행하기

```cmd
$ docker run ubuntu:22.04 /bin/bash -c "whoami && date"
root
Sun Dec 17 07:06:20 UTC 2023
```



#### [예시] 80 포트를 이용하여 `nginx` 서비스하기

```cmd
$ docker run -d -p 80:80 nginx:latest

# Result
$ CURL localhost:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```





### 컨테이너 목록 보기

> https://docs.docker.com/engine/reference/commandline/ps/

```shell
docker ps [OPTIONS]

# Include stopped container
docker ps -a
```



**주요 옵션**

| Option     | Short | Default | Description                                 |
| ---------- | ----- | ------- | ------------------------------------------- |
| `--all`    | `-a`  |         | 상태와 관계없이 모든 컨테이너를 표시합니다. |
| `--filter` | `-f`  |         | 지정된 조건에 맞는 컨테이너만 표시합니다.   |
| `--size`   | `-s`  |         | 전체 파일 사이즈도 표기합니다.              |



#### [예시] 종료된 컨테이너도 포함하여 목록 보기

```cmd
$ docker ps -a 
CONTAINER ID   IMAGE          COMMAND                   CREATED         STATUS                      PORTS     NAMES
803739f30047   nginx:latest   "/docker-entrypoint.…"   5 minutes ago   Exited (0) 17 seconds ago             stoic_diffie
```



#### [예시] 이름에 `server`가 들어간 컨테이너만 보기

```cmd
$ docker ps -a -f "name=server"
CONTAINER ID   IMAGE          COMMAND       CREATED      STATUS                     PORTS     NAMES
1e0d665497a7   ubuntu:22.04   "/bin/bash"   4 days ago   Exited (255) 3 hours ago             server3
939184ad2fe4   ubuntu:22.04   "/bin/bash"   4 days ago   Exited (255) 3 hours ago             server2
5c2817a20a00   ubuntu:22.04   "/bin/bash"   4 days ago   Exited (255) 3 hours ago             server
```



#### [예시] 컨테이너가 사용중인 볼륨(사이즈) 확인하기

```cmd
# 이미지를 실행한 직후
$ docker ps -s
CONTAINER ID   IMAGE          COMMAND       CREATED       STATUS       PORTS     NAMES                SIZE
a2306aa1de21   ubuntu:22.04   "/bin/bash"   3 hours ago   Up 3 hours             blissful_engelbart   0B (virtual 77.9MB)

# `apt-get update && apt-get upgrade` 실행 후
$ docker ps -s
CONTAINER ID   IMAGE          COMMAND       CREATED       STATUS       PORTS     NAMES                SIZE
a2306aa1de21   ubuntu:22.04   "/bin/bash"   3 hours ago   Up 3 hours             blissful_engelbart   46.8MB (virtual 125MB)
```





### 컨테이너 로그 확인하기

> https://docs.docker.com/engine/reference/commandline/logs/

```shell
docker logs [OPTIONS] <NAME_OR_ID>
```



**주요 옵션**

| Option         | Short | Default | Description                                                  |
| -------------- | ----- | ------- | ------------------------------------------------------------ |
| `--follow`     | `-f`  |         | 계속 로그를 표시합니다.                                      |
| `--tail`       | `-n`  | `all`   | 마지막 `n`개의 로그를 표시합니다.                            |
| `--timestamps` | `-t`  |         | `timestamp`를 표기합니다.                                    |
| `--since`      |       |         | 특정 시간 이후에 발생한 로그만 표시합니다. <br />- timestamp (e.g. `2013-01-02T13:23:37Z`) <br />- relative (e.g. `42m`, `10s`, ...) |
| `--until`      |       |         | 특정 시간 이전에 발생한 로그만 표시합니다. <br />- timestamp (e.g. `2013-01-02T13:23:37Z`) <br />- relative (e.g. `42m`, `10s`, ...) |



**주의사항**

- `docker logs`는 실제 로그 파일을 조회하는 것이 아닌, 출력 스트림(`STDOUT`/`STDERR`)를 조회합니다.



#### [예시] `nginx`의 마지막 로그 확인하기

`nginx` 컨테이너를 실행합니다.

```cmd
$ docker run --rm -d -p 80:80 --name nginx nginx:latest
```



다음 명령어를 통해 `nginx` 컨테이너에 API를 호출합니다.

```cmd
$ curl 127.0.0.1:80 
```



`nginx`의 마지막 1개 로그를 확인합니다.

```cmd
$ docker logs --tail 1 nginx
172.17.0.1 - - [21/Dec/2023:15:47:34 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/8.4.0" "-"
```





#### [예시] 특정 기간의 로그 확인하기

다음과 같이 매초마다 날짜를 출력하는 컨테이너를 실행합니다.

```cmd
$ docker run --name clock -d busybox sh -c "while true; do $(echo date); sleep 1; done"
```



현재 시간으로부터 `20초` 전부터 `10초`전까지 발생한 로그를 다음과 같이 확인합니다.

```cmd
$ docker logs -f --since 20s --until 10s clock && echo current time is %time%
Fri Dec 29 18:44:37 UTC 2023
Fri Dec 29 18:44:38 UTC 2023
Fri Dec 29 18:44:39 UTC 2023
Fri Dec 29 18:44:40 UTC 2023
Fri Dec 29 18:44:41 UTC 2023
Fri Dec 29 18:44:42 UTC 2023
Fri Dec 29 18:44:43 UTC 2023
Fri Dec 29 18:44:44 UTC 2023
Fri Dec 29 18:44:45 UTC 2023
Fri Dec 29 18:44:46 UTC 2023
current time is  3:44:57.61
```



### 컨테이너 접속하기

#### Attach

> https://docs.docker.com/engine/reference/commandline/attach/

```shell
docker attach [OPTIONS] CONTAINER
```

현재 실행중인 컨테이너의 터미널의 `STDIN`, `STDOUT`, `STDERR`에 접근합니다.



**주의사항**

- `Ctrl+C`를 통해 프로세스를 종료하게 되면 컨테이너도 종료됩니다.
- 컨테이너를 종료하지 않고 접속을 종료하기 위해서는 `Ctrl + P + Q`를 사용해야합니다.
- 일반적으로 사용하지 않습니다.



#### Exec

> https://docs.docker.com/engine/reference/commandline/exec/

```
docker exec -it [OPTIONS] <ID_OR_NAME> /bin/bash|sh|...
```

현재 실행중인 컨테이너에서 새로운 명령어를 실행합니다.



**주의사항**

- 메인 프로세스가 실행되는 동안에만 명령어를 실행할 수 있습니다.

- `exec`로 실행한 명령은 컨테이너 재실행(`start`)시 실행되지 않습니다.

- 일부 컨테이너는 `bash`, `sh`와 같은 `shell`이 없을 수 있습니다.
  - ex) `Scracth` 이미지, ...

- `-it` 옵션이 없다면 원격으로 명령어를 실행하고 프로세스가 종료됩니다.

```cmd
# `-it` 옵션이 없는 경우
$ docker exec clock sh -c "echo $$"
3031
$ docker exec clock sh -c "echo $$"
3039
$ docker exec clock sh -c "echo $$"
3047

# `-it` 옵션을 사용한 경우
$ docker exec -it clock sh
root@963c4fdec415:/ # echo $$
3221
root@963c4fdec415:/ # echo $$
3221
```



#### `Attach`와 `Exec` 차이점

|                | Attach                                 | Exec                      |
| -------------- | -------------------------------------- | ------------------------- |
| 접근 터미널    | 컨테이너의 `STDIN`, `STDOUT`, `STDERR` | 새로운 process의 Terminal |
| 접속 종료 방법 | `Ctrl + p + q`                         | `Ctrl + c`                |
| PID            | 1 (Main Process)                       | != 1                      |



#### [예시] `Attach`와 `Exec`를 각각 사용하여 `PID` 출력하기

다음 명령어를 사용하여 `ubuntu` 컨테이너를 실행합니다.

```cmd
$ docker run --rm -it -d --name server ubuntu:22.04
```



**Attach 사용한 경우**

`PID` 출력 시 메인 프로세스인 `1`이 출력됩니다.

```cmd
$ docker attach server
root@89834e856287:/# ps -aux
root@89834e856287:/# echo $$
1
```



**Exec 사용한 경우**

`PID` 출력 시 `1`이 아닌 다른 값이 출력되는 것을 볼 수 있습니다.

```cmd
$ docker exec -it server /bin/bash
root@89834e856287:/# echo $$
21
```





### 컨테이너 명령어 실행하기

> https://docs.docker.com/engine/reference/commandline/exec/

```shell
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```



**주의사항**

- 명령어는 `Working Directory`를 기준으로 실행되므로 파일 사용 시 주의가 필요합니다.

```cmd
$ docker exec clock sh -c "pwd"
/

$ docker exec --workdir /bin clock sh -c "pwd"
/bin
```



### 컨테이너 종료 & 시작

> https://docs.docker.com/engine/reference/commandline/stop/
>
> https://docs.docker.com/engine/reference/commandline/start/

```shell
# 종료
docker stop <NAME_OR_ID>
# 시작
docker start <NAME_OR_ID>
```



**주의사항**

- `Dockerfile`을 통해 실행되지 않은 프로세스들은 종료 후 재시작할 경우 살아나지 않습니다.

```cmd
$ docker run -d --name server ubuntu
$ docker exec -d server /bin/bash -c "while true; do $(echo date); sleep 1; done"
$ docker exec server ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 16:57 pts/0    00:00:00 /bin/bash
root         9     0  0 17:05 pts/1    00:00:00 /bin/sh
root       225     0  0 17:08 ?        00:00:00 /bin/bash -c while true; do $(echo date); sleep 1; done

$ docker stop server
$ docker start server
$ docker exec server ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 17:11 pts/0    00:00:00 /bin/bash
root         9     0  0 17:11 ?        00:00:00 ps -ef
```



### 컨테이너 중지 & 재실행

> https://docs.docker.com/engine/reference/commandline/pause/
>
> https://docs.docker.com/engine/reference/commandline/unpause/

```shell
# 중지
docker pause <NAME_OR_ID>
# 재실행
docker unpause <NAME_OR_ID>
```



**주의사항**

- 재실행하는 경우 정지되어 있던 `모든` 프로세스가 재실행됩니다.

```cmd
$ docker run -d --name server ubuntu
$ docker exec -d server /bin/bash -c "while true; do $(echo date); sleep 1; done"
$ docker exec server ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 16:57 pts/0    00:00:00 /bin/bash
root         9     0  0 17:05 pts/1    00:00:00 /bin/sh
root       225     0  0 17:08 ?        00:00:00 /bin/bash -c while true; do $(echo date); sleep 1; done

$ docker pause server
$ docker unpause server
$ docker exec server ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 16:57 pts/0    00:00:00 /bin/bash
root         9     0  0 17:05 pts/1    00:00:00 /bin/sh
root       225     0  0 17:08 ?        00:00:00 /bin/bash -c while true; do $(echo date); sleep 1; done
```



### 컨테이너 삭제

> https://docs.docker.com/engine/reference/commandline/rm/

```
docker rm [OPTIONS] CONTAINER [CONTAINER...]
```



**주요 옵션**

| Option      | Short | Default | Description                                                 |
| ----------- | ----- | ------- | ----------------------------------------------------------- |
| `--force`   | `-f`  |         | `SIGKILL` 시그널을 사용하여 실행중인 컨테이너를 삭제합니다. |
| `--volumes` | `-v`  |         | 컨테이너의 `anonymous volumes`도 함께 삭제합니다.           |



**Tip**

종료된 모든 컨테이너를 삭제하려면 다음과 같이 사용하면 됩니다.

> 로컬에서 학습용으로만 사용하세요

```cmd
$ docker rm $(docker ps -a -q -f status=exited)
```





### 컨테이너 리소스 사용량 조회

> https://docs.docker.com/engine/reference/commandline/stats/

```shell
docker stats [OPTIONS] [CONTAINER...]
```



**주요 옵션**

| Option        | Short | Default | Description                              |
| ------------- | ----- | ------- | ---------------------------------------- |
| `--all`       | `-a`  |         | 모든 컨테이너를 표시합니다.              |
| `--no-stream` |       |         | 결과를 지속적으로 업데이트하지 않습니다. |



#### [예시] `ubuntu` 컨테이너 리소스 사용량 조회하기

`ubuntu:22.04`를 이용하여 `server`를 실행합니다.

```cmd
$ docker run --rm -it -d --name server ubuntu:22.04
```



`server`가 사용중인 리소스는 다음과 같이 확인할 수 있습니다.

```cmd
$ docker stats --no-stream server
CONTAINER ID   NAME      CPU %     MEM USAGE / LIMIT   MEM %     NET I/O     BLOCK I/O   PIDS
5c2817a20a00   server    0.00%     896KiB / 15.56GiB   0.01%     586B / 0B   0B / 0B     1
```





---

### 연습 문제

#### [연습] Ubuntu 실행 및 Package 업데이트

> 조건
>
> - Ubuntu 이미지 Tag = 22.04
> - Container 이름 = server
> - 업데이트 이후에도 컨테이너는 실행중이어야 합니다.



**Ubuntu 실행하기**

```cmd
$ docker run -it -d --name server ubuntu:22.04
89834e856287c0875da2a9dddc5aa2905a8ad601f9348e014855badc9b345ccc
```



**Package 업데이트하기**

**방법 1 - Exec 사용하기**

- Interactive 모드

```cmd
$ docker exec -it server /bin/bash
root@89834e856287:/# apt-get update && apt-get ugrade
...
```

- Non-Interactive

```cmd
# Single Command
$ docker exec server apt-get update
Get:1 http://security.ubuntu.com/ubuntu jammy-security InRelease [110 kB]
...
$ docker exec server apt-get upgrade
Reading package lists...
Building dependency tree...
...

# Or Multiple Command
$ docker exec server /bin/bash -c "apt-get update && apt-get upgrade"
Get:1 http://security.ubuntu.com/ubuntu jammy-security InRelease [110 kB]
...
Reading package lists...
Building dependency tree...
...
```



**방법 2 - Attach 사용하기**

> Ubuntu 이미지의 경우 기본적으로 `/bin/bash`를 실행하므로 `attach`를 사용하여도 무방합니다.

```cmd
$ docker attach server
root@89834e856287:/# apt-get update && apt-get ugrade
...
```



**방법 3 - Container 실행시 Command 입력하기**

```cmd
$ docker run -it -d --name server ubuntu:22.04 /bin/bash -c "apt-get update && apt-get upgrade && /bin/bash"
Hit:1 http://security.ubuntu.com/ubuntu jammy-security InRelease
Hit:2 http://archive.ubuntu.com/ubuntu jammy InRelease
Hit:3 http://archive.ubuntu.com/ubuntu jammy-updates InRelease
Hit:4 http://archive.ubuntu.com/ubuntu jammy-backports InRelease
Reading package lists...
Reading package lists...
Building dependency tree...
Reading state information...
Calculating upgrade...
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
Done
```



#### [실습] PostgreSQL DB에 Table 생성하기

>조건
>
>- `postgres:16.1-bullseye` 이미지를 사용합니다.
>- 컨테이너의 이름은 `psql_db`로 설정합니다.
>- Container 생성시 다음 환경변수가 설정되어야 합니다.
   >  - POSTGRES_PASSWORD

1. `PostgreSQL` DB 컨테이너를 실행하세요
2. `exec`를 사용하여 컨테이너 터미널에 접근하세요
3. 터미널에서 `psql -U postgres`를 입력하여 `PostgreSQL`에 접속하세요
4. 다음 `Query`를 실행하세요

```sql
CREATE TABLE IF NOT EXISTS cloud_wave (
    id SERIAL PRIMARY KEY,
    timestamp timestamp
);
```

5. `\dt`를 실행하여 다음과 같이 생성한 테이블이 보이는지 확인하세요

```
postgres=# \dt
               List of relations
 Schema |        Name        | Type  |  Owner
--------+--------------------+-------+----------
 public |     cloud_wave     | table | postgres
(1 row)
```


