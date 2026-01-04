---
title: "CLI - Compose"
weight: 10
draft: false
---

## 2.1. Compose CLI

### 시작하기 전에

> https://docs.docker.com/compose/reference/

```
Usage:  docker compose [OPTIONS] COMMAND

Define and run multi-container applications with Docker.

Options:
      --ansi string                Control when to print ANSI control characters ("never"|"always"|"auto") (default "auto")
      --compatibility              Run compose in backward compatibility mode
      --env-file stringArray       Specify an alternate environment file
  -f, --file stringArray           Compose configuration files
      --parallel int               Control max parallelism, -1 for unlimited (default -1)
      --profile stringArray        Specify a profile to enable
      --project-directory string   Specify an alternate working directory
                                   (default: the path of the, first specified, Compose file)
  -p, --project-name string        Project name
```



**주요 옵션**

| Instruction    | Short | Description                           |
| :------------- | ----- | :------------------------------------ |
| `env-file`     |       | 환경 변수를 저장한 파일을 설정합니다. |
| `file`         | `-f`  | 사용할 `compose` 파일을 지정합니다.   |
| `profile`      |       | 사용할 프로파일을 지정합니다.         |
| `project-name` | `-p`  | 프로젝트 이름을 설정합니다.           |



#### 주의사항

- `-f`를 통해 사용할 `compose` 파일을 지정하지 않는 경우, `docker-compose.yml` 또는 `docker-compose.yaml` 파일이 사용됩니다.
- `Docker compose`파일의 `version`에 따라서 사용할 수 있는 기능과 변수명이 달라질 수 있습니다.
  따라서, 서버에 설치된 `Docker Engine`의 버젼과 호환되는지 체크하는 것이 좋습니다.
- 프로젝트 이름이 설정되지 않은 경우, 현재 디렉토리 이름이 프로젝트 이름으로 설정됩니다.





### `docker compose` 버젼 확인

> https://docs.docker.com/engine/reference/commandline/compose_version/

```
docker compose version [OPTIONS]
```

```cmd
$ docker compose version
Docker Compose version v2.18.1
```



### 프로젝트 실행

> https://docs.docker.com/engine/reference/commandline/compose_up/

```sh
docker compose up [OPTIONS] [SERVICE...]
```



**주요 옵션**

| Option                      | Short | Default | Description                                                  |
| --------------------------- | ----- | ------- | ------------------------------------------------------------ |
| `--abort-on-container-exit` |       |         | 1개 이상의 컨테이너가 멈춘(`stopped`) 경우, 모든 컨테이너를 종료합니다.<br />`-d`와는 함께 사용할 수 없습니다. |
| `--build`                   |       |         | 컨테이너 실행 전에 이미지를 빌드합니다.                      |
| `--detach`                  | `-d`  |         | 컨테이너를 백그라운드에서 실행시킵니다.                      |
| `--force-recreate`          |       |         | 변경 여부와 관계없이 모든 컨테이너를 재생성합니다.           |
| `--remove-orphans`          |       |         | `compose` 파일에 정의되어 있지 않은 서비스를 위한 컨테이너를 삭제합니다. |



**주의사항**

- 서로 다른 `docker-compose.yaml`을 **같은 프로젝트명(-p)** 으로 실행하면, 각 실행에서 생성된 리소스가 **동일한 프로젝트 네임스페이스** 아래에 묶여 함께 관리되는 것처럼 보일 수 있으며, 이로 인해 **리소스 충돌**이나 **orphan 컨테이너** 문제가 발생할 수 있습니다.
- 실행 과정에서 `.env`파일이 `docker-compose.yaml`파일을 해석 및 제어하는 과정에서 사용됩니다.
    - `docker-compose.yaml`내 `${VAR}` 같은 값들을 치환합니다.



#### [예시] 프로젝트 백그라운드로 실행하기

```cmd
$ docker compose up -d
[+] Building 0.0s (0/0)                                                                                      
[+] Running 6/6
 ✔ Network private                  Created                        0.0s
 ✔ Network db                       Created                        0.0s
 ✔ Volume "db-data"                 Created                        0.0s
 ✔ Container cloud_wave-frontend-1  Started                        0.6s
 ✔ Container cloud_wave-server-1    Started                        0.7s
 ✔ Container db_postgres            Started                        0.6s
```



#### [예시] `--abort-on-container-exit` 사용하기

```yaml
# docker-compose.yaml
version: '3.8'
name: 'cloudwave'

services:
  success:
    image: ubuntu:22.04
    entrypoint: /bin/bash
    command:
      - -c
      - sleep infinity
    networks:
      private: {}

  fail:
    image: ubuntu:22.04
    entrypoint: /bin/bash
    command:
      - -c
      - sleep fail
    networks:
      private: {}

networks:
  private:
```



`fail` 서비스를 위한 컨테이너가 정상적으로 실행되지 않으므로, 다음과 같이 프로젝트의 모든 컨테이너가 종료되는 것을 확인할 수 있습니다.

```cmd
$ docker compose up --abort-on-container-exit
[+] Building 0.0s (0/0)                                                                    docker:desktop-linux
[+] Running 2/0
 ✔ Container cloudwave-success-1  Created                                                                  0.0s 
 ✔ Container cloudwave-fail-1     Created                                                                  0.0s 
Attaching to cloudwave-fail-1, cloudwave-success-1
cloudwave-fail-1     | sleep: invalid time interval 'fail'
cloudwave-fail-1     | Try 'sleep --help' for more information.
cloudwave-fail-1 exited with code 1
Aborting on container exit...
[+] Stopping 2/2
 ✔ Container cloudwave-success-1  Stopped                                                                 10.1s 
 ✔ Container cloudwave-fail-1     Stopped                                                                  0.0s 
```





### 프로젝트 목록

> https://docs.docker.com/engine/reference/commandline/compose_ls/

```sh
docker compose ls [OPTIONS]
```



**주요 옵션**

| Option     | Default | Short | Description                                          |
| ---------- | ------- | ----- | ---------------------------------------------------- |
| `--all`    |         | `-a`  | 종료된 프로젝트를 포함한 모든 프로젝트를 표시합니다. |
| `--filter` |         |       | 필터 조건을 추가합니다.                              |
| `--quiet`  |         | `-q`  | 프로젝트의 이름만 표기합니다.                        |



#### [예시] 이름에 `cloud`가 포함된 모든 프로젝트 목록 보기

```cmd
$ docker compose ls --filter "name=cloud" -a
NAME               STATUS              CONFIG FILES
cloud_wave         running(3)          <PROJECT_PATH>/docker-compose.yaml
cloud_wave2        exited(2)           <PROJECT_PATH>/docker-compose.yaml
```



### 컨테이너 목록

> https://docs.docker.com/engine/reference/commandline/compose_ps/

```
docker compose ps [OPTIONS] [SERVICE...]
```



**주요 Options**

| Option       | Short | Default | Description                                                  |
| ------------ | ----- | ------- | ------------------------------------------------------------ |
| `--all`      | `-a`  |         | 종료된 컨테이너를 포함한 모든 컨테이너를 표시합니다.         |
| `--quiet`    | `-q`  |         | 컨테이너 ID만 표기합니다.                                    |
| `--services` |       |         | 서비스만 표기합니다.                                         |
| `--status`   |       |         | 상태값을 기반으로 서비스를 필터링합니다. <br />[paused \| restarting \| removing \| running \| dead \| created \| exited] |



**주의 사항**

- CLI를 통해 전달된 `project-name`또는  `configuration file`에 명시된 `name`을 기반으로 조회합니다.



#### [예시] 이름이 `cloud_wave`인 프로젝트의 실행중인 컨테이너 목록 보기

```cmd
$ docker compose -p cloud_wave ps
NAME                    IMAGE                    COMMAND                  SERVICE             CREATED             STATUS              PORTS
cloud_wave-frontend-1   nginx:latest             "/docker-entrypoint.…"   frontend            21 minutes ago      Up 21 minutes       0.0.0.0:80->80/tcp
cloud_wave-server-1     ubuntu:22.04             "/bin/bash -c 'sleep…"   server              21 minutes ago      Up 21 minutes       
db_postgres             postgres:16.1-bullseye   "docker-entrypoint.s…"   postgres            19 minutes ago      Up 19 minutes       5432/tcp
```



### 이미지 목록

> https://docs.docker.com/engine/reference/commandline/compose_images/

```
docker compose images [OPTIONS] [SERVICE...]
```



**주의 사항**

- CLI를 통해 전달된 `project-name`또는  `configuration file`에 명시된 `name`을 기반으로 조회합니다.



#### [예시] 이름이 `cloud_wave`인 프로젝트의 이미지 목록 보기

```cmd
$ docker compose images
CONTAINER               REPOSITORY          TAG                 IMAGE ID            SIZE
cloud_wave-frontend-1   nginx               latest              d453dd892d93        187MB
cloud_wave-server-1     ubuntu              22.04               174c8c134b2a        77.9MB
db_postgres             postgres            16.1-bullseye       aab92def9ff2        384MB
```



### 프로젝트 삭제

> https://docs.docker.com/engine/reference/commandline/compose_down/

```sh
docker compose down [OPTIONS] [SERVICES]
```



**주요 옵션**

| Option             | Short | Default | Description                                                  |
| ------------------ | ----- | ------- | ------------------------------------------------------------ |
| `--remove-orphans` |       |         | `Compose` 파일에 정의되지 않은 서비스를 구성하는 컨테이너도 삭제합니다. |
| `--volumes`        | `-v`  |         | `Copose` 파일에 정의된 `anonymous` & `named` 볼륨을 같이 삭제합니다. |



**주의 사항**

- 현재 디렉토리에 `docker-compose.yaml` 파일이 존재하지 않는 경우, `-p` 또는 `-f`를 이용하여야 합니다.
- `--remove-orphans`를 사용하지 않을 경우, `compose` 파일에 정의된 서비스를 구성하는 컨테이너만 삭제합니다.
- `external`로 정의된 `network`와 `volume`은 삭제되지 않습니다.
- 다른 프로젝트에서 사용중인 리소스(`volume`, `network`, ...)는 삭제되지 않습니다.



#### [예시] 이름이 `cloud_wave`인 프로젝트 삭제하기

```cmd
$ docker compose -p cloud_wave down
[+] Running 3/3
 ✔ Container cloud_wave-postgres-1  Removed                                                               0.1s 
 ✔ Container cloud_wave-frontend-1  Removed                                                               0.0s 
 ✔ Container cloud_wave-server-1    Removed                                                               10.1s 
```



#### [예시] 프로젝트 삭제시 `volume`도 함께 제거하기

```yaml
# docker-compose.yaml
version: '3.8'
name: 'cloud_wave'

services:
  success:
    image: ubuntu:22.04
    entrypoint: /bin/bash
    command:
      - -c
      - sleep infinity
    networks:
      private: {}
    volumes:
      - anonymous:/anonymous
      - named:/named
      - external:/external

networks:
  private:

volumes:
  anonymous:
  named:
    name: "named_volume"
  external:
    name: "external"
    external: true
```



`--volumes`을 사용하는 경우, 다음과 같이 `external`로 설정된 볼륨을 제외한 나머지 2개 볼륨이 함께 삭제되는 것을 확인할 수 있습니다.

```cmd
$ docker compose -p cloud_wave down --volumes
[+] Running 5/5
 ✔ Container cloud_wave-fail-1     Removed                                                                 0.0s 
 ✔ Container cloud_wave-success-1  Removed                                                                10.1s 
 ✔ Volume named_volume             Removed                                                                 0.0s 
 ✔ Volume cloud_wave_anonymous     Removed                                                                 0.0s 
 ✔ Network cloud_wave_private      Removed                                                                 0.1s 
```



### 재실행

> https://docs.docker.com/engine/reference/commandline/compose_restart/

```shell
docker compose restart [OPTIONS] [SERVICE...]
```



**주의 사항**

- 서비스를 명시하지 않은 경우, 모든 서비스에 대해서 종료된 컨테이너를 포함하여 재실행됩니다.
- 반영되지 않은 변경사항이 `compose` 파일에 존재하더라도, 해당 변경사항은 **반영되지 않습니다.**



#### [예시]  `cloud_wave` 프로젝트의 모든 서비스 재실행하기

```cmd
$ docker compose -p cloud_wave restart
[+] Restarting 3/3
 ✔ Container cloud_wave-frontend-1  Started                                                                0.5s 
 ✔ Container db_postgres            Started                                                                0.5s 
 ✔ Container cloud_wave-server-1    Started                                                                0.3s
```



#### [예시] `cloud_wave` 프로젝트의 특정 서비스만 재실행하기

```cmd
$ docker compose -p cloud_wave restart server
[+] Restarting 1/1
 ✔ Container cloud_wave-server-1  Started                                                                    0.1s 
```





### 종료 & 시작

> https://docs.docker.com/engine/reference/commandline/compose_stop/
>
> https://docs.docker.com/engine/reference/commandline/compose_start/

```shell
# 종료
docker compose stop [OPTIONS] [SERVICE...]
# 시작
docker compose start [SERVICE...]
```



#### [예시]  `cloud_wave` 프로젝트 종료하기

```cmd
$  docker compose -p cloud_wave stop
[+] Stopping 2/3
 ✔ Container cloud_wave-postgres-1  Stopped                                                                0.1s  
 ✔ Container cloud_wave-server-1    Stopped                                                               10.2s 
 ✔ Container cloud_wave-frontend-1  Stopped                                                                0.1s 
```



#### [예시]  `cloud_wave` 프로젝트 실행하기

```cmd
$ docker compose -p cloud_wave start
[+] Running 3/3
 ✔ Container cloud_wave-frontend-1  Started                                                                 0.6s 
 ✔ Container cloud_wave-postgres-1  Started                                                                 0.4s 
 ✔ Container cloud_wave-server-1    Started                                                                 0.5s 
```



### 중지 & 실행

> https://docs.docker.com/engine/reference/commandline/compose_pause/
>
> https://docs.docker.com/engine/reference/commandline/compose_unpause/

```shell
# 중지
docker compose pause [OPTIONS] [SERVICE...]
# 실행
docker compose unpause [SERVICE...]
```



#### [예시]  `cloud_wave` 프로젝트 중지하기

```cmd
$  docker compose -p cloud_wave pause
[+] Pausing 2/0
 ✔ Container cloud_wave-frontend-1  Paused                                                                 0.0s 
 ✔ Container db_postgres            Paused                                                                 0.0s 
```

#### [예시]  `cloud_wave` 프로젝트 실행하기

```cmd
$ docker compose -p cloud_wave unpause
[+] Running 2/0
 ✔ Container db_postgres            Unpaused                                                               0.0s 
 ✔ Container cloud_wave-frontend-1  Unpaused                                                               0.0s 
```



### 서비스 빌드하기

> https://docs.docker.com/engine/reference/commandline/compose_build/

```shell
docker compose build [OPTIONS] [SERVICE...]
```



**주요  옵션**

| Option        | Short | Default | Description                                |
| ------------- | ----- | ------- | ------------------------------------------ |
| `--build-arg` |       |         | 빌드시에 사용할 `ARG`를 설정합니다.        |
| `--builder`   |       |         | 사용할 `builder`를 설정합니다.             |
| `--no-cache`  |       |         | 이미지 빌드시 `Cache`를 사용하지 않습니다. |
| `--pull`      |       |         | 항상 이미지를 새로 다운로드 받습니다.      |
| `--push`      |       |         | 서비스의 이미지를 `Push` 합니다.           |



**주의 사항**

- `yaml` 파일에 명시된 서비스의 `image`를 이름으로 한 이미지가 생성됩니다.



#### [예시] 프로젝트를 위한 이미지 빌드하기

```yaml
# docker-compose.yaml
version: "3.8"

services:
  server:
    image: compose:build.v1
    build:
      dockerfile_inline: |
        FROM ubuntu:22.04
        RUN apt-get update && apt-get upgrade
      # Or Use `dockerfile`
      # dockerfile: "server.dockerfile"
    entrypoint: /bin/bash
    command:
      - -c
      - "sleep 3600"
    restart: no
```



다음과 같이 `server` 서비스의 `image`로 명시된 `compose:build.v1`로 명명된 것을 확인할 수 있습니다.

```cmd
$ docker compose build
[+] Building 0.3s (6/6) FINISHED                                                                                
 => [server internal] load build definition from Dockerfile                                                0.0s
 => => transferring dockerfile: 92B                                                                        0.0s
 => [server internal] load .dockerignore                                                                   0.0s
 => => transferring context: 2B                                                                            0.0s
 => [server internal] load metadata for docker.io/library/ubuntu:22.04                                     0.0s
 => [server 1/2] FROM docker.io/library/ubuntu:22.04                                                       0.0s
 => [server 2/2] RUN apt-get update && apt-get upgrade                                                     0.2s
 => [server] exporting to image                                                                            0.0s
 => => exporting layers                                                                                    0.0s
 => => writing image sha256:d30066830e7283c6f7c777b698277a666934dfdfc279fca4cd847be7157aaa1d               0.0s
 => => naming to docker.io/library/compose:build.v1                                                        0.0s
```





### 프로젝트 로그 확인하기

> https://docs.docker.com/engine/reference/commandline/compose_logs/

```
docker compose logs [OPTIONS] [SERVICE...]
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

- `--tail`은 전체가 아닌 각 컨테이너별 로그 수를 의미합니다.



#### [예시] `cloud_wave` 프로젝트의 모든 서비스의 로그를 2개 출력하기

```cmd
$ docker compose -p cloud_wave logs -n 2
cloud_wave-frontend-1  | 2023/12/27 08:28:52 [notice] 1#1: start worker process 23
cloud_wave-frontend-1  | 2023/12/27 08:28:52 [notice] 1#1: start worker process 24
db_postgres            | 2023-12-27 08:28:52.215 UTC [29] LOG:  database system was shut down at 2023-12-27 08:28:51 UTC
db_postgres            | 2023-12-27 08:28:52.219 UTC [1] LOG:  database system is ready to accept connections
```



### 프로세스 확인하기

> https://docs.docker.com/engine/reference/commandline/compose_top/

```shell
docker compose top [SERVICES...]
```



#### [예시] `cloud_wave` 프로젝트의 컨테이너별 프로세스 확인하기

```cmd
$ docker compose -p cloud_wave top              
cloud_wave-frontend-1
UID     PID     PPID    C    STIME   TTY   TIME       CMD
root    38700   38649   0    08:58   ?     00:00:00   nginx: master process nginx -g daemon off;   
uuidd   38783   38700   0    08:58   ?     00:00:00   nginx: worker process                        
uuidd   38784   38700   0    08:58   ?     00:00:00   nginx: worker process                        
uuidd   38785   38700   0    08:58   ?     00:00:00   nginx: worker process                        

cloud_wave-server-1
UID    PID     PPID    C    STIME   TTY   TIME       CMD
root   38513   38472   0    08:58   ?     00:00:00   sleep 3600   

db_postgres
UID   PID     PPID    C    STIME   TTY   TIME       CMD
999   38671   38622   0    08:58   ?     00:00:00   postgres                                 
999   38786   38671   0    08:58   ?     00:00:00   postgres: checkpointer                   
999   38787   38671   0    08:58   ?     00:00:00   postgres: background writer              
999   38789   38671   0    08:58   ?     00:00:00   postgres: walwriter                      
999   38790   38671   0    08:58   ?     00:00:00   postgres: autovacuum launcher            
999   38791   38671   0    08:58   ?     00:00:00   postgres: logical replication launcher  
```



### Config 확인

> https://docs.docker.com/engine/reference/commandline/compose_config/

```shell
docker compose config [OPTIONS] [SERVICE...]
```



#### [예시] `cloud_wave`의 `config` 확인하기

```cmd
$ docker compose -p cloud_wave config    
name: cloud_wave
services:
  frontend:
    configs:
    - source: server-config
    image: nginx:latest
    networks:
      private: null
    ports:
    - mode: ingress
      target: 80
      published: "80"
      protocol: tcp
    restart: always
  ...
networks:
	...
volumes:
  db-data:
    name: db-data
    labels:
      description: PostgreSQL 16.1 volume
secrets:
	...
configs:
  ...
```



### 종료한 서비스 삭제

> https://docs.docker.com/engine/reference/commandline/compose_rm/

```shell
docker compose rm [OPTIONS] [SERVICE...]
```

**주요 Options**

| Option      | Short | Default | Description                                         |
| ----------- | ----- | ------- | --------------------------------------------------- |
| `--force`   | `-f`  |         | Don't ask to confirm removal                        |
| `--stop`    | `-s`  |         | Stop the containers, if required, before removing   |
| `--volumes` | `-v`  |         | Remove any anonymous volumes attached to containers |



#### [예시] 프로젝트 삭제하기

```yaml
version: '3.8'
name: 'cloud_wave'

services:
  success:
    image: ubuntu:22.04
    entrypoint: /bin/bash
    command:
      - -c
      - sleep infinity
    networks:
      private: {}
  fail:
    image: ubuntu:22.04
    entrypoint: /bin/bash
    command:
      - -c
      - sleep fail
    networks:
      private: {}

networks:
  private:
```



프로젝트의 모든 컨테이너 목록을 다음과 같이 확인합니다.

```cmd
$ docker compose -p cloud_wave ps -a
NAME                   IMAGE          COMMAND                   SERVICE   CREATED         STATUS                     PORTS
cloud_wave-fail-1      ubuntu:22.04   "/bin/bash -c 'sleep…"   fail      6 seconds ago   Exited (1) 6 seconds ago   
cloud_wave-success-1   ubuntu:22.04   "/bin/bash -c 'sleep…"   success   6 seconds ago   Up 6 seconds       
```



해당 프로젝트를 삭제하면 다음과 같이 종료된 컨테이너만 삭제되는 것을 볼 수 있습니다.

```cmd
$ docker compose -p cloud_wave rm
? Going to remove cloud_wave-fail-1 Yes
[+] Removing 1/0
 ✔ Container cloud_wave-fail-1  Removed                                                                    0.0s 
```



#### [예시] 프로젝트 종료 후 삭제하기

`-s` 또는 `--stop`을 이용하여 프로젝트를 삭제하면, 다음과 같이 삭제 전 컨테이너를 종료하는 것을 확인할 수 있습니다.

```cmd
$ docker compose -p cloud_wave rm -s
[+] Stopping 2/2
 ✔ Container cloud_wave-fail-1     Stopped                                                                 0.0s 
 ✔ Container cloud_wave-success-1  Stopped                                                                10.1s 
? Going to remove cloud_wave-fail-1, cloud_wave-success-1 Yes
[+] Removing 2/0
 ✔ Container cloud_wave-success-1  Removed                                                                 0.0s 
 ✔ Container cloud_wave-fail-1     Removed                                                                 0.0s 
```



### 연습문제

#### [연습] 프로젝트 실행 및 `Network` 확인하기

다음 파일을 `example.yaml`로 저장합니다.

```yaml
version: '3.8'

services:
  ubuntu:
    image: ubuntu:22.04
    entrypoint: /bin/bash
    command:
      - -c
      - "apt-get update && apt-get upgrade && apt-get install -y curl && sleep 3600"
    restart: no

  nginx:
    image: nginx:latest
    expose:
      - 80
    restart: always
```



명령어를 이용하여 `project`를 실행합니다.

```cmd
$ docker compose -f example_1.yaml -p ex1 up -d
[+] Building 0.0s (0/0) 
[+] Running 3/3
 ✔ Network ex1_default     Created 
 ✔ Container ex1-nginx-1   Started 
 ✔ Container ex1-ubuntu-1  Started   
```



다음과 같이 `ex1_default`라는 네트워크가 새로 생성된 것을 확인할 수 있습니다.

```cmd
$ docker network ls
NETWORK ID     NAME             DRIVER    SCOPE
50f36ebc860b   bridge           bridge    local
5480e9b993ea   ex1_default      bridge    local
```



다음과 같이 생성된 `ex1-ubuntu-1` 컨테이너의 정보를 조회하면 `docker run`을 통해서 실행하였을 때와 다른점을 확인해볼 수 있습니다.

- `default` 네트워크가 아닌 `ex1_default` 네트워크만 할당되어 있습니다.
- `service`의 이름과 컨테이너의 이름이 `alias`로 등록되어 있는 것을 확인할 수 있습니다.

```cmd
$ docker inspect ex1-ubuntu-1 -f "{{ println .NetworkSettings.Networks }}"
map[ex1_default:0xc000000240]

$ docker inspect ex1-ubuntu-1 -f "Alias:{{ println .NetworkSettings.Networks.ex1_default.Aliases }}IP:{{ println .NetworkSettings.Networks.ex1_default.IPAddress }}"
Alias:[ex1-ubuntu-1 ubuntu f5cfe44b6bee]
IP:172.21.0.3
```


실제로 `ubuntu` 서버에서 다음과 같이 `curl` 명령어를 사용하면, 정상적으로 `nginx` 서버에서 결과를 얻어 오는 것을 확인할 수 있습니다.

```cmd
$ curl nginx:80
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



#### [연습] 프로젝트 업데이트 - 서비스 포트 추가하기

위에서 생성한 `ex1` 프로젝트의 `nginx` 서비스는 외부에 포트가 열려있지 않아 다음과 같이 `host`에서 접근할 수 없는 상태입니다.

```cmd
$ curl localhost:80
curl: (7) Failed to connect to localhost port 80 after 2237 ms: Couldn't connect to server
```



`host`에서도 접근할 수 있도록, 다음과 같이 `example.yaml`를 수정합니다.

```yaml
# exaple.yaml
version: '3.8'

services:
  ubuntu:
    image: ubuntu:22.04
    entrypoint: /bin/bash
    command:
      - -c
      - "apt-get update && apt-get upgrade && apt-get install -y curl && sleep 3600"
    restart: no

  nginx:
    image: nginx:latest
    # expose 대신 ports를 사용합니다. 
    ports:
      - 80:80
    restart: always
```



다음 명령어를 이용하여 프로젝트를 업데이트 합니다.

```cmd
$ docker compose -f example.yaml -p ex1 up -d
```



다시 `host`에서 접근을 시도하면 다음과 같이 정상적으로 접근이 되는 것을 확인할 수 있습니다.

```cmd
$ curl localhost:80
```



#### [연습] 이미지 빌드 후 프로젝트 실행하기

`server.dockerfile`와 `docker-compose.yaml`파일을 다음과 같이 작성하고 저장합니다.

`폴더 구조`

```tex
.
├── Dockerfile
└── docker-compose.yaml
```

`server.dockerfile`

```dockerfile
FROM ubuntu:22.04
RUN apt-get update && apt-get upgrade
```

`docker-compose.yaml`

```yaml
version: '3.8'

services:
  ubuntu:
    image: cloudwave:example.1
    build:
      dockerfile: "server.dockerfile"
    entrypoint: /bin/bash
    command:
      - -c
      - "sleep 3600"
    restart: no

  nginx:
    image: nginx:latest
    ports:
      - "80:80"
    restart: always
```



`--build` 옵션을 이용하여 다음과 같이 프로젝트를 실행합니다.

```cmd
$ docker compose -p cloud_wave up -d --build

[+] Building 0.0s (6/6) FINISHED                                                           docker:desktop-linux
 => [ubuntu internal] load .dockerignore                                                                   0.0s
 => => transferring context: 2B                                                                            0.0s
 => [ubuntu internal] load build definition from server.dockerfile                                         0.0s
 => => transferring dockerfile: 99B                                                                        0.0s
 => [ubuntu internal] load metadata for docker.io/library/ubuntu:22.04                                     0.0s
 => [ubuntu 1/2] FROM docker.io/library/ubuntu:22.04                                                       0.0s
 => CACHED [ubuntu 2/2] RUN apt-get update && apt-get upgrade                                              0.0s
 => [ubuntu] exporting to image                                                                            0.0s
 => => exporting layers                                                                                    0.0s
 => => writing image sha256:9fe790212381bd4b76c35994d028ce911c774b9719577f36ffbc6e8817eefe28               0.0s
 => => naming to docker.io/library/cloudwave:example.1                                                     0.0s
[+] Running 3/3
 ✔ Network cloud_wave_default     Created                                                                  0.0s 
 ✔ Container cloud_wave-nginx-1   Started                                                                  0.0s 
 ✔ Container cloud_wave-ubuntu-1  Started                                                                  0.0s 
```



`docker images` 또는 `docker compose images`를 통해 생성된 이미지를 확인해볼 수 있습니다.

```cmd
$ docker compose -p cloud_wave images ubuntu
CONTAINER             REPOSITORY          TAG                 IMAGE ID            SIZE
cloud_wave-ubuntu-1   cloudwave           example.1           9fe790212381        125MB
```



#### [연습] 서비스별로 마지막 5개 로그 출력 후 `follow`하기

```cmd
$ docker compose logs -n 5 -f
ex1-ubuntu-1  | Processing triggers for ca-certificates (20230311ubuntu0.22.04.1) ...
ex1-nginx-1   | 2023/12/25 16:17:36 [notice] 1#1: start worker process 41
ex1-nginx-1   | 2023/12/25 16:17:36 [notice] 1#1: start worker process 42
ex1-nginx-1   | 2023/12/25 16:17:36 [notice] 1#1: start worker process 43
ex1-nginx-1   | 2023/12/25 16:17:36 [notice] 1#1: start worker process 44
ex1-ubuntu-1  | Updating certificates in /etc/ssl/certs...
ex1-ubuntu-1  | 0 added, 0 removed; done.
ex1-ubuntu-1  | Running hooks in /etc/ca-certificates/update.d...
ex1-ubuntu-1  | done.
ex1-nginx-1   | 172.21.0.3 - - [25/Dec/2023:16:27:38 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.81.0" "-"
...
```



#### [연습] 프로젝트 삭제하기

다음 `docker-compose.yaml`파일을 이용하여 프로젝트를 실행합니다.

```yaml
# docker-compose.yaml
version: "3.8"
name: "cloud_wave"

services:
  frontend:
    image: nginx:latest
    ports:
      - "80:80"
    networks:
      - private
    restart: always

  server:
    image: ubuntu:22.04
    entrypoint: /bin/bash
    command:
      - -c
      - "sleep 3600"
    networks:
      - private
      - db
    restart: no

  postgres:
    container_name: db_postgres
    image: postgres:16.1-bullseye
    networks:
      - db
    expose:
      - 5432
    volumes:
      - db-data:/var/lib/postgresql/data:rw
    env_file:
      - .env
    restart: always


volumes:
  db-data:
    name: "db-data"
    labels:
      description: "PostgreSQL 16.1 volume"

networks:
  private:
    name: "private"
  db:
    name: "db"
```



```cmd
$ docker compose up -d
[+] Building 0.0s (0/0)                                                                    docker:desktop-linux
[+] Running 6/6
 ✔ Network db                       Created                                                                0.0s 
 ✔ Network private                  Created                                                                0.0s 
 ✔ Volume "db-data"                 Created                                                                0.0s 
 ✔ Container db_postgres            Started                                                                0.0s 
 ✔ Container cloud_wave-frontend-1  Started                                                                0.0s 
 ✔ Container cloud_wave-server-1    Started                                                                0.0s 
```



`docker-compose.yaml`에서 `frontend` 서비스를 제거합니다.

```yaml
# docker-compose.yaml
version: "3.8"
name: "cloud_wave"

services:
  server:
    image: ubuntu:22.04
    entrypoint: /bin/bash
    command:
      - -c
      - "sleep 3600"
    networks:
      - private
      - db
    restart: no

  postgres:
    container_name: db_postgres
    image: postgres:16.1-bullseye
    networks:
      - db
    expose:
      - 5432
    volumes:
      - db-data:/var/lib/postgresql/data:rw
    env_file:
      - .env
    restart: always


volumes:
  db-data:
    name: "db-data"
    labels:
      description: "PostgreSQL 16.1 volume"

networks:
  private:
    name: "private"
  db:
    name: "db"
```

이후, 다음과 같이 삭제를 시도하면 `frontend` 서비스와  `volume`을 제외한 나머지 컴포넌트에 대해서만 삭제를 시도하는 것을 확인할 수 있습니다.

```cmd
$ docker compose -p cloud_wave down
[+] Running 4/4
 ✔ Container db_postgres          Removed                                                                  0.1s 
 ✔ Container cloud_wave-server-1  Removed                                                                 10.2s 
 ! Network private                Resource is still in use                                                 0.0s 
 ✔ Network db                     Removed                                                                  0.0s 
```



다음과 같이 `docker-compose.yaml`에서 제거된  `frontend` 서비스가 살아있는 것을 확인할 수 있습니다.

```cmd
$ docker compose ps 
NAME                    IMAGE          COMMAND                   SERVICE    CREATED         STATUS         PORTS
cloud_wave-frontend-1   nginx:latest   "/docker-entrypoint.…"   frontend   5 minutes ago   Up 5 minutes   0.0.0.0:80->80/tcp
```



다음과 같이  `--volumes`와 `--remove-orphans` 옵션을 이용하면, 남아있는 컨테이너와 볼륨을 삭제할 수 있습니다.

```cmd
$ docker compose down --volumes --remove-orphans                
[+] Running 3/3
 ✔ Container cloud_wave-frontend-1  Removed                                                                0.2s 
 ✔ Volume db-data                   Removed                                                                0.0s 
 ✔ Network private                  Removed                                                                0.1s 
```



