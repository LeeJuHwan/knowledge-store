# Docker Story

<details>

<summary>Properties</summary>

:pencil:2024.08.06

</details>


## How to change the root directory in Docker?

> 왜 도커 저장 경로를 변경 하려고 하는가?

사내 개발 서버는 루트 디렉터리 디스크 스페이스가 8기가로 제한 되어있다. 하지만, data 영역에는 300기가로 할당 되어있기 때문에 도커 파일을 관리하는 저장소를 data 영역으로 설정 하는 작업이 필요했다.


### System 환경

- Amazon Linux
- Docker version: 25.0.6
- example directory name: /data/docker


### Docker service stop

1. 소켓 종료
```bash
sudo systemctl stop docker.socket
```

2. 서비스 종료
```bash
sudo systemctl stop docker.service
```


### Check the Docker root directory

```bash
docker info | grep "Docker Root Dir"
```

example:

한 번도 도커 설정을 건드린 적 없다면 기본 경로는 아래와 같다.

```plaintext
Docker Root Dir: /var/lib/docker
```


### Create the new directory

- 만약 보존해야 할 이미지가 있다면?
    ```bash
    sudo rsync -aP /var/lib/docker /data/docker
    ```

- 새롭게 도커 디렉터리를 구성한다면?
    ```
    mkdir /data/docker
    ```

permission 스스로 잘 해결 하자(본인에게 하는 소리)


### Customizing docker option

```json
{
    "data-root": "/data/docker"
}
```


### Restart

```bash
sudo systemctl start docker
```
