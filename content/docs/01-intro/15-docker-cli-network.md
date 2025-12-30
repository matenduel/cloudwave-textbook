---
title: "Network"
weight: 45
draft: false
---

## 1.6. Docker 네트워크

```
Usage:  docker network COMMAND

Manage networks

Commands:
  connect     Connect a container to a network
  create      Create a network
  disconnect  Disconnect a container from a network
  inspect     Display detailed information on one or more networks
  ls          List networks
  prune       Remove all unused networks
  rm          Remove one or more networks
```



### 네트워크 생성

> https://docs.docker.com/engine/reference/commandline/network_create/

```sh
docker network create [OPTIONS] NETWORK
```



**주요 옵션**

| Option     | Short | Default  | Description                                |
| ---------- | ----- | -------- | ------------------------------------------ |
| `--driver` | `-d`  | `bridge` | 네트워크에서 사용할 드라이버를 선언합니다. |
| `--label`  |       |          | 네트워크의 라벨을 설정합니다.              |



#### 네트워크 드라이버

- `bridge`

`Host` 내에서 외부 노출(`publish`) 없이 컨테이너끼리 통신하기 위해 사용합니다.

동일한 `bridge` 네트워크 내에선 IP 대신 DNS를 통해 컨테이너 이름으로 통신이 가능합니다.

- `host`

컨테이너가 `Host`와 동일한 네트워크를 사용합니다.

네트워크 격리가 이루어지지 않으므로, 충돌 및 보안 문제가 발생할 수 있으므로 `production`에서의 사용은 지양됩니다.

- `none`

`Host`와 완벽히 격리된 네트워크입니다.



**주의사항**

- `host` 네트워크는 인스턴스 별로 한 개만 생성할 수 있습니다.



#### [예시] `bridge` 네트워크 생성하기

```cmd
$ docker network create -d bridge private
```





### 네트워크 목록 보기

>https://docs.docker.com/engine/reference/commandline/network_ls/

```sh
docker network ls [OPTIONS]

Aliases:
  docker network ls, docker network list
```



**주요 옵션**

| Option     | Short | Default | Description                               |
| ---------- | ----- | ------- | ----------------------------------------- |
| `--filter` | `-f`  |         | 지정된 조건에 맞는 네트워크만 표시합니다. |
| `--quiet`  | `-q`  |         | 네트워크 ID만 표기합니다.                 |



#### [예시] `host` 네트워크만 조회하기

```cmd
$ docker network ls -f driver=host
NETWORK ID     NAME      DRIVER    SCOPE
b107e82764b5   host      host      local
```



### 네트워크 정보 보기

> https://docs.docker.com/engine/reference/commandline/network_inspect/

```sh
docker network inspect [OPTIONS] NETWORK [NETWORK...]
```



#### [예시] `private` 네트워크 조회하기

```cmd
$ docker network inspect private
[
    {
        "Name": "private",
        "Id": "04f294f613b357014305f09af1ae31fded48ae663ad451ed65f6c4043045fb9d",
        "Created": "2023-12-23T05:29:53.303432435Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.19.0.0/16",
                    "Gateway": "172.19.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```



### 네트워크 연결

> https://docs.docker.com/engine/reference/commandline/network_connect/

```sh
docker network connect [OPTIONS] NETWORK CONTAINER
```



**주요 옵션**

| Option    | Short | Default | Description                  |
| --------- | ----- | ------- | ---------------------------- |
| `--alias` |       |         | 네트워크 alias를 추가합니다. |
| `--ip`    |       |         | IPv4 주소를 지정합니다.      |



### 네트워크 연결 제거

> https://docs.docker.com/engine/reference/commandline/network_disconnect/

```sh
docker network disconnect [OPTIONS] NETWORK CONTAINER
```



### 네트워크 삭제

> https://docs.docker.com/engine/reference/commandline/network_rm/

```sh
docker network rm NETWORK [NETWORK...]
```

`network rm`은 한 개 이상의 명시된 네트워크들을 삭제하는 명령입니다.



### 모든 네트워크 삭제

> https://docs.docker.com/engine/reference/commandline/network_prune/

```sh
docker network prune [OPTIONS]
```

`network prune`은 **사용하지 않는** 모든 네트워크들을 삭제하는 명령어 입니다.



**주요 옵션**

| Option     | Short | Default | Description               |
| ---------- | ----- | ------- | ------------------------- |
| `--filter` |       |         | 필터링 조건을 설정합니다. |



---

### 연습 문제

#### [연습] `bridge` 네트워크를 이용하여 컨테이너 연결하기

사용할 네트워크를 다음과 같이 생성합니다.

```cmd
$ docker network -d bridge create private
private
```



생성된 네트워크는 `docker network list` 명령어를 통해서 확인할 수 있습니다.

```cmd
$ docker network list
NETWORK ID     NAME      DRIVER    SCOPE
71bf83fc2d7c   bridge    bridge    local
00c55e1a5560   host      host      local
ceedb973ae73   none      null      local
5bce335632dc   private   bridge    local
```



`PostgreSQL`과 `PgAdmin` 컨테이너를 생성합니다. 이 때, `PgAdmin` 컨테이너는 위에서 생성한 네트워크를 사용하여 생성합니다.

```cmd
# PostgreSQL DB 생성
$ docker run --rm -d --name db -e POSTGRES_PASSWORD=mysecretpassword postgres:16.1-bullseye

# PgAdmin Application 생성
$ docker run --rm -d -p 80:80 --name pgadmin -e PGADMIN_DEFAULT_EMAIL=user@sample.com -e PGADMIN_DEFAULT_PASSWORD=SuperSecret --network private dpage/pgadmin4:7.4
```



다음 명령어를 통해서 `db` 컨테이너의 `network`별 IP 주소를 확인합니다.

```cmd
$ docker inspect db -f '{{range $k, $v := .NetworkSettings.Networks}}{{print $k}}={{println $v.IPAddress}}{{end}}'
bridge=172.17.0.3
```



`bridge` 네트워크에 할당된 `IP`를 이용하여 `PgAdmin`에서 연결을 시도합니다.

- `PgAdmin` 페이지에 접속합니다.
    - http://localhost:80

![pgadmin_1]({{< relURL "images/docker/pgadmin_1.png" >}})

- `Servers`를 클릭한 다음, 좌측 상단 `Object > Register > Server`를 클릭합니다.

![pgadmin_2]({{< relURL "images/docker/pgadmin_2.png" >}})



- `General` 탭에서 `Name`을 기입합니다.
- `Connection` 탭에서 접속할 `DB`서버 정보를 입력합니다.
    - host: `bridge` 네트워크에서 할당된 IP
    - Username: `postgres`
    - Password: 컨테이너 실행시 입력한 `POSTGRES_PASSWORD`

![pgadmin_3]({{< relURL "images/docker/pgadmin_3.png" >}})

- `DB` 연결에 실패한 것을 확인할 수 있습니다.



이번엔 위에서 생성한 `private` 네트워크를 `DB`에 연결한 이후, `private` 네트워크에서 할당된 `IP` 주소를 확인합니다.

```cmd
$ docker network connect private db

$ docker inspect db -f '{{range $k, $v := .NetworkSettings.Networks}}{{print $k}}={{println $v.IPAddress}}{{end}}'
bridge=172.17.0.3
private=172.18.0.3

```



해당 `IP`를 이용하여 `PgAdmin`에서 연결의 시도합니다.

![pgadmin_4]({{< relURL "images/docker/pgadmin_4.png" >}})



정상적으로 연결이 되는 것을 확인할 수 있습니다.

![pgadmin_5]({{< relURL "images/docker/pgadmin_5.png" >}})



#### [연습] `alias`를 이용하여 `ip`없이 컨테이너 통신하기

다음과 같이 `ubuntu` 컨테이너를 생성하고 필요한 `Package`를 생성합니다.

```cmd
$ docker run --name main -itd ubuntu:22.04
$ docker exec main /bin/bash -c "apt-get update && apt-get upgrade && apt-get install -y wget dnsutils"
```



`nginx` 컨테이너를 다음과 같이 3개 생성합니다.

```cmd
$ docker run --rm -d --net private --net-alias web_app --name nginx1 nginx:latest
$ docker run --rm -d --net private --net-alias web_app --name nginx2 nginx:latest
$ docker run --rm -d --net private --net-alias web_app --net-alias ready --name nginx3 nginx:latest
```



`main` 컨테이너에서 `web_app`에 대해 `dig`을 사용하면, 다음과 같이 `DNS` 질의에 대한 응답이 없는 것을 확인할 수 있습니다.

> `nslookup web_app`을 이용하여 확인해도 됩니다.

```cmd
$ docker exec main dig web_app

; <<>> DiG 9.18.18-0ubuntu0.22.04.1-Ubuntu <<>> web_app
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 53782
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: bcdc7ef17f1d57fd (echoed)
;; QUESTION SECTION:
;web_app.			IN	A

;; Query time: 5 msec
;; SERVER: 127.0.0.11#53(127.0.0.11) (UDP)
;; WHEN: Sat Dec 23 07:59:12 UTC 2023
;; MSG SIZE  rcvd: 48
```



`main` 컨테이너에 `private` 네트워크를 다음과 같이 연결합니다.

```cmd
$ docker network connect --alias main private main
$ docker inspect main -f "Alias:{{ println .NetworkSettings.Networks.private.Aliases }}IP:{{ println .NetworkSettings.Networks.private.IPAddress }}"
Alias:[main 51c471535b96]
IP:172.19.0.5
```



다시 한번 `main` 서버에서 `dig`를 사용하면 3개 컨테이너의 `IP`가 반환된 것을 확인할 수 있습니다.

```cmd
$ docker exec main dig web_app

; <<>> DiG 9.18.18-0ubuntu0.22.04.1-Ubuntu <<>> web_app
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 43503
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;web_app.			IN	A

;; ANSWER SECTION:
web_app.		600	IN	A	172.19.0.4
web_app.		600	IN	A	172.19.0.2
web_app.		600	IN	A	172.19.0.3

;; Query time: 0 msec
;; SERVER: 127.0.0.11#53(127.0.0.11) (UDP)
;; WHEN: Sat Dec 23 07:51:11 UTC 2023
;; MSG SIZE  rcvd: 94
```



다음과 같이 `ping`을 사용할 때마다 응답하는 `IP`가 바뀌는 것을 볼 수 있습니다.

```cmd
$ docker exec main apt-get install -y iputils-ping
$ docker exec main ping web_app
PING web_app (172.19.0.2) 56(84) bytes of data.
64 bytes from nginx1.private (172.19.0.2): icmp_seq=1 ttl=64 time=0.060 ms
64 bytes from nginx1.private (172.19.0.2): icmp_seq=2 ttl=64 time=0.075 ms
64 bytes from nginx1.private (172.19.0.2): icmp_seq=3 ttl=64 time=0.076 ms
...
$ docker exec main ping web_app
PING web_app (172.19.0.4) 56(84) bytes of data.
64 bytes from nginx3.private (172.19.0.4): icmp_seq=1 ttl=64 time=0.136 ms
64 bytes from nginx3.private (172.19.0.4): icmp_seq=2 ttl=64 time=0.073 ms
...
```



#### [실습]  `DB`를 `alias`를 이용하여 연결하기

> 연습 문제(`bridge` 네트워크를 이용하여 컨테이너 연결하기)에서 `IP` 대신 `DNS`를 이용하세요

- `PostgreSQL`과 `PgAdmin` 컨테이너를 생성합니다
- `private` 네트워크를 `DB`에 연결하면서 `Alias`를 설정합니다.
- `PgAdmin`에서 `IP` 대신 `Alias`를 이용하여 `DB`에 연결합니다. 

