---
title: "Setup"
weight: 20
draft: false
---


## Docker 설치하기

### Windows

- 아래 링크에서 Docker Desktop을 다운로드합니다.
  [https://desktop.docker.com/win/main/amd64/Docker%20Desktop%20Installer.exe](https://desktop.docker.com/win/main/amd64/Docker Desktop Installer.exe)

- 설치 후 Docker Desktop을 실행합니다.



### Ubuntu

- 다음 명령어를 통해서 Docker를 설치합니다.

```shell
sudo apt-get update
sudo apt-get install -y docker.io
```

- Docker 서비스가 실행중인지 확인합니다.

```shell
sudo systemctl start docker
sudo systemctl enable docker
```



## Docker 설치 확인하기

- PowerShell 또는 터미널에서 다음 명령어를 실행합니다.

  ```cmd
  docker version
  ```

