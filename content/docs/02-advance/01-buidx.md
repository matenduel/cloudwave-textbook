---
title: "Advance - Buildx"
weight: 50
draft: false
---

## 1.7. Docker Advance - Image

### Docker commit

> https://docs.docker.com/engine/reference/commandline/commit/

```shell
docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
```



**주요 옵션**

| Option      | Short | Default | Description                                     |
| ----------- | ----- | ------- | ----------------------------------------------- |
| `--author`  | `-a`  |         | `Author`를 설정합니다.                          |
| `--change`  | `-c`  |         | 추가로 적용할 `Dockerfile` 명령어를 설정합니다. |
| `--message` | `-m`  |         | Commit message                                  |
| `--pause`   | `-p`  | `true`  | `commit`동안 컨테이너를 중지합니다.             |



**주의사항**

- `Production`에서 사용할 이미지라면 `commit` 대신  `Dockerfile`을 기반으로 제작하는 것이 좋습니다.
- 볼륨(`volume`) 또는 `bind mount`에 저장된 데이터는 포함되지 않습니다.
    - 컨테이너 레이어 (`RW Layer`)에 저장된 데이터만 포함됩니다.

- `--pause` 옵션을 설정하지 않은 경우, `commit`하는 동안 컨테이너를 중지(`pause`)됩니다.
- `--change`에서 지원하는 명령어는 다음과 같습니다
    - CMD
    - ENTRYPOINT
    - ENV
    - EXPOSE
    - LABEL
    - ONBUILD
    - USER
    - VOLUME
    - WORKDIR



### Docker buildx

> https://docs.docker.com/engine/reference/commandline/buildx/

```
Usage:  docker buildx [OPTIONS] COMMAND

Extended build capabilities with BuildKit

Options:
      --builder string   Override the configured builder instance

Management Commands:
  imagetools  Commands to work on images in registry

Commands:
  bake        Build from a file
  build       Start a build
  create      Create a new builder instance
  du          Disk usage
  inspect     Inspect current builder instance
  ls          List builder instances
  prune       Remove build cache
  rm          Remove a builder instance
  stop        Stop builder instance
  use         Set the current builder instance
  version     Show buildx version information

Run 'docker buildx COMMAND --help' for more information on a command.
```

`builder`는 BuildKit 기반의 `빌드 실행 환경(인스턴스)`이며 드라이버에 따라 멀티플랫폼/캐시/출력 방식이 달라집니다.



#### builder 목록 확인하기

> https://docs.docker.com/engine/reference/commandline/buildx_ls/

```shell
docker buildx ls
```



##### [예시] 현재 사용중인 `builder` 확인하기

현재 선택된 `builder`는 `*`이 붙어있습니다.

```cmd
$ docker buildx ls
NAME/NODE       DRIVER/ENDPOINT STATUS  BUILDKIT             PLATFORMS
default *       docker
  default       default         running v0.11.6+616c3f613b54 linux/amd64, linux/amd64/v2, linux/amd64/v3, linux/arm64, linux/riscv64, linux/ppc64le, linux/s390x, linux/386, linux/mips64le, linux/mips64, linux/arm/v7, linux/arm/v6
```



#### builder 생성하기

> https://docs.docker.com/engine/reference/commandline/buildx_create/

```shell
docker buildx create [OPTIONS] [CONTEXT|ENDPOINT]
```



| Option         | Short | Default | Description                                                  |
| -------------- | ----- | ------- | ------------------------------------------------------------ |
| `--driver`     |       |         | 사용할 드라이버를 설정합니다. <br />(`docker-container`, `kubernetes`, `remote`) |
| `--driver-opt` |       |         | 드라이버에 따른 옵션을 설정합니다.                           |
| `--name`       |       |         | `builder` 이름을 지정합니다.                                 |
| `--platform`   |       |         | `platform`을 지정합니다.                                     |
| `--use`        |       |         | 생성 후 해당 `builder`를 사용합니다.                         |



##### Driver별 특징

> https://docs.docker.com/build/drivers/

| Feature                      | `docker` | `docker-container` | `kubernetes` |      `remote`      |
| :--------------------------- | :------: | :----------------: | :----------: | :----------------: |
| **Automatically load image** |    ✅     |                    |              |                    |
| **Cache export**             |    ✓*    |         ✅          |      ✅       |         ✅          |
| **Tarball output**           |          |         ✅          |      ✅       |         ✅          |
| **Multi-arch images**        |          |         ✅          |      ✅       |         ✅          |
| **BuildKit configuration**   |          |         ✅          |      ✅       | Managed externally |



##### [예시] `Multi-platform`용 `builder` 생성하기

```cmd
$ docker buildx create --driver docker-container --name multi-builder --platform linux/amd64,linux/arm64

$ docker buildx inspect multi-builder                                                                    
Name:          multi-builder
Driver:        docker-container
Last Activity: 2024-07-07 06:39:02 +0000 UTC

Nodes:
Name:      multi-builder0
Endpoint:  npipe:////./pipe/docker_engine
Status:    inactive
Platforms: linux/amd64*, linux/arm64*
```



#### builder 정보 조회

> https://docs.docker.com/engine/reference/commandline/buildx_inspect/

```shell
docker buildx inspect [NAME]
```



##### [예시] 현재 사용중인 `builder` 조회하기

```cmd
$ docker buildx inspect --bootstrap
Name:          default
Driver:        docker
Last Activity: 2023-12-22 02:37:31 +0000 UTC

Nodes:
Name:      default
Endpoint:  default
Status:    running
Buildkit:  v0.11.6+616c3f613b54
Platforms: linux/amd64, linux/amd64/v2, linux/amd64/v3, linux/arm64, linux/riscv64, linux/ppc64le, linux/s390x, linux/386, linux/mips64le, linux/mips64, linux/arm/v7, linux/arm/v6
Labels:
 org.mobyproject.buildkit.worker.moby.host-gateway-ip: 192.168.65.254
GC Policy rule#0:
 All:           false
 Filters:       type==source.local,type==exec.cachemount,type==source.git.checkout
 Keep Duration: 172.8µs
 Keep Bytes:    2.764GiB
GC Policy rule#1:
 All:           false
 Keep Duration: 5.184ms
 Keep Bytes:    20GiB
GC Policy rule#2:
 All:        false
 Keep Bytes: 20GiB
GC Policy rule#3:
 All:        true
 Keep Bytes: 20GiB
```



#### builder 선택하기

> https://docs.docker.com/engine/reference/commandline/buildx_use/

```shell
docker buildx use [OPTIONS] NAME
```



| Option      | Short | Default | Description             |
| ----------- | ----- | ------- | ----------------------- |
| `--default` |       |         | `default`로 설정합니다. |



#### 이미지 빌드하기

> https://docs.docker.com/engine/reference/commandline/buildx_build/

```shell
docker buildx build [OPTIONS] PATH | URL | -
```



**주요 옵션**

| Option        | Short | Default | Description                                                  |
| ------------- | ----- | ------- | ------------------------------------------------------------ |
| `--build-arg` |       |         | `ARG`를 설정합니다.                                          |
| `--file`      | `-f`  |         | `Dockerfile`의 경로를 지정합니다.                            |
| `--label`     |       |         | 라벨을 추가합니다.                                           |
| `--no-cache`  |       |         | 이미지 빌드시 캐시를 사용하지 않습니다.                      |
| `--platform`  |       |         | `platform`을 지정합니다.                                     |
| `--pull`      |       |         | 관련된 이미지를 저장 유무에 관계없이 `pull`합니다.           |
| `--load`      |       |         | 이미지를 `host`에 저장합니다.<br />(`--output=type=docker`와 동일합니다.) |
| `--push`      |       |         | 이미지를 `registry`에 저장합니다<br />(`--output=type=registry`와 동일합니다.) |
| `--tag`       | `-t`  |         | 이름과 `Tag`를 설정합니다.                                   |



**주의사항**

- `docker` 드라이버를 사용하는 `builder`는 단일 `platform` 이미지만 빌드할 수 있습니다.
- `docker` 드라이버가 아닌 다른 드라이버를 사용하는 경우 `--load`를 사용하여야 이미지를 가져올 수 있습니다.
    - `--load`는 `export`를 이용하여 이미지를 추출하고 `import`합니다.
- 2개 이상의 `platform`을 지원하는 이미지를 제작하는 경우 `--load`가 불가능하므로 `--push`를 통해 `registry`로 바로 업로드해야 합니다.



##### [예시] `linux/arm/v7`용 이미지 제작하기

현재 `builder`에서 지원하는 `platform`을 다음과 같이 확인합니다.

```cmd
$ docker buildx inspect --bootstrap
Name:          multi-arch-builder
Driver:        docker-container
Last Activity: 2023-12-23 13:46:16 +0000 UTC

...
Platforms: linux/amd64, linux/amd64/v2, linux/amd64/v3, linux/arm64, linux/riscv64, linux/ppc64le, linux/s390x, linux/386, linux/mips64le, linux/mips64, linux/arm/v7, linux/arm/v6
...
```



`--platform` 옵션을 이용하여 `linux/arm/v7`용 이미지를 제작합니다.

```cmd
$ docker buildx build --platform linux/arm/v7 -t cloudwave:arm .
...

$ docker image inspect cloudwave:arm -f "arch: {{ .Architecture }}/{{ .Variant }}" 
arch: arm/v7
```



---

### 연습 문제

#### [연습] Commit을 이용하여 패키지가 추가로 설치된 이미지 생성하기

다음과 같이 `Ubuntu` 컨테이너를 실행합니다.

```cmd
$ docker run -it -d --name base ubuntu:22.04
ebb9fe13e5cfb0747e5bea1db7c41ef30bf416bdcd643d7311e161ee4300b628
```



`curl` 설치 여부를 확인합니다.

```cmd
$ docker exec base apt list --installed "curl*"
WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

Listing...
```



다음과 같이 `Curl`을 설치한 후, 설치 여부를 재확인합니다.

```cmd
$ docker exec base /bin/bash -c "apt-get update && apt-get upgrade && apt-get install -y curl"

$ docker exec base apt list --installed "curl*"

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

Listing...
curl/jammy-updates,jammy-security,now 7.81.0-1ubuntu1.15 amd64 [installed]
```



`commit`을 이용하여 `base` 컨테이너를 다음과 같이 저장합니다.

```cmd
$ docker commit base commit:v1
sha256:56c923d569eccda8bd094286ca7356ea2fa1a3a7794df6f22369980dd78bc943

$ docker images commit
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
commit       v1        56c923d569ec   16 seconds ago   132MB
```



저장된 이미지를 실행하여 `curl`이 설치되어 있는지 확인합니다.

```cmd
$ docker run --name restore commit:v1 apt list --installed "curl*"

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

Listing...
curl/jammy-updates,jammy-security,now 7.81.0-1ubuntu1.15 amd64 [installed]
```



#### [연습] 실행중인 컨테이너의 Port를 추가로 `Expose`하기

컨테이너를 다음과 같이 생성하고, `Port`가 노출되지 않은 것을 체크합니다다.

```cmd
$ docker run -it -d --name base ubuntu:22.04
$ docker ps
CONTAINER ID   IMAGE          COMMAND                   CREATED       STATUS       PORTS     NAMES
51c471535b96   ubuntu:22.04   "/bin/bash"              2 hours ago   Up 2 hours             base
```



`--change` 옵션을 이용하여 80번 포트를 `Expose`합니다.

이 때, 기존에 실행중인 컨테이너가 멈추지 않도록 `--pause`를 `false`로 설정합니다.

```cmd
$ docker commit --change="EXPOSE 80" --pause=false base commit:v1
sha256:f908d82f79101ec792d5383a627c8ffa9dd92d65bf5674cf0ad8e9c0c7c943b2
```



저장한 이미지를 사용하여 새로운 컨테이너를 생성하고, 80번 포트가 열려있는 것을 확인합니다.

```cmd
$ docker run -itd commit:v1
c551f14323f9bc6dccf23b0a68fb9c4610e73079e7d76fe09306ed35aa8c0f57

$ docker ps
CONTAINER ID   IMAGE          COMMAND                   CREATED              STATUS              PORTS     NAMES
c551f14323f9   commit:v1      "/bin/bash"               About a minute ago   Up About a minute   80/tcp    commit
51c471535b96   ubuntu:22.04   "/bin/bash"               2 hours ago          Up 2 hours                    base
```



`inspect`를 사용하면 다음과 같이 `commit:v1` 이미지에 새로운 레이어가 추가되어 있는 것을 확인할 수 있습니다.

> `jq`가 설치되어 있지 않은 경우, `Appendix`를 참고하여 설치해주세요.

```cmd
$ docker inspect ubuntu:22.04 | jq ".[0].RootFS.Layers"
[
  "sha256:a1360aae5271bbbf575b4057cb4158dbdfbcae76698189b55fb1039bc0207400"
]
$ docker inspect commit:v1 | jq ".[0].RootFS.Layers"
[
  "sha256:a1360aae5271bbbf575b4057cb4158dbdfbcae76698189b55fb1039bc0207400",
  "sha256:36005e181ab5ff954b4d4191c6a3f6a69a62cf2d3367c53e6889ee2d14757c44"
]
```



#### [연습] `buildx`를 이용하여 `ARM`용 `cloudwave:base.v1` 이미지 제작하기

> [연습] 실습용 `ubuntu` 이미지 제작하기

`buildx ls`를 이용하여 현재 사용중인 `builder`가 지원하는 `platform` 목록을 확인합니다.

```cmd
$ NAME/NODE             DRIVER/ENDPOINT                STATUS  BUILDKIT             PLATFORMS
default *             docker
  default             default                        running v0.11.6+616c3f613b54 linux/amd64, linux/amd64/v2, linux/amd64/v3, linux/arm64, linux/riscv64, linux/ppc64le, linux/s390x, linux/386, linux/mips64le, linux/mips64, linux/arm/v7, linux/arm/v6
```



`buildx build` 명령어를 이용하여 `linux/arm/v7`용 이미지를 제작합니다.

```cmd
# builder의 driver가 docker인 경우
$ docker buildx build --platform linux/arm/v7 -t cloudwave:arm.v1 . 

# builder의 driver가 docker-container인 경우 `--load` 옵션을 추가합니다.
$ docker buildx build --platform linux/arm/v7 -t cloudwave:arm.v1 --load .
```



다음 명령어를 실행하여 생성한 이미지의 `Architecture`를 확인 할 수 있습니다.

```cmd
$ docker image inspect cloudwave:arm.v1 -f "arch: {{ .Architecture }}/{{ .Variant }}"
arch: arm/v7
```



#### [실습] `buildx`를 이용하여 `Multi-platform` 이미지 제작하기

1. `Docker Hub`에 로그인 합니다.
2. `docker-container`를 사용하는 `builder`를 생성합니다.
    - `--use` 옵션을 사용하여 바로 사용할 수 있도록 설정합니다.
3. 다음 조건을 만족하는 `Multi-platform` 이미지를 `build`합니다
    - `--push`를 이용하여 `Docker Hub`로 업로드해야 합니다.
    - 이미지는 `linux/amd64`, `arm/v6`를 지원해야 합니다.
4. `Docker Hub`에서 업로드한 이미지를 확인합니다. 

