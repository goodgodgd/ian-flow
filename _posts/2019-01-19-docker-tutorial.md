---
layout: post
title:  "Practice Docker building ORB-SLAM"
date:   2019-01-19 09:00:01
categories: tools

---

## Motivation

연구 때문에 [ORB-SLAM](https://github.com/raulmur/ORB_SLAM) 이라는 걸 돌려봐야 했는데 오래된 코드라 테스트 환경이 무려 **Ubuntu 12.04 or 14.04**로 되어있을 뿐만 아니라 저 우분투에서만 작동하는 **ROS Hydro or Indigo**까지 dependency로 잡혀있어서 Ubuntu 14.04를 설치해야 하는 상황이 되었다. 이거 하자고 우분투를 VirtualBox로 깔긴 싫고 이럴 때 쓰라고 만든 **Docker**를 처음 써보기로 했다.

## Docker 란?
나처럼 무식한 사람이 떠드는 것 보다는 이미 훌륭한 분들이 잘 정리한 블로그를 보는게 나을듯 하다.
- [초보를 위한 도커 안내서 - 도커란 무엇인가?](https://subicura.com/2017/01/19/docker-guide-for-beginners-1.html)
- [도커란 무엇인가?(WHAT IS DOCKER?)](http://avilos.codes/infra-management/virtualization-platform/docker/what-is-docker/)
- [도커란 무엇인가? / Docker 컨테이너 / Docker 이미지 / LXC / 가상화](http://dev.youngkyu.kr/32)

한 줄 요약: 엄청 가벼운 가상 머신 (처럼 느껴지는 무엇?)
핵심 개념:

> 컨테이너: 컨테이너는 격리된 공간에서 프로세스가 동작하는 기술입니다. 가상화 기술의 하나지만 기존방식과는 차이가 있습니다. 기존의 가상화 방식은 주로 OS를 가상화하였습니다. 우리에게 익숙한 VMware나 VirtualBox같은 가상머신은 호스트 OS위에 게스트 OS 전체를 가상화하여 사용하는 방식입니다. 이 방식은 여러가지 OS를 가상화(리눅스에서 윈도우를 돌린다던가) 할 수 있고 비교적 사용법이 간단하지만 무겁고 느려서 운영환경에선 사용할 수 없었습니다. ... (중략)... 이를 개선하기 위해 프로세스를 격리 하는 방식이 등장합니다. 리눅스에서는 이 방식을 리눅스 컨테이너라고 하고 단순히 프로세스를 격리시키기 때문에 가볍고 빠르게 동작합니다. CPU나 메모리는 딱 프로세스가 필요한 만큼만 추가로 사용하고 성능적으로도 거어어어어의 손실이 없습니다.
> 
> 이미지: 이미지는 컨테이너 실행에 필요한 파일과 설정값등을 포함하고 있는 것으로 상태값을 가지지 않고 변하지 않습니다. 컨테이너는 이미지를 실행한 상태라고 볼 수 있고 추가되거나 변하는 값은 컨테이너에 저장됩니다. 같은 이미지에서 여러개의 컨테이너를 생성할 수 있고 컨테이너의 상태가 바뀌거나 컨테이너가 삭제되더라도 이미지는 변하지 않고 그대로 남아있습니다. ... (중략)... 새로운 서버가 추가되면 미리 만들어 놓은 이미지를 다운받고 컨테이너를 생성만 하면 됩니다. 한 서버에 여러개의 컨테이너를 실행할 수 있고, 수십, 수백, 수천대의 서버도 문제없습니다. [출처](https://subicura.com/2017/01/19/docker-guide-for-beginners-1.html)
> 

## Docker 설치

설치 방법은 공식 홈페이지에 잘 나와있다.
[우분투 설치](https://docs.docker.com/v17.09/engine/installation/linux/docker-ce/ubuntu/)
[윈도우 설치](https://docs.docker.com/v17.09/docker-for-windows/install/)

내가 설치한 Ubuntu 18.04 기준으로 요약하면 다음과 같다.
```bash
# remove old versions
$ sudo apt-get remove docker docker-engine docker.io

# install required pakages
$ sudo apt-get update
$ sudo apt-get install apt-transport-https ca-certificates curl software-properties-common

# add docker's official GPG key
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
# verify the key
$ sudo apt-key fingerprint 0EBFCD88
pub   4096R/0EBFCD88 2017-02-22
      Key fingerprint = 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid                  Docker Release (CE deb) <docker@docker.com>
sub   4096R/F273FCD8 2017-02-22

# add repository
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
$ sudo apt-get update
$ sudo apt-get install docker-ce
```

## Docker 사용법

### 사용자 등록
Docker의 설치 파일이나 이미지, 컨테이너 등이 시스템에서 실제 위치하는 곳은 시스템 경로인 `/var/lib/docker`이다. 그래서 docker 명령어는 `sudo`로 실행해야 한다. 하지만 docker 사용자 그룹에 현재 사용자를 추가하면 `sudo`를 생략해도 된다. Docker 설치 후에는 아래 커맨드를 실행하자.
```bash
# use docker without sudo
sudo usermod -aG docker $USER
```

### Base Image 받기

일단 Docker Hub에서 base image를 받아보자. `ubuntu:14.04`에서 `ubuntu`는 이미지 이름이고 `14.04`는 tag다. tag를 붙이지 않으면 기본 tag인 `latest`가 자동으로 붙어서 최신 우분투 버전이 설치되므로 꼭 정확한 tag를 붙여줘야 한다.
``` bash
# pull base image
sudo docker pull ubuntu:14.04
```

### 이미지 생성 계획

Ubuntu 14.04에 ROS Indigo를 설치하고 그 다음에 ORB SLAM을 빌드할 것이다. 나는 ROS Indigo를 설치하는 것 까지는 도커 이미지로 만들고 ORB SLAM을 빌드하는 것은 컨테이너 내부에서 실행 할 것이다.  
만약 ORB SLAM 소스를 빌드하는 과정까지 도커 이미지 빌드 과정에 넣어버리면 ORB SLAM을 빌드하는 과정에서 생긴 오류를 수정해서 새로운 이미지를 빌드 할 때마다 ROS Indigo 설치하는 것을 기다려야 하기 때문이다.  
Base image를 바탕으로 내가 원하는 것이 설치된 새로운 이미지를 만들기 위해서는 `Dockerfile`을 작성해야 한다. 

### trusty-ros-full 이미지 만들기

#### Dockerfile 작성
다음은 Ubuntu 14.04에 ROS Indigo를 설치하기 위한 Dockerfile 이다. 
```
FROM ubuntu:14.04

ENV DEBIAN_FRONTEND=noninteractive
RUN mkdir -p /cd /work/share
WORKDIR /work
COPY ros_setup.sh /work
RUN chmod a+x ros_setup.sh \
	&& ./ros_setup.sh
```

너무 간단하지 않은가? 사실 ROS를 설치하는 과정은 모두 [ros_setup.sh](https://github.com/goodgodgd/orbslam-docker/blob/master/trusty-ros-full/ros_setup.sh)에 들어있다. 이걸 모두 `RUN` 명령어를 이용해 작성하자면 Dockerfile이 너무 지저분해질 거 같아서 따로 스크립트로 만들었다. 여기서 쓰인 Dockerfile의 키워드들은 다음과 같다.

- FROM: 상속 받을 base image 지정
- ENV: 환경변수 설정
- WORKDIR: 작업공간 설정, 컨테이너에 들어갔을 때 시작 위치
- COPY <host path> <image path>: 호스트의 파일 또는 폴더를 이미지로 복사
- RUN <command>: 명령어 실행

Dockerfile을 작성하는 자세한 방법은 아래 링크들을 확인하길 바란다.

- 공식 홈페이지 (기본 명령어 설명 잘 됨): https://docs.docker.com/develop/develop-images/dockerfile_best-practices/
- 각 키워드에 대한 한글 설명(추천): https://rampart81.github.io/post/dockerfile_instructions/
- 간단하게 쓰는 방법: https://www.codementor.io/aviaryan/writing-your-first-dockerfile-7e0rjhual
- 잘 쓰는 방법: https://rock-it.pl/how-to-write-excellent-dockerfiles/

#### 이미지 빌드

Dockerfile를 작성했으면 이를 이미지로 빌드해 보자. 여기서도 마찬가지로 이름과 태그를 지정해주자. `-t` 옵션을 주면 tag를 지정할 수 있다. Dockerfile이 있는 위치에서 아래 명령을 실행한다.
```bash
# Usage: docker build -t <image name>:<tag> <Dockerfile dir>
$ docker build -t ros-trusty-full:0.1 .
```

### 컨테이너 생성 및 실행

이미지가 생성되었으면 이를 구현한 컨테이너를 실행해 보자. 컨테이너를 만들어 터미널을 여는 기본 명령은 다음과 같다.
```bash
docker run --name <container name> -it <image name>:<tag>  bash
```

`docker run` 옵션에 대해 알아보자. 자세한 내용은 [이곳](http://blog.naver.com/PostView.nhn?blogId=alice_k106&logNo=220340499760)에도 설명이 잘 되어있다.
- `-i`: interactive의 약자로 docker에서 실행한 결과를 호스트로 출력해 준다.
- `-t`: tty의 약자로 터미널을 쓰게 해준다. `-i`와 합쳐서 `-it` 혹은 `-ti`로 써도 된다.
- `-v`: volume의 약자로 `-v <host path>:<container path>` 옵션을 주면 호스트의 디렉토리를 컨테이너와 공유할 수 있다.
- `--name`: 컨테이너의 이름을 지정한다. 이 옵션을 주지 않으면 도커가 임의로 이름을 짓기 때문에 나중에 컨테이너를 다시 실행할 때 이름을 찾는 번거로움이 있으므로 이름을 지정해주는 것이 좋다. 

옵션들을 조합하여 나는 아래와 같이 컨테이너를 실행하였다. `roscore`가 작동하는지 확인해보고 컨테이너를 종료하고 빠져나올 때는 `exit`를 치면 된다.
```bash
$ docker run --name orbslam -it -v /home/ian/workplace/docker/trusty-ros-full/share:/work/share trusty-ros-full:0.2 bash
root@95793a452c6d:/work# roscore
root@95793a452c6d:/work# exit
$
```

### 도커 이미지 및 컨테이너 관리

도커를 사용하다보면 이미지나 컨테이너를 많이 만들고 지우게 된다. 지우지 않고 쓰다보면 어느순간 큰 용량을 차지하고 있으므로 관리를 해줘야 한다.

#### 이미지 관리

도커 이미지들을 관리하는 명령어들이다.   
~~일상에서 말하는 [이미지 관리](http://www.wikitree.co.kr/main/news_view.php?id=165057)와는 다르다.~~
```bash
# 이미지 리스트 보기
$ docker image list
# 이미지 삭제
$ docker rmi <image name>:<tag>
# tag되지 않은 이미지 삭제
$ docker rmi $(docker images | grep "^<none>" | awk "{print $3}")
```

도커 이미지를 빌드하다가 실패하면 이름이 <none>으로 되어있는 ~~찌꺼기~~ 이미지가 남는데 이때 tag되지 않은 이미지 삭제 명령어가 유용하다.

#### 컨테이너 관리

```bash
# 컨테이너 목록 보기
$ docker ps -a
# 정지된 컨테이너 재실행
$ docker start <container name>
# 실행중인 컨테이너의 bash 실행(접속)
$ docekr exec -it <container name> bash
# 실행중인 컨테이너 정지
$ docker stop <container name>
# 컨테이너 삭제
$ docker rm <container name>
# 휴면중인 컨테이너 삭제
$ docker container prune -f
```

## ORB SLAM 빌드 및 실행

### ORB SLAM 빌드

[GitHub](https://github.com/goodgodgd/orbslam-docker)를 보면 ORB SLAM의 소스코드가 `trusty-ros-full/share/ORB_SLAM` 폴더에 있는 것을 볼 수 있다. 그리고 [`trusty-ros-full/share/orb_setup.sh`](https://github.com/goodgodgd/orbslam-docker/blob/master/trusty-ros-full/share/orb_setup.sh)는 빌드에 필요한 명령어들을 모아놓았다. 이제 도커를 실행하고 스크립트를 실행하면 된다.

```bash
$ docker exec -it orbslam bash
/work# cd share
/work/share# chmod a+x orb_setup.sh
/work/share# ./orb_setup.sh
```

### ORB SLAM 실행

ORB SLAM은 ROS를 기반으로 작동하므로 작동하기 위해서는 세 개의 터미널이 필요하다: 1) roscore 실행, 2) ORB SLAM 실행, 3) rosbag 실행  
`docker exec -it orbslam bash`를 세 개의 터미널에서 실행하고 각 터미널에서 다음 명령어들을 순서대로 실행해야 한다. Bag파일은 [ORB SLAM GitHub](https://github.com/raulmur/ORB_SLAM)에서 받을 수 있다. 이 파일을 받아서 share 폴더에 넣으면 컨테이너 내부에서도 접근할 수 있다.
```bash
Terminal1# roscore
Terminal2# rosrun ORB_SLAM ORB_SLAM Data/ORBvoc.txt Data/Settings.yaml
Terminal3# rosbag play <path/to/bag file>
```

## 주의사항

1. 내가 만든 [GitHub](https://github.com/goodgodgd/orbslam-docker)에 들어있는 ORB SLAM은 원본에 수정을 가하였다.(내가 수정중이면 심지어 빌드 에러가 날 수도 있다 -_-;) 원본을 쓰고 싶으면 [ORB SLAM GitHub](https://github.com/raulmur/ORB_SLAM)에서 소스를 받아 `share/ORB_SLAM` 폴더를 교체하면 된다. 빌드는 똑같이 `orb_setup.sh` 스크립트를 사용하면 된다.

2. ROS Indigo 서버가 뭔가 작업중인지 자꾸 `apt update`를 할 때마다 `Hash sum mismatch`라는 에러가 난다. 오래된 버전이라 관리를 제대로 안해주는건지... 구글에 "Hash sum mismatch"라고 치면 주로 `rm -rf /var/lib/apt/lists/*` 라든가 `apt-get clean` 명령어를 쳐보라고 하는데 효과가 없었다. 시스템 내부 문제면 저걸로 해결이 되겠지만 서버자체에 문제가 있으면 해결이 되기를 기다리는 수밖에 없다. 며칠 지나면 문제 없다가 다시 생기기도 하고... 암튼 저걸로 시간을 많이 버렸는데 저 에러가 나면 너무 애쓰지 말고 기다려보길 바란다.

