---
title: "Practice-1"
weight: 91
draft: false
---


### [실습 1-1] 실습용 `Code-server` 이미지 만들기

> `code-server`는 Web에서 VS Code를 사용할 수 있게 만들어주는 서비스입니다.
> ![code-server]({{< relURL "images/docker/code-server-banner.png" >}})

- `linuxserver/code-server:4.107.0` 이미지를 사용합니다.

- `Dockerfile`에서 다음 환경변수들을 설정합니다.

  ```
  PASSWORD=password
  DEFAULT_WORKSPACE=/code
  PUID=0
  PGID=0
  TZ="Asia/Seoul"
  ```

- 다음 스크립트를 사용하여 Docker를 설치합니다.

    - https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository
    - `Dockerfile` 작성에 앞서 설치 명령이 정상 동작하는지 확인하기 위해, 미리 `code-server` 컨테이너 실행 후 먼저 설치 명령어를 테스트해보는 것이 좋습니다.

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
apt-get install -y docker-ce docker-ce-cli
```

- 제작한 이미지를 이용하여 다음 조건을 만족하는 컨테이너를 실행합니다.
    - 컨테이너의 이름은 `ide`로 설정합니다.
    - 다음과 같이 Volume을 설정합니다.
        - 로컬 머신의 `src` 디렉토리를 컨테이너의 `/code` 디렉토리에 마운트 합니다.
        - 로컬 머신의 `/var/run/docker.sock`를 컨테이너의 `/var/run/docker.sock`에 마운트 합니다.

    - 브라우저에서 `localhost:8443` 로 접근할 수 있어야합니다.

- `VSC` 터미널에서 다음 명령어를 사용하여 Docker 버젼을 확인합니다.

```shell
docker --version
```

![image-20240706234414182]({{< relURL "images/docker/image-20240706234414182.png" >}})





### [실습 1-2] DockerHub에 실습용 이미지 푸시하기

- `실습 1-1`에서 작성한 `Dockerfile`을 다음 조건을 만족하도록 수정하세요.

    - `Digest`를 사용하여 `code-server`의 버젼을 지정하세요.

        - `sha256:e2ebedc28ab9e2ebe08093cf7e78515f97822956ff7cbac3d86fb0bd9e4b6bca`

    - 다음 스크립트를 참고하여 패키지들을 추가로 설치하세요.

        - Ubuntu Packages

          ```sh
          apt install -y software-properties-common wget unzip apt-transport-https
          ```

        - AWS CLI

          ```sh
          curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
          unzip awscliv2.zip
          ./aws/install
          ```


- `Dockerfile`을 이용하여 이미지를 빌드하세요

    - 이름은 `cloudwave`로 설정해주세요
    - Tag는 `practice.v1`으로 설정해주세요

- `Docker Hub`에 이미지 푸시하세요

### [Extra] `buildx`를 이용하여 `Docker Hub`에 `Multi platform` 이미지를 `Push` 하기

- [선택] `docker login`을 이용하여 `Registry`에 로그인하세요.

- 다음 조건을 만족하는 `builder`를 생성하세요

    - `linux/amd64`와 `linux/arm64`를 지원해야 합니다.
    - `docker-container`를 `driver`로 설정하세요

- 다음 조건을 만족하는 `Dockerfile`을 작성하세요.

    - `Digest`대신 `tag`를 사용하세요

    - `Multi-stage`를 이용하여 `amazon/aws-cli:2.17.9` 이미지에서 `aws-cli` 패키지를 복사하세요.

        - `aws-cli`는 `/usr/local/aws-cli/v2`에 설치되어 있습니다.
        - `aws` 명령어를 사용하기 위해서 다음과 같이 `Symbolic link`를 설정하세요.

      ```sh
      ln -s /usr/local/aws-cli/v2/current/bin/aws /usr/local/bin/aws
      ln -s /usr/local/aws-cli/v2/current/bin/aws_completer /usr/local/bin/aws_completer
      ```

    - `aws-cli`를 제외한 나머지 패키지는 기존 방식대로 설치해도 상관 없습니다.

- `Docker Hub`에서 다음과 같이 `Multi-platform` 이미지가 `Push`되었는지 체크합니다.

![image-20240707014910268]({{< relURL "images/docker/image-20240707014910268.png" >}})


