---
title: "Advance"
weight: 30
draft: false
---
## 2.3. Compose file - Advance

### 2.3.1. anchor & Alias

> https://docs.docker.com/compose/compose-file/11-extension/

```yaml
version: '3.8'

x-common:
  &common
  restart: always
  volumes:
    - source:/code
  environment:
    &default-env
    BY: "x-common"

x-value: &v1 x

services:
  ubuntu:
    <<: *common
    image: ubuntu:22.04
    environment:
      <<: *default-env
      FROM: "env definition"
      X: *v1
    entrypoint: /bin/bash
    command:
      - -c
      - echo 'env from ${FROM}' && echo env from $${BY}
    restart: no

volumes:
  source:
```

- `Anchor & Alias`를 이용하여 반복적으로 사용되는 요소들을 모듈화하여 사용할 수 있습니다.
- 재사용을 위한 모듈은 `x-`를 `prefix`로 사용하여야 합니다.
- `<<:`를 사용하면 `yaml`을 병합하는 형태로 사용하게 됩니다.



#### [예시] `Anchor & Alias`를 이용한 `YAML` 파일 확인하기

위에 있는 `docker-compose.yaml`를 대상으로 `docker compose config`를 사용하면,
다음과 같이 `Anchor & Alias`가 반영된 `YAML`을 확인할 수 있습니다.

```cmd
$ docker compose config
WARN[0000] The "FROM" variable is not set. Defaulting to a blank string. 
name: compose
services:
  ubuntu:
    command:
      - -c
      - echo 'env from ' && echo env from $${BY} && echo env from alias $${X}
    entrypoint:
      - /bin/bash
    environment:
      BY: x-common
      FROM: env definition
      X: x
    image: ubuntu:22.04
    networks:
      default: null
    restart: "no"
    volumes:
      - type: volume
        source: source
        target: /code
        volume: {}
networks:
  default:
    name: compose_default
volumes:
  source:
    name: compose_source
x-common:
  environment:
    BY: x-common
  restart: always
  volumes:
    - source:/code
x-value: x
```



### 2.3.2. profile

> https://docs.docker.com/compose/profiles/
>
> https://docs.docker.com/compose/compose-file/15-profiles/

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:16.1-bullseye
    environment:
      - POSTGRES_PASSWORD=mysecretpassword

  server:
    image: ubuntu:22.04
    stdin_open: true # docker run -i
    tty: true        # docker run -t
    depends_on:
      - postgres

  pgadmin:
    image: dpage/pgadmin4:7.4
    environment:
      - PGADMIN_DEFAULT_EMAIL=user@sample.com
      - PGADMIN_DEFAULT_PASSWORD=SuperSecret
    depends_on:
      - postgres
      - server
    profiles:
      - debug
```



#### [예시] `profile`을 이용하여 일부 서비스만 실행하기

```yaml
# docker-compose.yaml
version: '3.8'

services:
  postgres:
    image: postgres:16.1-bullseye
    environment:
      - POSTGRES_PASSWORD=mysecretpassword

  server:
    image: ubuntu:22.04
    stdin_open: true # docker run -i
    tty: true        # docker run -t
    depends_on:
      - postgres

  pgadmin:
    image: dpage/pgadmin4:7.4
    environment:
      - PGADMIN_DEFAULT_EMAIL=user@sample.com
      - PGADMIN_DEFAULT_PASSWORD=SuperSecret
    depends_on:
      - postgres
      - server
    profiles:
      - debug
```

`--profile` 없이 프로젝트를 실행시키면 다음과 같이 `profiles`가 선언되지 않은 2개의 서비스만 실행되는 것을 확인할 수 있습니다.

```cmd
$ docker compose -f docker-compose.yaml up -d
[+] Building 0.0s (0/0)                                        docker-container:multi-arch-builder
[+] Running 4/4
 ✔ Network compose_default         Created                     0.1s 
 ✔ Container compose-postgres-1    Started                     0.1s 
 ✔ Container compose-server-1      Started                     0.1s 
```



`debug` 프로파일을 이용하여 프로젝트를 재실행하면, 기존에 생성되지 않았던 `pgadmin` 서비스가 실행되는 것을 확인할 수 있습니다.

```cmd
$ docker compose -f docker-compose.yaml --profile debug up -d
[+] Building 0.0s (0/0)                                         docker-container:multi-arch-builder
[+] Running 3/3
 ✔ Container compose-postgres-1  Running                        0.0s 
 ✔ Container compose-server-1    Running                        0.0s 
 ✔ Container compose-pgadmin-1   Started                        0.1s 
```



### 2.3.3. deploy

> https://docs.docker.com/compose/compose-file/deploy/

```yaml
deploy:
  resources:
    limits:
      cpus: '0.50'
      memory: 50M
      pids: 1
    reservations:
      cpus: '0.25'
      memory: 20M
  restart_policy:
    condition: on-failure
    delay: 5s
    max_attempts: 3
    window: 120s
  update_config:
    parallelism: 2
    delay: 10s
    order: stop-first
```



#### [예시] 서비스를 컨테이너 3개로 구성하기

다음 `docker-compose.yaml`을 이용하여 프로젝트를 실행합니다.

```yaml
# docker-compose.yaml
name: deploy-replica
services:
  web:
    image: nginx:latest
    expose:
      - 80
    deploy:
      replicas: 3
```

```cmd
$ docker compose up -d
[+] Building 0.0s (0/0)                                                                                      docker:desktop-linux
[+] Running 3/3
 ✔ Container deploy-replica-web-3  Started                                                                                   0.1s 
 ✔ Container deploy-replica-web-2  Started                                                                                   0.1s 
 ✔ Container deploy-replica-web-1  Started                                                                                   0.1s 
```



다음과 같이 `web` 서비스가  `nginx` 컨테이너 3개로 구성되어진 것을 확인할 수 있습니다.

```cmd
$ docker compose -p deploy-replica ps
NAME                   IMAGE          COMMAND                   SERVICE   CREATED          STATUS          PORTS
deploy-replica-web-1   nginx:latest   "/docker-entrypoint.…"   web       47 seconds ago   Up 46 seconds   80/tcp
deploy-replica-web-2   nginx:latest   "/docker-entrypoint.…"   web       47 seconds ago   Up 46 seconds   80/tcp
deploy-replica-web-3   nginx:latest   "/docker-entrypoint.…"   web       47 seconds ago   Up 46 seconds   80/tcp
```



하지만, 다음과 같이 `container_name`을 설정하게되면 실행시 에러가 발생하는 것을 확인할 수 있습니다.

```yaml
# docker-compose.yaml
name: deploy-replica
services:
  web:
    container_name: "web"
    image: nginx:latest
    expose:
      - 80
    deploy:
      replicas: 3
```



```cmd
$ docker compose up -d               
[+] Building 0.0s (0/0)                                                                                      docker:desktop-linux
WARNING: The "web" service is using the custom container name "web". Docker requires each container to have a unique name. Remove the custom name to scale the service.
```



#### [예시] `resource` 할당하기

다음 `docker-compose.yaml`을 이용하여 프로젝트를 실행합니다.

```yaml
# docker-compose.yaml
name: deploy-replica
services:
  web:
    image: nginx:latest
    expose:
      - 80
    deploy:
      replicas: 1
      resources:
        limits:
          memory: 500M
```

```cmd
$ docker compose up -d
[+] Building 0.0s (0/0)                                                                                      docker:desktop-linux
[+] Running 3/3
 ✔ Container deploy-replica-web-1  Started                                                                                   0.6s 
```



`docker stats`를 이용하여 메모리 제한을 확인할 수 있습니다.

```cmd
$ docker stats deploy-replica-web-1
CONTAINER ID   NAME                   CPU %     MEM USAGE / LIMIT   MEM %     NET I/O     BLOCK I/O     PIDS
e07340054e9a   deploy-replica-web-1   0.00%     5.805MiB / 500MiB   1.16%     806B / 0B   0B / 12.3kB   7
```



### 2.3.4. depend_on

> https://docs.docker.com/compose/startup-order/
>
> https://docs.docker.com/compose/compose-file/05-services/#depends_on

```yaml
# docker-compose.yaml
version: '3.8'

services:
  postgres:
    image: postgres:16.1-bullseye
    environment:
      - POSTGRES_PASSWORD=mysecretpassword

  server:
    image: ubuntu:22.04
    stdin_open: true
    tty: true        
    # Short Syntax - List
    depends_on:
      - postgres

  pgadmin:
    image: dpage/pgadmin4:7.4
    environment:
      - PGADMIN_DEFAULT_EMAIL=user@sample.com
      - PGADMIN_DEFAULT_PASSWORD=SuperSecret
    # Long Syntax - Map
    depends_on:
      postgres:
        condition: service_started
        restart: false
      server:
        condition: service_started
    profiles:
      - debug
```



#### Long Syntax

| Attributes  | Description                                                  |
| ----------- | ------------------------------------------------------------ |
| `condition` | 상위 서비스의 상태 조건을 정의합니다. <br />- `service_started`: 서비스가 실행된 상태<br />- `service_healthy`: 서비스가 `healthy`인 상태<br />- `service_completed_successfully`: 서비스가 실행 완료된 상태 |
| `restart`   | 상위 서비스가 업데이트된 경우, 서비스를 재실행합니다.        |
| `required`  | `false`인 경우 상위 서비스가 실행되지 않았더라도 서비스를 실행합니다. |



### 2.3.5. health check

> https://docs.docker.com/engine/reference/builder/#healthcheck

```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost"]
  interval: 1m30s
  timeout: 10s
  retries: 3
  start_period: 40s
  start_interval: 5s
```

- `healthcheck`를 이용하여 서비스의 상태를 확인할 수 있습니다.
- 시간은 `{value}{unit}` 형태로 작성하며, 지원하는 시간 단위는 다음과 같습니다.
    - `us(microseconds)`
    - `ms(milliseconds)`
    - `s(seconds)`
    - `m(minutes)`
    - `h(hours)`



| Attributes       | Description                                                  |
| ---------------- | ------------------------------------------------------------ |
| `test`           | 컨테이너 상태 체크를 위한 명령어를 설정합니다. <br />`NONE`으로 설정한 경우 `Dockerfile`에 정의된 `HEALTHCHECK`를 비활성화 합니다. |
| `interval`       | 상태 체크 간격을 의미합니다.                                 |
| `timeout`        | 응답까지 걸리는 최대 시간을 의미합니다.<br />만약 해당 시간까지 응답이 도착하지 않으면 `fail`로 판단합니다. |
| `start_period`   | 컨테이너 실행까지 소요되는 시간을 의미합니다. <br />해당 시간동안에는 상태가 `fail`인 경우에도 카운트되지 않습니다. |
| `start_interval` | `start_period`내에서의 체크 간격을 의미합니다.               |
| `disable`        | `HEALTHCHECK`를 비활성화 합니다. <br />`test: ["NONE"]`과 동일합니다. |



### 연습문제

#### [연습] Anchor & Alias 이용하여 `yaml`파일 작성하기

```yaml
version: '3.8'

x-common:
  &common
  restart: always
  volumes:
    - source:/code
  environment:
    &default-env
    BY: "x-common"

services:
  ubuntu:
    <<: *common
    image: ubuntu:22.04
    environment:
      <<: *default-env
      FROM: "env definition"
    entrypoint: /bin/bash
    command:
      - -c
      - echo 'env from ${FROM}' && echo env from $${FROM}
    restart: no

volumes:
  source:

```



#### [연습] `depend_on`을 이용하여 `DB` 생성 후 `application` 실행하기

```yaml
# docker-compose.yaml
version: '3.8'

services:
  postgres:
    image: postgres:16.1-bullseye
    environment:
      - POSTGRES_PASSWORD=mysecretpassword

  server:
    image: ubuntu:22.04
    stdin_open: true # docker run -i
    tty: true        # docker run -t
    depends_on:
      - postgres

  pgadmin:
    image: dpage/pgadmin4:7.4
    environment:
      - PGADMIN_DEFAULT_EMAIL=user@sample.com
      - PGADMIN_DEFAULT_PASSWORD=SuperSecret
    depends_on:
      - postgres
      - server
    profiles:
      - debug
```

