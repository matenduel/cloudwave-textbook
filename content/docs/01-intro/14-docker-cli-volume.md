---
title: "Volume"
weight: 44
draft: false
---
## 1.5. Docker 볼륨

```
Usage:  docker volume COMMAND

Manage volumes

Commands:
  create      Create a volume
  inspect     Display detailed information on one or more volumes
  ls          List volumes
  prune       Remove all unused local volumes
  rm          Remove one or more volumes
```

- 컨테이너에서 사용 및 관리하는 저장 공간입니다.
- 컨테이너에 종속되어 있지 않으므로, 컨테이너의 라이프 사이클을 따라가지 않습니다.



### 볼륨 생성

> https://docs.docker.com/engine/reference/commandline/volume_create/

```shell
docker volume create [OPTIONS] [VOLUME]
```



**주요 옵션**

| Option   | Short | Default | Description        |
| -------- | ----- | ------- | ------------------ |
| `--name` |       |         | 이름을 지정합니다. |



### 볼륨 목록 보기

> https://docs.docker.com/engine/reference/commandline/volume_ls/

```shell
docker volume ls [OPTIONS]
```



**주요 옵션**

| Option     | Short | Default | Description                           |
| ---------- | ----- | ------- | ------------------------------------- |
| `--filter` | `-f`  |         | 지정된 조건에 맞는 볼륨만 표시합니다. |
| `--quiet`  | `-q`  |         | 볼륨 이름만 표시합니다.               |



#### [예시] 사용하지 않는 볼륨 목록 보기

```cmd
$ docker volume ls -f "dangling=true"  # or "dangling=1"
DRIVER    VOLUME NAME
local     fecea4ffd139a0d85b735f9ea8fe247bfabc9227df31ac9ff1d8e66f6b77d229
```



### 볼륨 정보 보기

> https://docs.docker.com/engine/reference/commandline/volume_inspect/

```shell
docker volume inspect [OPTIONS] VOLUME [VOLUME...]
```



#### [예시] 볼륨 생성 일자 확인하기

```cmd
$ docker volume inspect --format "{{ .CreatedAt }}" <VOLUME_NAME>
2023-12-22T01:23:07Z
```



### 볼륨 삭제

> https://docs.docker.com/engine/reference/commandline/volume_rm/

```shell
docker volume rm [OPTIONS] VOLUME [VOLUME...]
```

`volume rm`은 한 개 이상의 명시된 볼륨들을 삭제하는 명령입니다.



### 모든 볼륨 삭제

> https://docs.docker.com/engine/reference/commandline/volume_prune/

```sh
docker volume prune [OPTIONS]
```

`volume prune`은 사용하지 않는 모든 `anonymous ` 볼륨을 삭제하는 명령어 입니다.



**주요 옵션**

| Option     | Short | Default | Description                           |
| ---------- | ----- | ------- | ------------------------------------- |
| `--all`    | `-a`  |         | 사용하지 않는 모든 볼륨을 삭제합니다. |
| `--filter` |       |         | 필터링 조건을 설정합니다.             |



---

### Practice

#### [연습] Volume에 DB 데이터 저장하기

>- `postgres:16.1-bullseye` 이미지를 사용합니다.
>- 컨테이너의 이름은 `psql_db`로 설정합니다.
>- 컨테이너 생성시 다음 환경변수가 설정되어야 합니다.
> - POSTGRES_PASSWORD
>
>- 생성한 Volume은 컨테이너의 `/var/lib/postgresql/data`에 마운트합니다.



`DB`에서 사용할 볼륨을 다음과 같이 생성합니다.

```cmd
$ docker volume create db_data
db_data
```



생성된 볼륨은 `docker volume list` 명령어를 통해서 확인할 수 있습니다.

```cmd
$ docker volume list
DRIVER    VOLUME NAME
local     db_data
```



`PostgreSQL`의 데이터는 `/var/lib/postgresql/data`에 저장됩니다.
그러므로, `db_data` 볼륨을 해당 디렉토리에 마운트합니다.

```cmd
$ docker run --rm -d --name psql_db -v db_data:/var/lib/postgresql/data -e POSTGRES_PASSWORD=mysecretpassword postgres:16.1-bullseye
```



컨테이너 터미널에 접근하여 `psql`을 이용해 DB에 접속합니다.

```cmd
$ docker exec -it psql_db /bin/bash
root@7523c983f729:/# psql -U postgres
psql (16.1 (Debian 16.1-1.pgdg110+1))
Type "help" for help.
```



SQL을 이용하여 테이블을 생성합니다.

```sql
# SQL
CREATE TABLE IF NOT EXISTS cloud_wave (
    id SERIAL PRIMARY KEY,
    timestamp timestamp
);
```



생성한 테이블을 `\dt`를 이용하여 확인합니다.

```cmd
postgres=# \dt
               List of relations
 Schema |    Name    | Type  |  Owner
--------+------------+-------+----------
 public | cloud_wave | table | postgres
(1 row)
```



DB를 종료합니다.

```cmd
$ docker stop psql_db
psql_db

# 컨테이너 이름에 `psql_db`가 포함된 컨테이너 목록을 보여줍니다. 
$ docker ps -a -f name=psql_db
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```



컨테이너를 재생성 합니다.

```cmd
$ docker run --rm -d --name psql_db -v db_data:/var/lib/postgresql/data -e POSTGRES_PASSWORD=mysecretpassword postgres:16.1-bullseye
```



컨테이너 내부에서 DB 테이블이 남아있는 것을 확인합니다.

```cmd
$ docker exec -it psql_db /bin/bash
root@3f19c9efb04b:/# psql -U postgres
psql (16.1 (Debian 16.1-1.pgdg110+1))
Type "help" for help.

# Table 목록
postgres=# \dt
               List of relations
 Schema |    Name    | Type  |  Owner
--------+------------+-------+----------
 public | cloud_wave | table | postgres
(1 row)
```



#### [연습] `bind mount`를 사용하여 소스코드 변경하기

> - Host의 `./app`는 컨테이너의 `/code/app`와 바인드 되어야 합니다.
> - 이미지 이름은 `was`로 설정합니다.

폴더 구조는 다음과 같습니다.

```
.
├── app
│   ├── __init__.py
│   └── main.py
├── Dockerfile
└── requirements.txt
```



**main.py**

```python
from typing import Union

from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def read_root():
    return {"Hello": "World"}
```



**Dockerfile**

```
FROM python:3.9

WORKDIR /code

COPY ./requirements.txt /code/requirements.txt

RUN pip install --no-cache-dir --upgrade -r /code/requirements.txt

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "80", "--reload"]
```



**requirements.txt**

```tex
fastapi[standard]>=0.113.0,<0.114.0
pydantic>=2.7.0,<3.0.0
```



`Dockerfile`이 위치한 폴더에서 다음 명령어를 실행하여 이미지를 빌드합니다.

```cmd
$ docker build -t was:fast.1 .
```



다음 명령어를 통해서 `FastAPI` 서버를 실행합니다.

> 사용하는 환경이 Window면 \ 를 사용하고 Linux 또는 WSL이라면 / 를 사용합니다.

```cmd
$ docker run --name bind -p 80:80 -v <FOLDER_PATH>\app:/code/app was:fast.1
INFO:     Will watch for changes in these directories: ['/code']
INFO:     Uvicorn running on http://0.0.0.0:80 (Press CTRL+C to quit)
INFO:     Started reloader process [1] using statreload
INFO:     Started server process [8]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
```



`http://localhost/redoc`에 접속하여 API 목록을 확인합니다.

![fastapi_redoc_1]({{< relURL "images/docker/fastapi_redoc_1.png" >}})



**File 업데이트 하기**

`main.py`를 다음과 같이 수정합니다.

```python
import socket
from typing import Union

from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def read_root():
    return {"Hello": "World"}


@app.get("/hostname")
def get_hostname():
    return {"name": socket.gethostname()}
```



`http://localhost/redoc`에 접속하면 `get_hostname` API가 추가된 것을 확인할 수 있습니다.

![fastapi_redoc_2]({{< relURL "images/docker/fastapi_redoc_2.png" >}})

