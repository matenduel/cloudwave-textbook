---
title: "CLI - Build & Dockerfile"
weight: 42
draft: false
---

## 1.3. Docker 이미지 생성하기

### 시작하기에 앞서

- 이미지는 컨테이너를 실행하기 위한 필요한 구성요소가 담겨있는 완성된 패키지입니다.
- 일반적으로 모든 수정 사항은 `Dockerfile`을 수정하여 새로운 이미지를 제작해 반영합니다.



### 이미지 생성하기

> https://docs.docker.com/engine/reference/commandline/build/

```cmd
# docker build [OPTIONS] PATH | URL | - [-f <PATH_TO_FILE>]
$ docker build . [-f <PATH_TO_FILE>]

# or
$ docker buildx build [OPTIONS] PATH | URL | - [-f <PATH_TO_FILE>]
```



**주요 옵션**

| Option        | Short | Default | Description                                        |
| ------------- | ----- | ------- | -------------------------------------------------- |
| `--build-arg` |       |         | `ARG`를 설정합니다.                                |
| `--file`      | `-f`  |         | `Dockerfile`의 경로를 지정합니다.                  |
| `--label`     |       |         | 라벨을 추가합니다.                                 |
| `--no-cache`  |       |         | 이미지 빌드시 캐시를 사용하지 않습니다.            |
| `--platform`  |       |         | `platform`을 지정합니다.                           |
| `--pull`      |       |         | 관련된 이미지를 저장 유무에 관계없이 `pull`합니다. |
| `--tag`       | `-t`  |         | 이름과 `Tag`를 설정합니다.                         |



### Dockerfile

| Instruction                                                  | Description                               |
| :----------------------------------------------------------- | :---------------------------------------- |
| [`FROM`](https://docs.docker.com/engine/reference/builder/#from) | 사용할 `Base` 이미지를 설정합니다.        |
| [`ARG`](https://docs.docker.com/engine/reference/builder/#arg) | `build`에 사용할 변수를 설정합니다.       |
| [`ENV`](https://docs.docker.com/engine/reference/builder/#env) | 환경 변수를 설정합니다.                   |
| [`ADD`](https://docs.docker.com/engine/reference/builder/#add) | 파일 또는 폴더(`directory`)를 추가합니다. |
| [`COPY`](https://docs.docker.com/engine/reference/builder/#copy) | 파일 또는 폴더(`directory`)를 복사합니다. |
| [`LABEL`](https://docs.docker.com/engine/reference/builder/#label) | 라벨을 추가합니다.                        |
| [`EXPOSE`](https://docs.docker.com/engine/reference/builder/#expose) | 포트를 외부에 노출합니다.                 |
| [`USER`](https://docs.docker.com/engine/reference/builder/#user) | `User`와 `Group`을 설정합니다.            |
| [`WORKDIR`](https://docs.docker.com/engine/reference/builder/#workdir) | `Working Directory`를 변경합니다.         |
| [`RUN`](https://docs.docker.com/engine/reference/builder/#run) | 명령어를 실행합니다.                      |
| [`CMD`](https://docs.docker.com/engine/reference/builder/#cmd) | 기본 명령어를 정의합니다.                 |
| [`ENTRYPOINT`](https://docs.docker.com/engine/reference/builder/#entrypoint) | 필수로 실행할 명령어를 정의합니다.        |



#### 주의사항

- `-f`를 통해 사용할 `Dockerfile`을 명시하지 않는 경우, 파일 이름이 `Dockerfile`인 파일이 사용됩니다.
- `Dockerfile`내 모든 상대 경로들은 `build`시 입력된 `Path(Context)`를 기준으로 계산됩니다.
- `cache`를 사용할 경우, `apt-get`을 통해 설치한 패키지 버젼이 최신이 아닐 수 있습니다.



#### 많이 사용하는 `base` 이미지

##### - Scratch 이미지

> https://hub.docker.com/_/scratch

`binary`를 실행하기 위한 최소한의 이미지입니다.
`Dockerfile`에서 사용했을 시, 별도의 `layer`가 추가되지 않습니다.



##### - Alpine linux

> https://hub.docker.com/_/alpine

`Alpine` 리눅스는 용량이 5Mb 이하로 매우 가벼우며 보안성이 뛰어납니다.
많은 이미지들이 `alpine`을 기반으로한 경량화된 이미지를 함께 제공합니다.



##### - Distroless 이미지

> https://github.com/GoogleContainerTools/distroless

`Distroless` 이미지는 애플리케이션 실행에 필요한 런타임 종속성만 포함되어있습니다.
`bash` 와 같은 셸도 포함하고 있지 않아 보안성이 매우 뛰어납니다.



### Multi-stage builds

> https://docs.docker.com/build/building/multi-stage/

일반적으로 어플리케이션을 실행하기 위한 환경(`runtime dependencies`)과 빌드용 환경(`build-time dependencies`)은 같지 않습니다. 따라서 `Production` 환경에서 보안성 증대 및 이미지 경량화등을 위해 `Multi-stage build`를 활용하여 어플리케이션 코드를 `build`한 이후, `runtime`을 위한 이미지로 옮겨 사용하는 경우가 많습니다.



#### 사용 예시

- 보안 강화 및 이미지 경량화를 위해 빌드용 이미지와 배포용 이미지를 따로 사용하는 경우

```dockerfile
# 'golang:1.19-alpine' 이미지를 이용하여 Go Application을 컴파일 합니다. 
FROM golang:1.19-alpine as build
WORKDIR /app
COPY src ./
RUN CGO_ENABLED=0 go build -o main

---
# 위에서 컴파일한 소스코드만 'Scratch' 이미지로 가져옵니다. 
FROM scratch as release
COPY --from=build /app/main /app/
WORKDIR /app
CMD ["/app/main"]
```



- 필요한 `Package` 또는 파일을 다른 이미지에서 가져오는 경우

```dockerfile
# 'nginx:latest' 이미지에서 'nginx.conf' 파일을 가져옵니다. 
COPY --from=nginx:latest /etc/nginx/nginx.conf /nginx.conf
```





---

### 연습문제

#### [연습] Go 서버 이미지 제작하기

> 다음 3개 이미지를 기반으로 이미지를 빌드합니다.
>
> - Ubuntu
> - Alpine
> - Scratch

전체적인 폴더 구조는 다음과 같습니다.

```
.
├── src
│   ├── go.mod
│   └── main.go
└── Dockerfile
```

**go.mod**

```go
module example/hello
go 1.19
```

**main.go**

```go
package main

import (
    "io"
    "net/http"
    "log"
)

func HelloServer(w http.ResponseWriter, req *http.Request) {
    io.WriteString(w, "Hello, Worlds!\n")
}

func main() {
    http.HandleFunc("/", HelloServer)
    log.Fatal(http.ListenAndServe(":80", nil))
}

```

##### Debian (bullseye)을 이용한 Dockerfile

```dockerfile
FROM golang:1.19-bullseye

WORKDIR /app

COPY src ./

RUN CGO_ENABLED=0 go build -o main

CMD ["/app/main"]
```

##### Alpine을 이용한 Dockerfile

```dockerfile
FROM golang:1.19-alpine

WORKDIR /app

COPY src ./

RUN CGO_ENABLED=0 go build -o main

CMD ["/app/main"]
```

##### Scratch를 이용한 Dockerfile

```dockerfile
FROM golang:1.19-alpine as build
WORKDIR /app
COPY src ./
RUN CGO_ENABLED=0 go build -o main

FROM scratch as release
COPY --from=build /app/main /app/
WORKDIR /app
CMD ["/app/main"]
```



`Dockerfile`들을 이용하여 이미지를 생성합니다.

``` cmd
$ docker build -t go:<TAG_NAME> .
```



생성된 이미지는 다음과 같이 실행하여 확인해볼 수 있습니다.

```cmd
$ docker run -d -p 80:80 go:scratch
$ curl localhost:80
Hello, Worlds!
```



각 이미지별 크기는 다음과 같이 확인할 수 있습니다.

```cmd
$ docker images go
REPOSITORY   TAG       IMAGE ID       CREATED              SIZE
go           debian    7b5f70dd7edf   About a minute ago   1.02GB
go           alpine    ac6365a46643   6 minutes ago        378MB
go           scratch   97bcb5fd82c5   26 seconds ago       6.47MB
```



#### [연습] 실습용 `ubuntu` 이미지 제작하기

파일 구조는 다음과 같습니다.

```tex
.
├── install_docker_engine.sh
└── Dockerfile
```



**install_docker_engine.sh**

> https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository

```shell
# Add Docker's official GPG key:
apt-get update && apt-get upgrade
apt-get install -y ca-certificates curl gnupg
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  tee /etc/apt/sources.list.d/docker.list > /dev/null
apt-get update
```

**Dockerfile**

```dockerfile
FROM ubuntu:22.04

RUN mkdir -p /scripts
COPY install_docker_engine.sh /scripts

WORKDIR /scripts

RUN chmod +x install_docker_engine.sh
RUN ./install_docker_engine.sh

RUN apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```



다음 명령어를 통해서 `cloudwave:base.v1` 이미지를 빌드합니다.

```cmd
$ docker build -t cloudwave:base.v1 .
```



정상적으로 `docker`가 설치되었다면 다음과 같은 결과를 얻을 수 있습니다.

```cmd
$ docker run cloudwave:base.v1 docker version
Client: Docker Engine - Community
 Version:           24.0.7
 API version:       1.43
 Go version:        go1.20.10
 Git commit:        afdd53b
 Built:             Thu Oct 26 09:07:41 2023
 OS/Arch:           linux/amd64
 Context:           default
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
```



#### [실습] `Arg`를 이용하여 `base`이미지 변경하기

1. `Go 서버 이미지 제작하기`에서 사용한 `Dockerfile`을 수정합니다.
    1. `ARG`이름은 `OS`로 설정합니다.
    2. `ARG`를 이용하여 `Base 이미지`를 입력받아야 합니다.
    3. `ARG`의 값을 환경변수(`BASE`)에 저장합니다.
2. `--build-args` 옵션을 사용하여 `Debian(bullseye)`, `alpine`를 기반으로한 이미지를 생성합니다.
3. `exec`를 이용하여 환경변수 `BASE` 값을 확인합니다.



#### [실습] 실습용 `ubuntu` 이미지의 Dockerfile 개선하기

문제에서 제공된 Dockerfile은 동작은 하지만, 아래 측면에서 개선 여지가 있습니다.

- 캐시 활용과 재현성
- 불필요한 레이어/파일 제거
- 빌드 실패/변경에 취약
- 이미지 용량 최적화
- 보안 관점(불필요 패키지 최소화, 권한 명확화)

이를 고려하여 기존에 작성된 `Dockerfile`을 개선해보세요.



**완료 조건**

- `docker build`가 성공해야 합니다.
- 생성된 이미지에서 `docker --version` 명령이 동작해야 합니다.
- 기존 Dockerfile 대비 레이어 수 또는 이미지 용량이 개선되어야 합니다(둘 중 하나 이상).



**힌트**

1. 여러개의 RUN을 합치는 것이 더 좋을 수 있습니다.
2. 필요한 파일을 Remote에서 가져오도록 할 수 있습니다.
3. 컨테이너 실행에 불필요한 파일은 삭제하는 것이 좋습니다. 



