---
title: "Practice"
weight: 40
draft: false
---

## 종합 문제

### [실습-1] `code-server` 실행을 위한 `docker-compose.yaml` 작성하기

- 다음과 같이 `secret`을 생성 및 정의합니다.

  - 다음 명령어를 이용하여 `password.txt`를 생성합니다.

    > 임의의 텍스트 파일을 메모장등을 이용해서 생성해도 무방합니다.

    ```cmd
    # Window - CMD
    $ echo %RANDOM% > password.txt 
    
    # Linux
    $ openssl rand -base64 14 > password.txt 
    ```

  - 위에서 생성한 `password.txt` 파일을 `docker-compose.yaml`에 정의합니다.

    - 이름은 `code-server-password`로 설정합니다.

- 다음과 같이 `volume`을 생성 및 정의합니다.

  - `local_code` 이름으로 `volume`을 생성합니다.
  - 생성한 `volume`을 `docker-compose.yaml`에 정의합니다.

- `.env` 파일을 다음과 같이 생성합니다.

```tex
DEFAULT_WORKSPACE=/code
PUID=1000
PGID=1000
```

- `code-server` 서비스를 다음과 같이 정의합니다

  - 이미지는 `Docker Hub`에 `Push` 했던 `cloudwave:practice.v1`를 사용합니다.

  - 컨테이너 이름은 `ide`로 설정합니다.

  - 다음과 같이 환경 변수를 설정합니다.

    ```tex
    FILE__PASSWORD: /run/secrets/<SECRET_NAME>
    ```

  - `.env` 파일을 이용하여 추가로 환경 변수를 설정합니다.

  - `/code` 디렉토리를 `Working directory`로 설정합니다.

  - 로컬 머신의 `8443` 포트와 컨테이너의 `8443` 포트를 바인딩합니다.

  - 다음 `secret`을 사용할 수 있도록 설정합니다.

    - `code-server-password`

  - 다음과 같이 `volume`을 정의합니다.

    - 로컬 머신의 `/var/run/docker.sock`를 컨테이너의 `/var/run/docker.sock`로 마운트합니다.
    - `volume`의 `local_code`를 컨테이너의 `/code/local`에 마운트합니다.

- 작성한 `docker-compose.yaml`을 이용하여 프로젝트를 실행합니다.

- 브라우저에서 `localhost:8443`을 입력하고 `VSC`에 접속합니다.

  - 비밀번호는 생성한 `password.txt` 파일을 보고 확인합니다.



### [실습-2] `git-sync` 서비스 추가하기

- 다음과 같이 `volume`을 추가로 생성 및 정의합니다.

  - `remote_code` 이름으로 `volume`을 생성합니다.
  - 생성한 `volume`을 `docker-compose.yaml`에 정의합니다.

- `GitSync` 서비스를 다음과 같이 정의합니다.

  - 이미지는 `registry.k8s.io/git-sync/git-sync:v4.1.0`를 사용합니다.

  - `source_code` 볼륨을 `/tmp`에 마운트합니다.

  - 다음과 같이 환경 변수를 설정합니다.

    ```tex
    GITSYNC_REPO=https://github.com/matenduel/cloudwave
    GITSYNC_ROOT=/tmp/git
    GITSYNC_REF=main
    GITSYNC_DEPTH=1
    GITSYNC_ONE_TIME=1
    ```

  - `profile`이 `init`인 경우에만 실행되도록 설정하세요.

- `code-server` 서비스에 다음 사항을 추가로 정의합니다.

  - `volume`의 `remote_code`를 컨테이너의 `/code/remote`에 마운트합니다.
  - `depend_on`을 이용하여 `GitSync` 컨테이너가 생성된 다음 컨테이너가 실행되도록 설정하세요.
    - `GITSYNC_ONE_TIME`가 `1`이라면 `service_completed_successfully` 로 설정하세요.
    - `GITSYNC_ONE_TIME`가 설정되지 않았다면 `service_started` 로 설정하세요.


### [실습-3] `build`를 이용한 `docker compose` 프로젝트 생성하기

- 다음 조건을 만족하는 `Dockerfile`을 작성합니다.

  - `ubuntu:22.04`이미지를 베이스로 사용합니다.
  - `apt-get update`와 `apt-get upgrade`를 수행해야 합니다.
  - `apt`를 이용해서 `dnsutils`와 `wget`을 설치합니다.

- `ubuntu` 서비스를 다음과 같이 정의합니다.

  - 컨테이너의 이름을 `server`로 설정합니다.
  - 이미지 이름은 `cloudwave:ubuntu.dig.v1`으로 설정합니다.
  - 위에서 만든 `Dockerfile`을 기반으로 빌드되도록 설정합니다.
  - 컨테이너가 정지되지 않도록 다음과 같이 `command`를 설정합니다.
    - `sleep infinity`

- `web-app` 서비스를 다음과 같이 정의합니다.

  - 이미지는 `nginx:latest`를 사용합니다.
  - `replicas`를 3으로 설정합니다.
  - 80번 포트를 `Expose` 합니다.

- 작성한 `docker-compose.yaml`을 이용하여 이미지를 빌드한 다음 프로젝트를 실행합니다.

- `server` 컨테이너에 접속하여 다음 명령어를 실행합니다.

  ```
  dig web-app
  ```


