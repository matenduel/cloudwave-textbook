---
title: "CLI - Push"
weight: 43
draft: false
---

## 1.4. Docker 이미지 업로드 하기

### 이미지 태그 변경하기

> https://docs.docker.com/engine/reference/commandline/tag/

```shell
docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
```



**주의사항**

- 하나의 Docker 이미지는 여러 개의 `Tag`를 가질 수 있으며, `Tag`는 이미지의 복사본이 아니라 동일한 이미지(`Image ID`)를 가리키는 참조(`reference`)입니다.



#### [예시] 이미지 이름 변경하기

```cmd
$ docker tag ubuntu:22.04 my-ubuntu:v1
$ docker images
REPOSITORY                         TAG             IMAGE ID       CREATED         SIZE
my-ubnutu                          v1              174c8c134b2a   9 days ago      77.9MB
ubuntu                             22.04           174c8c134b2a   9 days ago      77.9MB
```



### 이미지 업로드하기

```
docker push [OPTIONS] NAME[:TAG]
```

**주요 옵션**

| Option       | Short | Default | Description                                 |
| ------------ | ----- | ------- | ------------------------------------------- |
| `--all-tags` | `-a`  |         | Push all tags of an image to the repository |



**주의사항**

- `Docker hub`가 아닌 `ECR` 또는 자체 구축한 `Hub`와 같이 별도의 `registry`에 업로드하려는 경우,
  이름은 `full image name` 형식으로 작성되어야 합니다.
- `Push`전 `docker login`을 통해서 사용할 `registry`에 인증해야 합니다.



#### Full image name 포맷

```tex
[HOST[:PORT_NUMBER]/]PATH
# namespace를 명시하는 경우
[HOST[:PORT_NUMBER]/][NAMESPACE/]REPOSITORY
```



`full image name`의 예시는 다음과 같습니다.

```tex
# Example - ECR
<aws_account_id>.dkr.ecr.ap-northeast-2.amazonaws.com/cloudwave:v1

# Example - private registry
my.registry.com:5000/cloudwave/spring:v1
```



| 컴포넌트    | 설명                                                         |
| ----------- | ------------------------------------------------------------ |
| HOST        | `registry`의 HOST을 의미합니다.                              |
| PORT_NUMBER | `registry` 서버의 포트를 의미합니다.                         |
| NAMESPACE   | 논리적 구분 단위입니다.<br />값이 설정되지 않은 경우, `library`로 설정됩니다. |
| REPOSITORY  | 이미지의 이름을 의미합니다.                                  |



### `Registry` 로그인하기

```shell
docker login [OPTIONS] [SERVER[:PORT]]
```



**주요 옵션**

| Option             | Short | Default | Description                             |
| ------------------ | ----- | ------- | --------------------------------------- |
| `--password`       | `-p`  |         | password                                |
| `--password-stdin` |       |         | `STDIN`을 통해 password를 입력받습니다. |
| `--username`       | `-u`  |         | Username                                |



**주의사항**

- 상황과 목적에 맞는 올바른 계정을 사용하고 있는지 꼭 체크하세요!



---

### 연습 문제

#### [연습] Docker hub에 이미지 업로드하기

`Docker hub`에 로그인합니다.

```cmd
$ docker login -u <USERNAME>
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: <USERNAME>
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```



`ubuntu:22.04` 이미지의 이름을 `<USERNAME>/cloudwave:ubuntu.22.04`로 변경합니다.

```cmd
$ docker tag ubuntu:22.04 <USERNAME>/cloudwave:ubuntu.22.04
```

이미지를 `Push`합니다.

```cmd
$ docker push <USERNAME>/cloudwave:ubuntu.22.04
```



업로드한 이미지는 `search` 명령어를 통해 다음과 같이 확인할 수 있습니다.

```cmd
$ docker search <USERNAME>/cloudwave
NAME                   DESCRIPTION   STARS     OFFICIAL   AUTOMATED
<USERNAME>/cloudwave                  0
```



#### [연습] ECR에 이미지 업로드하기

> - 연습하기에 앞서 AWS IAM 및 ECR 생성이 필요합니다.
>
> - `1.3`에서 제작한 `ubuntu`이미지(`cloudwave:base.v1`)를 사용합니다.

다음 명령어를 통해 `Host`에 설치된 Docker Daemon을 컨테이너에 바인드합니다.

> Volume은 다음 챕터에서 자세히 다룰 예정입니다.

```cmd
# Windows
$ docker run --rm -it -d -v "//var/run/docker.sock://var/run/docker.sock" --name practice cloudwave:docker.v1 /bin/bash
```

컨테이너에서 `docker images` 명령을 실행할 시, 다음과 같이 `host`와 동일한 이미지 목록을 가져오는 것을 확인할 수 있습니다.

```cmd
# Container
$ docker images
REPOSITORY                         TAG             IMAGE ID       CREATED         SIZE
busybox                            latest          9211bbaa0dbd   3 days ago      4.26MB
...

# Host Machine - Windows
$ docker images
REPOSITORY                         TAG             IMAGE ID       CREATED         SIZE
busybox                            latest          9211bbaa0dbd   3 days ago      4.26MB
...
```



`practice` 컨테이너에 접속하여 `aws-cli`를 설치합니다.

```cmd
$ docker run -it -d cloudwave:base.v1
$ docker exec -it practice /bin/bash
root@6ed755e5c292:/scripts# apt install -y awscli
...
Geographic area: 6
...
Time zone: 69
```



`aws-cli`가 설치되었다면 다음 명령어를 통해 버전을 확인할 수 있습니다.

```cmd
$ aws --version
aws-cli/1.22.34 Python/3.10.12 Linux/5.15.133.1-microsoft-standard-WSL2 botocore/1.23.34
```



사용할 AWS 계정을 설정합니다.

- 방법 1 - 환경 변수

```cmd
$ export AWS_ACCESS_KEY_ID=<ACCESS_KEY>
$ export AWS_SECRET_ACCESS_KEY=<SECRET_KEY>
$ export AWS_DEFAULT_REGION=ap-northeast-2
```

- 방법 2 - `aws configure`

```cmd
$ aws configure
AWS Access Key ID [None]: <ACCESS_KEY>
AWS Secret Access Key [None]: <SecretKey>
Default region name [None]: ap-northeast-2
Default output format [None]: json
```



정상적으로 AWS 계정을 설정하였다면, 다음 명령어를 통해서 `ECR private registry`에 로그인할 수 있습니다.

```cmd
$ aws ecr get-login-password --region ap-northeast-2 | docker login --username AWS --password-stdin <AWS_ACCOUNT_ID>.dkr.ecr.ap-northeast-2.amazonaws.com

WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```



업로드할 이미지의 `Tag`를 다음과 같이 설정합니다.

```cmd
$ docker tag cloudwave:base.v1 <AWS_ACCOUNT_ID>.dkr.ecr.ap-northeast-2.amazonaws.com/cloudwave:base.v1
```



만약 ECR Repository 정보(Host)를 모르겠다면,  `aws ecr describe-repositories` 명령어를 통해서 확인할 수 있습니다.

```cmd
# Example
$ aws ecr describe-repositories
{
    "repositories": [
        {
            "repositoryArn": "arn:aws:ecr:ap-northeast-2:300274840224:repository/cloudwave",
            "registryId": "300274840224",
            "repositoryName": "cloudwave",
            "repositoryUri": "300274840224.dkr.ecr.ap-northeast-2.amazonaws.com/cloudwave",
            "createdAt": 1703210889.0,
            "imageTagMutability": "MUTABLE",
            "imageScanningConfiguration": {
                "scanOnPush": false
            },
            "encryptionConfiguration": {
                "encryptionType": "AES256"
            }
        }
    ]
}
```

`ECR`에 이미지를 업로드합니다.

> Docker Hub Public Registry와 다르게 `repository`이름이 일치하지 않는 경우 에러가 발생합니다.

```
docker push <AWS_ACCOUNT_ID>.dkr.ecr.ap-northeast-2.amazonaws.com/cloudwave:base.v1

4293f97defd8: Pushed 
base.v1: digest: sha256:e61678f5601ef33803093805be4ae474105d6f6e86b8d268e8688f5c1c9bad35 size: 529
```



업로드한 이미지는 `AWS Console` 또는 `aws cli`를 통해서 확인할 수 있습니다.

> https://docs.aws.amazon.com/cli/latest/reference/ecr/

```cmd
$ aws ecr list-images --repository-name cloudwave
{
    "imageIds": [
        {
            "imageDigest": "sha256:e61678f5601ef33803093805be4ae474105d6f6e86b8d268e8688f5c1c9bad35",
            "imageTag": "base.v1"
        }
    ]
}
```


