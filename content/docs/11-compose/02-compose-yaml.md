---
title: "YAML"
weight: 20
draft: false
---

## 2.2. Compose file - Basic

- `docker-compose`를 위한 `configuration` 파일은 `YAML`을 이용하여 작성합니다.

- `Python`과 같이 들여쓰기를 기반으로 구조를 결정합니다.



**`docker-compose.yaml` 파일 예시**

```yaml
version: "3.8"
name: "cloud_wave"

services:
  frontend:
    image: nginx:latest
    ports:
      - "80:80"
    networks:
      - private
    configs:
      - server-config
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
    secrets:
      - db_password
    env_file:
      - .env
    environment:
      DB_PASSWORD_PATH: /run/secrets/db_password
    restart: always

volumes:
  db-data:
    name: "db-data"
    labels:
      description: "PostgreSQL 16.1 volume"

configs:
  server-config:
    file: "server.conf"

# Secret not encrypted when swarm is off
secrets:
  db_password:
    file: "secret.txt"

networks:
  private:
    name: "private"
  db:
    name: "db"
```



### 2.2.1. Version

- `version`은 해당 `compose` 파일에서 사용한 버전을 의미합니다.

- `version`을 선언하여도 `docker compose`는 해당 파일을 실행할 수 있는 가장 최신 버전을 사용합니다.



### 2.2.2. Name

- 프로젝트의 이름을 의미합니다.

- 프로젝트의 이름은 다음과 같은 순서로 결정됩니다.

  1. `Compose CLI`의 `-p / --project-name`가 설정된 경우

  2. `configuration file(docker-compose.yaml)`에 `name`이 설정된 경우

  3. `directory` 이름



### 2.2.3. Service

> https://docs.docker.com/compose/compose-file/05-services/

```yaml
# Example
services:
  frontend:
    image: nginx:latest
    ports:
      - "80:80"
    configs:
      - server-config
    restart: always
```



#### image

```yaml
image: string
```

- 컨테이너에서 사용할 이미지입니다.
- 이미지를 `build`하는 경우 생성된 이미지의 이름으로 사용됩니다.



#### container_name

```yaml
container_name: string
```

- 값을 명시하지 않은 경우, 프로젝트, 서비스, 컨테이너 번호를 기반으로 생성됩니다.
- 알파벳 대소문자, 숫자 및 일부 특수기호(`.`, `_`, `-`)만 사용할 수 있습니다.
- `container_name`이 설정되어 있는 경우, 해당 서비스를 위한 컨테이너를 1개만 생성할 수 있습니다.



#### expose

```yaml
expose:
	- 80  # INT
	- "8080" # STRING
	- "1000-1010" # RANGE
```

- 같은 네트워크를 사용하는 서비스에서만 접근 가능하며, `host`로 `publish`되지 않습니다.
- `-`를 이용하여 노출할 포트 범위를 지정할 수 있습니다.



#### ports

```yaml
ports:
	# Short Syntax
	- [HOST:]CONTAINER[/PROTOCOL]
	# Long Syntax
	- target: 80
    host_ip: HOST
    published: "8080"
    protocol: tcp
```

- 외부에서 접근할 수 있도록 `host`로 `publish`할 포트를 정의합니다.
- `-`를 이용하여 노출할 포트 범위를 지정할 수 있습니다.



#### entrypoint

```yaml
# String
entrypoint: STRING
# List
entrypoint:
	- STRING
	- STRING
```

- 컨테이너의 `Dockerfile`에서 정의된 `ENTRYPOINT`를 `override`합니다.
- `ENTRYPOINT`가 선언된 경우, `Dockerfile`에서 정의된 `CMD`는 무시됩니다.



#### command

```yaml
# String
command: bundle exec thin -p 3000
# List
command: [ "bundle", "exec", "thin", "-p", "3000" ]
# List - yaml
command: 
	- STRING
	- STRING
```

- 컨테이너의 `Dockerfile`에서 정의된 `CMD`를 `override`합니다.
- 값이 `null`인 경우 `Image`의 `command`가 사용되지만, `[]` 또는 `''`로 설정된 경우  `Image`의 `command`는 무시됩니다.



#### environment

```yaml
# Map Syntax
environment:
  KEY1: value
  KEY2: "value"
  KEY3:
  
# Array Syntax
environment:
  - KEY1=value
  - KEY2="value"
  - KEY3
```

- `Map` 또는 `Array`형식으로 작성할 수 있습니다.
- 값이 정의되지 않은 경우, `env_file`등으로 제공되지 않으면 `unset` 됩니다.
- 동일한 변수가 `env_file`에도 선언되어 있는 경우, `environment`의 값이 설정됩니다.



#### env_file

```yaml
# Single
env_file: .env  # string

# Multiple
env_file:
  - ./a.env
  - ./b.env
```

- 여러개의 파일에 동일한 변수가 선언된 경우, 가장 마지막 파일에 있는 값으로 설정됩니다.
- `env_file`은 다음과 같은 양식으로 작성합니다.
  - `VAR[=[VAL]]`



**주의사항**

- `.env`는 기본적으로 **변수 치환**에 쓰이는 파일입니다.
  - `docker-compose.yaml`파일을 파싱할 때 `${VAR}` 값을 치환하는 데 사용됩니다.
- `env_file`은 컨테이너에 **실제로 전달될 환경변수** 파일을 지정하는 옵션입니다.



#### build

> https://docs.docker.com/compose/compose-file/build/

```yaml
services:
	# Short
  frontend:
    image: example/webapp
    build: ./webapp
	
	# Long - dockerfile
  backend:
    image: example/database
    build:
      context: backend
      dockerfile: ../backend.Dockerfile
      args:
        GIT_COMMIT: cdc3b19
```

- 서비스의 컨테이너를 `build`할 때 사용하며, 각 서비스별로 `context` 경로를 설정할 수 있습니다.
- `dockerfile` 대신 `dockerfile_inline`을 이용하여 사용할 수도 있습니다.

```yaml
build:
  context: .
  dockerfile_inline: |
    FROM baseimage
    RUN some command    
```



#### pull_policy

```yaml
pull_policy: string
```

- `Compose`가 이미지를 어떻게 가져올 것인지를 설정합니다.
- 사용할 수 있는 값은 다음과 같습니다.

| Value     | Description                                                  |
| --------- | ------------------------------------------------------------ |
| `always`  | 매번 `registry`에서 이미지를 다운로드 합니다.                |
| `never`   | 저장된 이미지만을 사용합니다. 이미지가 존재하지 않는 경우 실행되지 않습니다. |
| `missing` | 저장된 이미지가 없는 경우에만 `registry`에서 다운로드 합니다. |
| `build`   | 매번 이미지를 `build`합니다.                                 |



#### restart

```yaml
restart: string
```

- 컨테이너가 종료된 경우 어떻게 처리할 것인지를 정의합니다.
- 사용할 수 있는 값은 다음과 같습니다.

| Value            | Description                                              |
| ---------------- | -------------------------------------------------------- |
| `no`             | 어떠한 경우에도 컨테이너를 재실행하지 않습니다.          |
| `always`         | 제거되지 않는 한, 컨테이너를 항상 재실행합니다.          |
| `on-failure`     | `exit code`가 `error`인 경우에만 재실행합니다.           |
| `unless-stopped` | `종료`또는 `삭제`하는 경우를 제외하고 항상 재실행합니다. |



#### platform

```yaml
platform: string

# Example
platform: windows/amd64
platform: linux/arm64/v8
```

- 컨테이너의 `platform`을 명시합니다.



#### CLI option <-> Service elements

| Docker CLI Options | Service Elements |
| ------------------ | ---------------- |
| -i                 | stdin_open       |
| -t                 | tty              |
| -p                 | ports            |
| -e                 | environment      |
| --env-file         | env_file         |
| --name             | container_name   |



#### 연습 문제

##### [연습] `ubuntu` 서버 실행하기

아래에 있는 `docker-compose.yaml` 파일을 이용하여 프로젝트를 실행할 경우, `ubuntu` 서비스가 생성 후 바로 종료되는 것을 확인할 수 있습니다.

```yaml
# docker-compose.yaml
version: '3.8'

services:
  ubuntu:
    image: ubuntu:22.04
    restart: no
```

```cmd
$ docker compose up
$ docker compose ps -a
NAME                IMAGE          COMMAND       SERVICE   CREATED          STATUS                     PORTS
example2-ubuntu-1   ubuntu:22.04   "/bin/bash"   ubuntu    15 seconds ago   Exited (0) 4 seconds ago   
```



`docker run` 때와 마찬가지로 두가지 방식을 이용하여 유지시킬 수 있습니다.

1. `stdin_open`과 `tty`를 이용합니다.
  - `docker run`에서의 `-it` 옵션과 동일합니다.

```yaml
version: '3.8'

services:
  ubuntu:
    image: ubuntu:22.04
    tty: true
    stdin_open: true
    restart: no
```

2. `sleep`과 같은 명령어를 실행하여 프로세스를 유지합니다.

```yaml
version: '3.8'

services:
  ubuntu:
    image: ubuntu:22.04
    entrypoint: /bin/bash
    command:
      - -c
      - sleep infinity
    restart: no
```



##### [연습] 환경변수를 이용하여 `compose file` 제어하기

환경변수를 통해 `docker compose` 파일을 설정하는 방법은 일반적으로 다음과 같습니다.

1. `Host`에서 환경변수 설정하기
2. 환경 변수파일(`.env`)를 사용하기
  - `CLI`

다음 `.env`와 `docker-compose.yaml`을 이용하여 프로젝트를 빌드 후 실행합니다.

```tex
# .env
FROM=".env file"
```

```yaml
# project-1.yaml
version: '3.8'

services:
  ubuntu:
    image: ubuntu:22.04
    entrypoint: /bin/bash
    command:
      - -c
      - echo 'env from "$FROM"'
    restart: no
```



`docker compose up`을 이용하여 실행하면 다음과 같이 `.env`의 환경변수가 기입된 것을 확인할 수 있습니다.

```cmd
$ docker compose up
[+] Building 0.0s (0/0)                                                                                                                                                                                            docker:desktop-linux
[+] Running 1/0
 ✔ Container example3-ubuntu-1  Recreated                                                                                                                                                                                          0.0s 
Attaching to example3-ubuntu-1
example3-ubuntu-1  | env from ".env file"
example3-ubuntu-1 exited with code 0
```



이번에는 `host`에서 다음 명령어를 통해 다음과 같이 환경변수를 설정한 뒤, 실행해보면 다음과 같은 결과를 얻을 수 있습니다.

```cmd
$ export FROM="host"
$ docker compose up
[+] Building 0.0s (0/0)                                                                                                                                                                                            docker:desktop-linux
[+] Running 1/0
 ✔ Container example3-ubuntu-1  Recreated                                                                                                                                                                                          0.0s 
Attaching to example3-ubuntu-1
example3-ubuntu-1  | env from host
example3-ubuntu-1 exited with code 0
```



환경변수에 대해 `default` 값을 설정하고 싶은 경우 다음과 같이 작성하면 됩니다.

- `${ENV:-DEFAULT_VALUE}`

```yaml
# project-1.yaml
version: '3.8'

services:
  ubuntu:
    image: ubuntu:22.04
    entrypoint: /bin/bash
    command:
      - -c
      - echo 'env from "$FROM"' && echo 'env from ${BY:-default}'
    restart: no
```

위의 `docker-compose.yaml`을 실행하면 다음과 같이 "default" 값이 출력된 것을 확인할 수 있습니다.

```cmd
$ docker compose up
[+] Building 0.0s (0/0)                                                                                                                                                                                            docker:desktop-linux
[+] Running 1/0
 ✔ Container example3-ubuntu-1  Recreated                                                                                                                                                                                          0.0s 
Attaching to example3-ubuntu-1
example3-ubuntu-1  | env from host
example3-ubuntu-1  | env from default
example3-ubuntu-1 exited with code 0
```



##### [연습] `command`에서 컨테이너 환경변수 사용하기

`docker-compose.yaml` 파일에서 `$`를 이용하여 환경변수를 사용하는 경우,  컨테이너 내부의 환경변수가 사용되지 않습니다.

`command`에서 컨테이너 내부의 환경변수를 사용하고 싶은 경우 다음과 같이 `$$`를 이용하면 됩니다.

```yaml
version: '3.8'

services:
  ubuntu:
    image: ubuntu:22.04
    environment:
      - FROM="env definition"
    entrypoint: /bin/bash
    command:
      - -c
      - echo 'env from ${FROM}' && echo env from $${FROM}
    restart: no
```



위의 `yaml` 파일을 실행하면 다음과 같은 결과 값을 확인할 수 있습니다.

```cmd
$ docker compose up
[+] Building 0.0s (0/0)                                                                                                                                                                                            docker:desktop-linux
[+] Running 1/0
 ✔ Container example3-ubuntu-1  Recreated                                                                                                                                                                                          0.0s 
Attaching to example3-ubuntu-1
example3-ubuntu-1  | env from host
example3-ubuntu-1  | env from "env definition"
example3-ubuntu-1 exited with code 0
```



`docker inspect` 명령어를 통해서도 확인해볼 수 있습니다.

```cmd
$ docker inspect example3-ubuntu-1 -f "{{ .Args }}"
[-c echo 'env from host' && echo env from ${FROM}]
```



##### [연습] Docker compose에서 build 사용하기

``` yaml
version: '3.8'

services:
  ubuntu:
    container_name: server
    build:
      context: .
      dockerfile_inline: |
        FROM ubuntu:22.04
        RUN apt-get update && apt-get upgrade && apt-get install -y curl
    image: cloudwave/compose:inline_build.v1
    restart: no

  nginx:
    image: nginx:latest
    expose:
      - 80
    restart: always
```



```cmd
$ docker compose -f example_1.yaml -p ex1 up -d --build
```





### 2.2.4. volume

```yaml
services:
  backend:
    image: example/database
    volumes:
      - db-data:/etc/data

  backup:
    image: backup-service
    volumes:
      - type: volume
        source: mydata
        target: /data
        volume:
          nocopy: true
        read_only: true
      - db-data:/var/lib/backup/data:rw

volumes:
  db-data:
    name: "my-app-data"
    external: true
    labels:
      com.example.description: "Database volume"
```



#### 볼륨 정의하기

```yaml
volumes:
  db-data:  # volume_key
    name: "db-volume"
    external: true
    labels:
      com.example.description: "Database volume"
```



##### Name

- 볼륨의 이름입니다.
- 설정되지 않은 경우 `{project_name}_{volume_key}` 형태의 이름으로 명명됩니다.



##### External

- 생성되어 있는 기존 볼륨의 사용 여부입니다.
- 해당 볼륨이 생성되어 있지 않은 경우, 프로젝트가 실행되지 않습니다.
- `name`이 지정되어 있지 않은 경우, `volumes`에 정의된 `key`가 사용됩니다.
  - `db-volume`이 명시되지 않았다면, `db_data`란 이름을 가진 `volume`을 탐색합니다.



##### Labels

- 관리를 위한 라벨을 설정합니다.



**주의 사항**

- `volume`을 정의하였더라도 `service`에서 사용하지 않는 경우, 생성되지 않습니다.



#### 볼륨 사용하기

```yaml
services:
  backend:
    image: example/database
    volumes:
      - db-data:/etc/data

  backup:
    image: backup-service
    volumes:
      - type: volume
        source: mydata
        target: /data
        volume:
          nocopy: true
        read_only: true
      - db-data:/var/lib/backup/data:rw
```

- 정의된 볼륨은 서비스의 `volumes`에서 사용할 수 있습니다.

- `Short Syntax`, `Long Syntax` 두 가지 방식으로 서비스의 `volumes`를 작성할 수 있습니다.



##### Short Syntax

```tex
volumes:
	- VOLUME:CONTAINER_PATH:ACCESS_MODE
```

| Attributes       | Description                                                  |
| ---------------- | ------------------------------------------------------------ |
| `VOLUME`         | `host`의 경로 또는 `volume` 이름을 사용합니다. <br />`volume` 이름을 사용하는 경우 경로를 지정할 수 없습니다. |
| `CONTAINER_PATH` | 볼륨이 마운트될 컨테이너의 경로를 의미합니다.                |
| `ACCESS_MODE`    | 해당 볼륨의 `Access mode`를 설정합니다. <br />- `rw`: 읽기&쓰기 모두 가능합니다.<br />- `ro`: 읽기 전용으로 설정합니다. |



##### Long Syntax

> https://docs.docker.com/compose/compose-file/05-services/#long-syntax-5

```yaml
volumes:
  - type: volume
    source: mydata
    target: /data
    read_only: true
```

| Attributes  | Description                                                  |
| ----------- | ------------------------------------------------------------ |
| `type`      | 해당 볼륨의 `mount type`을 설정합니다. <br />- `volume`: `source`로 `volume`을 사용합니다.<br />- `bind`: `source`로 `host`의 `directory`를 사용합니다 |
| `source`    | `volume`이름 또는 `host`의 `directory` 경로를 설정합니다.    |
| `target`    | 컨테이너에서 볼륨이 `mount`될 `directory` 경로를 설정합니다. |
| `read_only` | 읽기 전용 여부를 설정합니다.                                 |



#### 연습 문제

##### [연습] External volume 사용하기

다음과 같이 `volume`을 생성합니다.

```cmd
$ docker volume create vault
vault
```



`volume`을 `ubuntu` 컨테이너의 `/root/vault`에 마운트하도록 `docker-compose.yaml`을 작성합니다.

```yaml
version: '3.8'
name: 'volume-external'

services:
  master:
    image: ubuntu:22.04
    entrypoint: /bin/bash
    command:
      - -c
      - sleep infinity
    volumes:
      - vault:/root/vault

volumes:
  vault:
    external: true
    name: 'vault'
```



##### [연습] read_only 로 설정하여 사용하기

다음 `docker-compose.yaml`을 이용하여 프로젝트를 실행합니다.

```yaml
# docker-compose.yaml
version: '3.8'
name: 'volume-external'

services:
  master:
    image: ubuntu:22.04
    entrypoint: /bin/bash
    command:
      - -c
      - sleep infinity
    volumes:
      - vault:/root/vault:rw

  slave:
    image: ubuntu:22.04
    entrypoint: /bin/bash
    command:
      - -c
      - sleep infinity
    volumes:
      - vault:/root/vault:ro

volumes:
  vault:
    external: true
    name: 'vault'
```

```cmd
$ docker compose -f docker-compose.yaml up -d
[+] Building 0.0s (0/0)                                                      docker:desktop-linux
[+] Running 2/2
 ✔ Container volume-external-master-1  Started                                              0.0s 
 ✔ Container volume-external-slave-1   Started                                              0.0s 
```



다음과 같이 `exec`를 이용하여 `master` 서비스의 컨테이너에서 `/root/vault`에 파일을 생성합니다.

```cmd
$ docker exec volume-external-master-1 /bin/bash -c "echo master > /root/vault/temp.txt"
```

생성한 파일이 정상적으로 읽어지는지 다음과 같이 확인해볼 수 있습니다.

```cmd
$ docker exec volume-external-master-1 /bin/bash -c "cat /root/vault/temp.txt"        
master
```



`slave` 서비스의 컨테이너에서는 `/root/vault`에 저장한 파일을 읽을 수 있지만, 파일을 기록하는 것은 불가능한 것을 확인할 수 있습니다.

```cmd
$ docker exec volume-external-slave-1 /bin/bash -c "cat /root/vault/temp.txt"         
master
$ docker exec volume-external-slave-1 /bin/bash -c "echo slave > /root/vault/temp.txt"
/bin/bash: line 1: /root/vault/temp.txt: Read-only file system
```



##### [연습] `volumes_from`을 이용하여 `volume` 사용하기

> https://docs.docker.com/compose/compose-file/05-services/#volumes

다음 파일을 이용하여 새로운 프로젝트를 실행합니다.

- 위에서 생성한 `volume-external` 프로젝트는 실행중이어야 합니다.

```yaml
# docker-compose.yaml
version: '3.8'
name: 'volume-external2'

services:
  other:
    image: ubuntu:22.04
    entrypoint: /bin/bash
    command:
      - -c
      - sleep infinity
    volumes_from:
      - container:volume-external-slave-1:ro
```



```cmd
$ docker compose -f docker-compose.yaml up -d                       
[+] Building 0.0s (0/0)                                                       docker:desktop-linux
[+] Running 2/2
 ✔ Network volume-external2_default    Created                                                0.1s 
 ✔ Container volume-external2-other-1  Started                                                0.0s 
```



별도의 `volumes`를 정의하지 않고도 `volumes_from`을 이용하여 다른 서비스의 `volume`을 사용할 수 있는것을 확인할 수 있습니다.

```cmd
$ docker exec volume-external2-other-1 /bin/bash -c "cat /root/vault/temp.txt"
master
$ docker exec volume-externa2-other-1 /bin/bash -c "echo other > /root/vault/temp.txt"
/bin/bash: line 1: /root/vault/temp.txt: Read-only file system
```



### 2.2.5. network

```yaml
services:
  frontend:
    image: example/webapp
    networks:
      - private

networks:
  private:
    name: "private_net"
    external: true
    labels:
      com.example.description: "private network"
```



#### 네트워크 정의하기

```yaml
networks:
  private:  # network_key
    name: "private_net"
    external: true
    labels:
      com.example.description: "private network"
```



##### Name

- 네트워크의 이름입니다.
- 설정되지 않은 경우 `{project_name}_{network_key}` 형태의 이름으로 명명됩니다.



##### External

- 생성되어 있는 기존 네트워크 사용 여부입니다.
- 해당 네트워크가 생성되어 있지 않은 경우, 프로젝트가 실행되지 않습니다.
- `name`이 지정되어 있지 않은 경우, `networks`에 정의된 `key`가 사용됩니다.
  - `private_net`이 명시되지 않았다면, `private`란 이름을 가진 `network`를 탐색합니다.



##### Labels

- 관리를 위한 라벨을 설정합니다.



**주의 사항**

- `network`를 정의하였더라도 `service`에서 사용하지 않는 경우, 생성되지 않습니다.





#### 네트워크 사용하기

```yaml
services:
  backend:
    image: example/database
    # List Syntax
    networks:
      - private

  backup:
    image: backup-service
    # Map Syntax
    networks:
      private:
        aliases:
          - alias1
          
  bastion:
    image: bastion
    network_mode: host
```

- 정의된 볼륨은 서비스의 `networks`에서 사용할 수 있습니다.
- `host` 또는 `none` 네트워크를 사용하려는 경우, `network_mode`를 이용하여 설정합니다.



**주의사항**

- 1개 이상의 컨테이너에서 동일한 `Alias`를 사용할 경우, 응답할 컨테이너를 특정지을 수 없습니다.



#### 연습 문제

##### [연습] `host` 네트워크를 사용하는 컨테이너 만들기

`docker-compose.yaml`을 이용하여 `host`를 사용하는 컨테이너를 실행합니다.

```yaml
# docker-compose.yaml
version: '3.8'
name: 'network-host'

services:
  ubuntu:
    image: ubuntu:22.04
    entrypoint: /bin/bash
    command:
      - -c
      - sleep infinity
    network_mode: host
```

```cmd
$  docker compose up -d
[+] Building 0.0s (0/0)                                                      docker:desktop-linux
[+] Running 1/0
 ✔ Container network-host-ubuntu-1  Created                                                  0.0s 
```



테스트를 위해서 다음과 같이 `nginx` 컨테이너를 실행합니다.

```cmd
$ docker run -d -p 80:80 --name nginx nginx:latest 
```



`ubuntu` 컨테이너에 `curl`을 설치합니다.

```cmd
$ docker exec network-host-ubuntu-1 /bin/bash -c "apt-get update && apt-get upgrade && apt-get install -y curl"
```



다음과 같이 컨테이너 내부에서 `localhost:80`으로 `nginx` 응답이 오는 것을 확인합니다.

```cmd
$ docker exec network-host-ubuntu-1 /bin/bash -c "curl localhost:80"
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



##### [연습] 이미 생성된 네트워크 사용하기

`bridge` 드라이버를 사용하는 네트워크를 생성합니다.

```cmd
$ docker network create private -d bridge
1be27c483205a3b2dfbdf4ae36c2cc20ae0a53bc939eb7bd050a65574a9b9299

$ docker network ls -f name=private      
NETWORK ID     NAME                    DRIVER    SCOPE
1be27c483205   private                 bridge    local
```



다음 `compose`파일을 이용하여 `private` 네트워크를 사용하는 프로젝트를 실행합니다.

```yaml
# docker-compose.yaml
version: '3.8'
name: 'network-external'

services:
  ubuntu:
    image: ubuntu:22.04
    entrypoint: /bin/bash
    command:
      - -c
      - sleep infinity
    networks:
      private:

networks:
  private:
  	name: "private"
    external: true
```

```cmd
$ docker compose up -d                                                                             
[+] Building 0.0s (0/0)                                                                                                                                                                                            docker:desktop-linux
[+] Running 1/1
 ✔ Container network-external-ubuntu-1  Started   
```



다음과 같이 `name`을 제거하여도 프로젝트가 정상적으로 실행됩니다.

```yaml
# docker-compose.yaml
version: '3.8'
name: 'network-external'

services:
  ubuntu:
    image: ubuntu:22.04
    entrypoint: /bin/bash
    command:
      - -c
      - sleep infinity
    networks:
      private:

networks:
  private:
    external: true
```

```cmd
$ docker compose up -d                                                                             
[+] Building 0.0s (0/0)                                                                                                                                                                                            docker:desktop-linux
[+] Running 1/1
 ✔ Container network-external-ubuntu-1  Started   
```



하지만, `name`을 `my-private`로 변경하면 다음과 같이 프로젝트가 실행되지 않는 것을 확인할 수 있습니다.

```yaml
version: '3.8'
name: 'network-external'

services:
  ubuntu:
    image: ubuntu:22.04
    entrypoint: /bin/bash
    command:
      - -c
      - sleep infinity
    networks:
      private:

networks:
  private:
    name: "my-private"
    external: true
```

```cmd
$ docker compose up -d
[+] Building 0.0s (0/0)                                                                                                                                                                                            docker:desktop-linux
network my-private declared as external, but could not be found
```





##### [연습] `alias` 설정하기

다음 `docker-compose.yaml`을 이용하여 프로젝트를 생성합니다.

```yaml
# docker-compose.yaml
version: '3.8'
name: 'network-alias'

services:
  ubuntu:
    image: ubuntu:22.04
    entrypoint: /bin/bash
    command:
      - -c
      - sleep infinity
    networks:
      private:

networks:
  private:
```



`inspect`를 이용하여 컨테이너에 설정된 `alias`를 확인하면 다음과 같이 3개가 표기되는 것을 확인할 수 있습니다.

- 컨테이너 이름
- 서비스 이름
- 컨테이너 ID

```cmd
$ docker inspect network-alias-ubuntu-1 | jq ".[0].NetworkSettings.Networks" | jq -r '.[].Aliases' 
[
  "network-alias-ubuntu-1",
  "ubuntu",
  "d1295a0aae14"
]
```



다음과 같이 `docker-compose.yaml`에  `alias`를 추가하여 업데이트 해줍니다.

```yaml
version: '3.8'
name: 'network-alias'

services:
  ubuntu:
    image: ubuntu:22.04
    entrypoint: /bin/bash
    command:
      - -c
      - sleep infinity
    networks:
      private:
        aliases:
          - server

networks:
  private:
```

```cmd
$ docker compose up -d
```



그러면 다음과 같이 기존에 있던 `alias`에 추가로 설정한 `server`가 추가된 것을 확인해볼 수 있습니다.

```cmd
$  docker inspect network-alias-ubuntu-1 | jq ".[0].NetworkSettings.Networks" | jq -r '.[].Aliases' 
[
  "network-alias-ubuntu-1",
  "ubuntu",
  "server",
  "1679779c1978"
]
```



#### [실습] 생성된 `network`를 이용하여 서로 다른 프로젝트의 서비스와 연결하기

- `across_project` 네트워크를 생성하세요.
- `network-across-project-1` 프로젝트를 생성하세요
  - `ubuntu:22.04`
  - `curl`이 설치되어 있어야합니다.
  - `across_project` 네트워크의 alias를 `main`으로 설정하세요
- `network-across-project-2` 프로젝트를 생성하세요
  - `nginx:latest`
  - `80` 포트를 `expose`
  - `across_project`네트워크의 alias를 `web`으로 설정하세요
- `docker network inspect`를 이용하여 각 서비스의 IP를 확인하세요

```cmd
$ docker network inspect across_project | jq '.[].Containers'
```

- `ubuntu`서버에서 curl을 이용하여 `web`을 호출하세요
  - IP
  - DNS(alias)



### 2.2.6. config & secret

```yaml
services:
  redis:
    image: redis:latest
    configs:
      - http_config
      - source: my_config
        target: /redis_config
        uid: "103"
        gid: "103"
        mode: 0440  # Octal notation
    secrets:
      - source: server-certificate
        target: server.cert
        uid: "103"
        gid: "103"
        mode: 0440  # Octal notation
configs:
  http_config:
    file: ./httpd.conf
secrets:
  server-certificate:
    file: ./server.cert
```



#### Config Vs. Secret

|                 | Config                                | Sercret                                    |
| --------------- | ------------------------------------- | ------------------------------------------ |
| 목적            | 외부에서 서버 설정을 위한 변수를 제공 | 비밀번호와 같은 민감한 데이터를 제공       |
| 기본 Mount 위치 | `/<config-name>`                      | `/run/secrets/<secret-name>`               |
| 암호화 여부     | X                                     | O (`Swarm`을 사용하는 경우)<br />X (그 외) |



#### Config Vs. Mount(Volume)

- 컨테이너 외부에서 파일을 제공한다는 점에서는 두개는 유사합니다.
- `Volume`과는 달리 `Config`는 `Mode`를 이용하여 파일의 권한을 세부적으로 관리할 수 있습니다.



#### 연습 문제

##### [연습] config를 특정 디렉토리에 mount하기

다음 명령어를 이용하여 `password.txt`를 생성합니다.

> 임의의 텍스트 파일을 메모장등을 이용해서 생성해도 무방합니다.

```cmd
# Window - CMD
$ echo %RANDOM% > password.txt 

# Linux
$ openssl rand -base64 14 > password.txt 
```



다음 `docker-compose.yaml`을 이용하여 프로젝트를 실행합니다.

```yaml
# docker-compose.yaml
version: '3.8'
name: 'config-mount'

services:
  ubuntu:
    image: ubuntu:22.04
    entrypoint: /bin/bash
    command:
      - -c
      - sleep infinity
    configs:
      - source: password
        target: /root/web_password.txt

configs:
  password:
    file: password.txt
```



`docker exec`를 통해 `config` 파일을 확인해볼 수 있습니다.

```cmd
$ docker exec config-mount-ubuntu-1 /bin/bash -c "cat /root/web_password.txt && ls -al /root/"  
6M9tLBo4u8EEIbb6Pis=
total 20
drwx------ 1 root root 4096 Dec 28 07:58 .
drwxr-xr-x 1 root root 4096 Dec 28 07:58 ..
-rw-r--r-- 1 root root 3106 Oct 15  2021 .bashrc
-rw-r--r-- 1 root root  161 Jul  9  2019 .profile
-rw-r--r-- 1 root root   21 Dec 28 07:51 web_password.txt
```





