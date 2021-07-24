# Docker 원격으로 이용해 보기

맥북에서 도커를 이용하던 중, 사양의 한계를 느껴 집에 있는 데스크톱을 활용하고 싶다는 생각을 하게 되었고, 여러 방법을 생각해 보다가 '도커 데몬만 데스크톱에 올려두고, 맥북으로는 커맨드만 날릴 수 없을까?'라는 생각을 하게 되었다. 그리고 검색해 본 결과 가능하다는 것을 알게 되었고, 그 내용을 정리하고자 한다.

## 이게 왜 가능할까?

![docker_architcture](https://docs.docker.com/engine/images/architecture.svg)

> 출처: 도커 공식 문서(docs.docker.com)

위 그림은 도커 공식 문서에 올라와 있는 도커의 구조이다. 그림에서 알 수 있듯이, 도커는 **Client / Server(위 그림에서 DOCKER_HOST)**로 이루어져 있다.

일반적으로는 위 그림에서 분리되어 있는 Client와 Server를 한 machine에서 구동하기 때문에 우리 눈에는 REST API를 이용하는 것처럼 보이지도 않고, Client와 Server가 따로 있다는 것도 잘 느껴지지 않지만, **구조상 Client와 Server는 분리되어 있기 때문에 같은 호스트에 존재할 필요도 없고, 따라서 원격지 Server에게 REST API를 통해 명령을 보낼 수도 있다.**

## 설정 방법

### 원격지 Docker daemon 설정

> 우분투를 기준으로 설명한다.

로컬에서 Docker를 이용할 때는 기본적으로 유닉스 소켓을 이용하여 통신하며. 반대로 네트워크를 이용해 통신할 때는 TCP 소켓을 이용한다. 이로 인해 유닉스 소켓을 열어주지 않으면 local에서 Docker command가 제대로 동작하지 않으며, TCP 소켓을 열어주지 않으면 원격에서 Docker command가 동작하지 않는다. 따라서 두 소켓을 다 열어주어야 한다.

#### Docker Daemon CLI(dockerd)로 직접 실행하기

```bash
$ service docker stop
$ sudo dockerd -H unix:///var/run/docker.sock -H tcp://0.0.0.0:2375
```

`-H` 옵션을 통해 Docker daemon을 실행할 떄 소켓 옵션을 설정해 줄 수 있다. 위의 명령어는 차례로 도커 서비스를 중단시키고, 유닉스 소켓과 TCP 소켓을 둘 다 열어 docker daemon을 시작시키는 명령어이다.

#### Docker service 파일 수정하기

도커가 재시작될 떄마다 위의 과정을 반복하려면 번거로울 것이다. service 파일에서 시작 명령어를 수정하여 도커 서비스가 실행될 때 TCP 소켓을 열어둔 채로 시작하도록 설정할 수도 있다!

```no-highlight
...
ExecStart=/usr/bin/dockerd -H unix:///var/run/docker.sock -H tcp://0.0.0.0:2375
...
```

`/lib/systemd/system/docker.service` 파일에서 `ExecStart=`를 찾아 위와 같이 수정하면 된다. 이렇게 하면 Docker service가 시작될 때 유닉스 소켓과 TCP 소켓을 연 채로 Docker daemon이 실행되게 된다.

### 클라이언트 설정

원격지의 Docker에 접속하기 위해서는 클라이언트에서도 설정이 필요하다.

### Docker CLI에 -H 옵션 주기

간단하게, 터미널에서 docker 명령어를 이용할 때 `-H {주소}` 옵션을 추가하여 이용하는 것이다. 예를 들어 `192.168.0.200`에 있는 도커에 docker ps 명령을 이용한다고 하면, `docker -H 192.168.0.200:2375 ps`와 같이 이용하면 되는 것이다.

### 환경변수 이용하기

docker 명령어를 이용할 때마다 -H 옵션을 넣기에는 번거로울 수 있는데, `DOCKER_HOST`라는 환경변수를 설정해 주면 환경변수가 유효한 동안 -H 옵션 없이도 원격으로 docker 명령어를 이용할 수 있다.

### alias를 이용하는 방법

각 터미널(bash, zsh)에서 alias를 직접 등록하는 방식으로 명령어를 이용할 수도 있는데, bash를 예로 들면 ~/.bashrc를 수정해서

```no-highlight
...
alias docker2="docker -H 192.168.0.200:2375"
...
```

와 같이 커스텀 alias를 적용한 뒤 터미널에서 `docker2 ps`처럼 사용할 수도 있다.
