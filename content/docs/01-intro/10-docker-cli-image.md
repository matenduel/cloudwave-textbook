---
title: "CLI - Image"
weight: 20
draft: false
---

## 1.1. Docker 이미지 관리

### 이미지 검색

> https://docs.docker.com/engine/reference/commandline/search/

```shell
docker search [OPTIONS] <Keyword>
```



다음은 `ubuntu` 이미지를 검색한 결과입니다.

![cmd_docker_search_result]({{< relURL "images/docker/cmd_docker_search_result.png" >}})



CLI 대신 Web에서도 사용할 이미지를 검색할 수 있습니다.

> https://hub.docker.com/

![docker_search_hub_result]({{< relURL "images/docker/docker_search_hub_result.png" >}})



### 이미지 다운로드

> https://docs.docker.com/engine/reference/commandline/pull/

```shell
docker pull [OPTIONS] REPOSITORY[:TAG|@DIGEST]

# Default
docker pull <REPOSITORY>:latest

# By Tag
docker pull <REPOSITORY>:<TAG>

# By digest
docker pull <REPOSITORY>@<DIGEST>
```



**주요 옵션**

| Option       | Short | Default | Description                          |
| ------------ | ----- | ------- | ------------------------------------ |
| `--all-tags` | `-a`  |         | 태그된 모든 이미지를 다운로드합니다. |
| `--platform` |       |         | 이미지의 `platform`을 설정합니다.    |



`Tag` 또는 `Digest`를 사용하지 않는 경우, `latest` 이미지를 다운로드 받습니다.

> 이미지를 다운로드할 때 이미지 자체가 아니라 이미지의 구성 정보를 담고 있는 `manifest`를 먼저 확인하고 그 정보를 기준으로 필요한 **레이어**를 선택하여 내려받습니다.

```cmd
$ docker pull ubuntu

Using default tag: latest
latest: Pulling from library/ubuntu
Digest: sha256:6042500cf4b44023ea1894effe7890666b0c5c7871ed83a97c36c76ae560bb9b
Status: Downloaded newer image for ubuntu:latest
docker.io/library/ubuntu:latest
```



#### Tag & Digest

- `Tag`는 사용자가 특정 이미지에 부여한 값이므로, 같은 `Tag`라도 다운로드 시점에 따라 다른 이미지를 나타낼 수 있습니다.
    - ex) `latest` 태그
- `Digest`는 변경이 불가능한 고유의 값이며, 동일한 방식으로 생성된 이미지라면 동일한 `Digest`가 생성됩니다.



#### Manifest

- Docker 이미지의 `manifest`는 해당 이미지가 어떤 파일 시스템 레이어들로 구성되어 있는지와 어떤 CPU 아키텍처(`amd64`, `arm64` 등)를 지원하는지를 설명하는 메타데이터입니다.



**주의사항**

- `docker pull`은 레지스트리의 이미지 `manifest(digest)`를 기준으로 로컬 캐시와 비교하여 이미지 다운로드 여부를 결정합니다.
    - `docker run` 시에는 로컬에 이미지가 존재한다면 따로 값을 확인하지 않습니다.

- 동일한 이미지더라도 `Tag`가 다른 경우 이미지가 추가됩니다.
- 이미지마다 지원하는 `arch`가 다를 수 있습니다.
    - `arch` = arm, amd64, ...




### 이미지 목록 보기

> https://docs.docker.com/engine/reference/commandline/images/

```shell
docker images [OPTIONS] [REPOSITORY[:TAG]]

# Alias
docker image list
docker image ls
```



**주요 옵션**

| Option       | Short | Default | Description                                     |
| ------------ | ----- | ------- | ----------------------------------------------- |
| `--all`      | `-a`  |         | 중간 이미지(`intermediate image`)도 표시합니다. |
| `--digests`  |       |         | `Digest`를 표기합니다.                          |
| `--filter`   | `-f`  |         | 필터 조건에 맞는 이미지만 표시합니다.           |
| `--no-trunc` |       |         | `sha256` 형식의 전체 이미지 ID를 표기합니다.    |
| `--quiet`    | `-q`  |         | 이미지 ID만 표기합니다.                         |



`Repository`를 명시한 경우, 해당 이름을 가진 이미지만 검색합니다.

```cmd
$ docker images ubuntu
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
ubuntu       22.04     174c8c134b2a   2 weeks ago   77.9MB
ubuntu       latest    174c8c134b2a   2 weeks ago   77.9MB
```



### 이미지 삭제하기

> https://docs.docker.com/engine/reference/commandline/rmi/

```shell
docker rmi [OPTIONS] IMAGE [IMAGE...]

# Alias
docker image rm
```

`docker rmi`는 한 개 이상의 명시된 이미지들을 삭제하는 명령입니다.



**주요 옵션**

| Option    | Short | Default | Description                                  |
| --------- | ----- | ------- | -------------------------------------------- |
| `--force` | `-f`  |         | 컨테이너가 존재하더라도 이미지를 삭제합니다. |



**주의사항**

- 동일한 `Image ID`를 가진 이미지가 2개 이상인 경우 `Image ID`를 이용하여 삭제할 수 없습니다.
    -  `-f` 옵션을 사용한 경우 모든 이미지를 삭제합니다.
- 다른 이미지가 삭제하려는 이미지를 베이스 이미지(Base Image)로 사용하고 있는 경우 삭제할 수 없습니다.



### 모든 이미지 삭제하기

> https://docs.docker.com/engine/reference/commandline/image_prune/

```sh
docker image prune [OPTIONS]
```

`docker image prune`은 모든 `dangling` 이미지를 삭제하는 명령어 입니다.



#### `dangling` 이미지

- `Repository`가 `none`으로 표기되는 이미지입니다.
- 일반적으로 다음과 같은 상황에서 발생합니다.
    - 동일한 이름(`Name`&`Tag`)을 가진 다른 이미지를 생성한 경우
    - `Multi-Stage` 빌드중 생성된 중간 이미지

```cmd
$ docker images -f dangling=true
REPOSITORY   TAG       IMAGE ID       CREATED              SIZE
<none>       <none>    5b95739d14ad   About a minute ago   1.02GB
```



**주요 옵션**

| Option     | Short | Default | Description                                                  |
| ---------- | ----- | ------- | ------------------------------------------------------------ |
| `--all`    | `-a`  |         | `dangling` 이미지를 포함한 사용하지 않는 모든 이미지를 삭제합니다. |
| `--filter` |       |         | Provide filter values (e.g. `until=<timestamp>`)             |



### 이미지 정보 조회

> https://docs.docker.com/engine/reference/commandline/inspect/

```shell
docker inspect [OPTIONS] NAME|ID [NAME|ID...]
```



#### [예시]  `ubuntu:22.04`이미지 정보 조회하기

```cmd
$ docker inspect ubuntu:22.04
[
    {
        "Id": "sha256:174c8c134b2a94b5bb0b37d9a2b6ba0663d82d23ebf62bd51f74a2fd457333da",
        "RepoTags": [
            "ubuntu:22.04",
            "ubuntu:latest"
        ],
        "RepoDigests": [
            "ubuntu@sha256:6042500cf4b44023ea1894effe7890666b0c5c7871ed83a97c36c76ae560bb9b"
        ],
        "Parent": "",
        "Comment": "",
        "Created": "2023-12-12T11:38:59.637410824Z",
        "Container": "24376bd380dd052e15c3e218153b3ba996b290ab24a579085cb3dd4d4b44e5ad",
        "ContainerConfig": {
            "Hostname": "24376bd380dd",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "#(nop) ",
                "CMD [\"/bin/bash\"]"
            ],
            "Image": "sha256:76cebcaf36a483db6a1f4eeebafda25bcfd25839a3c97101aa578d3120a8b0f8",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {
                "org.opencontainers.image.ref.name": "ubuntu",
                "org.opencontainers.image.version": "22.04"
            }
        },
        "DockerVersion": "20.10.21",
        "Author": "",
        "Config": {
            "Hostname": "",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/bash"
            ],
            "Image": "sha256:76cebcaf36a483db6a1f4eeebafda25bcfd25839a3c97101aa578d3120a8b0f8",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {
                "org.opencontainers.image.ref.name": "ubuntu",
                "org.opencontainers.image.version": "22.04"
            }
        },
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 77850032,
        "VirtualSize": 77850032,
        "GraphDriver": {
            "Data": {
                "MergedDir": "/var/lib/docker/overlay2/08fca0476137c27031e67bdadf559dd8e4d5e5fcebeec0c5e0534024e880c322/merged",
                "UpperDir": "/var/lib/docker/overlay2/08fca0476137c27031e67bdadf559dd8e4d5e5fcebeec0c5e0534024e880c322/diff",
                "WorkDir": "/var/lib/docker/overlay2/08fca0476137c27031e67bdadf559dd8e4d5e5fcebeec0c5e0534024e880c322/work"
            },
            "Name": "overlay2"
        },
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:a1360aae5271bbbf575b4057cb4158dbdfbcae76698189b55fb1039bc0207400"
            ]
        },
        "Metadata": {
            "LastTagTime": "0001-01-01T00:00:00Z"
        }
    }
]
```



---

### 연습 문제

#### [연습] ARM용 Ubuntu 다운로드하기

다음 명령어를 이용하여 `ubuntu:22.04`의 `linux/arm64`용 이미지를 다운로드할 수 있습니다.

```cmd
$ docker pull ubuntu:22.04 --platform linux/arm64/v8
22.04: Pulling from library/ubuntu
e730d307d74e: Pull complete
Digest: sha256:3c61d3759c2639d4b836d32a2d3c83fa0214e36f195a3421018dbaaf79cbe37f
Status: Downloaded newer image for ubuntu:22.04
docker.io/library/ubuntu:22.04
```



다운로드한 이미지의 `arch`를 `inspect`를 이용하여 확인합니다.

```cmd
$ docker inspect ubuntu:22.04 -f "{{ .Architecture }}"
arm64
```



#### [연습] PostgreSQL 이미지의 레이어 갯수 확인하기

> 조건
>
> - digest: `sha256:a2282ad0db623c27f03bab803975c9e3942a24e974f07142d5d69b6b8eaaf9e2`

제시된 `Digets`를 이용하여 다음과 같이 이미지를 다운로드합니다.

```cmd
$ docker pull postgres@sha256:a2282ad0db623c27f03bab803975c9e3942a24e974f07142d5d69b6b8eaaf9e2
```



다운로드한 이미지를 확인합니다.

```cmd
$ docker images postgres
REPOSITORY   TAG             IMAGE ID       CREATED        SIZE
postgres     <none>          391a00ec7cac   13 days ago    425MB

# Digest도 함께 표시
$ docker images --digests postgres
```



다음과 같이 `Docker Id`를 이용하여 해당 이미지를 구성하고 있는 레이어들을 확인할 수 있습니다.

```cmd
$ docker inspect 391a00ec7cac -f "{{range $v := .RootFS.Layers}}{{println $v}}{{end}}"
sha256:92770f546e065c4942829b1f0d7d1f02c2eb1e6acf0d1bc08ef0bf6be4972839
sha256:94ef9904d4df70d048f10800d60a22f6df9f45fe3c4d2a12e485761b7d695892
sha256:65c0efddc8b86104633e023acee9707dfa41249636f3690d315c01c987da56fe
sha256:7e28e769eedfd948a6c6d1dd70ee5a4b0d14b4576f38bb23e64ee30d9afd4d48
sha256:7fad9b4e65a17abc549695d9529b286d97c2453cb2e43e288dcc9a31f87b01b0
sha256:b693edccc3930a496794bd45d6e0fb1d0e7a2d00c3ec72130fe944696479e379
sha256:4f0b1281c6dcf2ceca4094d9730ea0a6184894ca020f94ff43ede84742b4da03
sha256:059a984b41a09df6a5309e7928fdb61772b9db821cae4dad464b4091b126d78b
sha256:b885153181c241ae470af66c8ee62619bb667a0579f9a93fcbecaf1482bbafc3
sha256:931d2dae8d071197900c6b9fd55756e9db68d383fd863f9c1dc40e4e9545f46e
sha256:983e1162f18556cbac001e4a4ae18766a64d157ad14bfef23bdab9a836be4b63
sha256:e59308f2f1f587291d3de81d2df9893e8ee395981e5f753dc370f3b8eda44b24
sha256:39db9e416d9745d9160895485c1d22543edbcffb365bc7ececd44ceab38a9c67
```




