---
layout: post
title:  "Docker with GUI Tutorial (Sep. 2019 Update)"
date:   2019-06-25 09:00:01
categories: tools

---

## Motivation

연구 때문에 [ORB-SLAM](https://github.com/raulmur/ORB_SLAM) 이라는 걸 돌려봐야 했는데 오래된 코드라 테스트 환경이 무려 Ubuntu 12.04 or 14.04로 되어있을 뿐만 아니라 구버전 우분투에서만 작동하는 ROS Hydro or Indigo까지 dependency로 잡혀있어서 **Ubuntu 14.04를 설치해야 하는 상황**이 되었다. 이거 하자고 옛날 우분투를 시스템에 새로 깐다거나 VirtualBox로 깔긴 싫고 이럴 때 쓰라고 만든 **Docker**를 처음 써보기로 했다. 그런데 쓰다보니 GUI도 돌려야해서 본 포스트는 크게 "도커 사용법"과 도커에서 "GUI 사용법" 크게 두 가지 내용을 다룬다.



## Docker 란?

무식한 내가 떠드는 것 보다는 이미 훌륭한 분들이 잘 정리한 블로그를 보는게 나을듯 하다. 특히 "초보를 위한 도커 안내서"는 기본 개념부터 사용법까지 상세하게 잘 정리되어 있으니 초보자라면 처음부터 다 읽어보면 좋다.~~(저걸 다 읽으면 GUI 빼곤 이 포스트를 읽을 필요가 없습니다;;)~~
- [도커란 무엇인가?(WHAT IS DOCKER?)](http://avilos.codes/infra-management/virtualization-platform/docker/what-is-docker/)
- [도커란 무엇인가? / Docker 컨테이너 / Docker 이미지 / LXC / 가상화](http://dev.youngkyu.kr/32)
- [초보를 위한 도커 안내서 - 도커란 무엇인가?](<https://subicura.com/2017/01/19/docker-guide-for-beginners-1.html>)
- [초보를 위한 도커 안내서 - 설치하고 컨테이너 실행하기](<https://subicura.com/2017/01/19/docker-guide-for-beginners-2.html>)
- [초보를 위한 도커 안내서 - 이미지 만들고 배포하기](<https://subicura.com/2017/02/10/docker-guide-for-beginners-create-image-and-deploy.html>)



한 줄 요약: 고정량의 하드디스크와 메모리를 할당하지 않고 컨테이너 안의 격리된 프로세스로 돌아가는 가상 머신   

핵심 개념

> 컨테이너: 컨테이너는 격리된 공간에서 프로세스가 동작하는 기술입니다. 가상화 기술의 하나지만 기존방식과는 차이가 있습니다. 기존의 가상화 방식은 주로 OS를 가상화하였습니다. 우리에게 익숙한 VMware나 VirtualBox같은 가상머신은 호스트 OS위에 게스트 OS 전체를 가상화하여 사용하는 방식입니다. 이 방식은 여러가지 OS를 가상화(리눅스에서 윈도우를 돌린다던가) 할 수 있고 비교적 사용법이 간단하지만 무겁고 느려서 운영환경에선 사용할 수 없었습니다. ... (중략)... 이를 개선하기 위해 프로세스를 격리 하는 방식이 등장합니다. 리눅스에서는 이 방식을 리눅스 컨테이너라고 하고 단순히 프로세스를 격리시키기 때문에 가볍고 빠르게 동작합니다. CPU나 메모리는 딱 프로세스가 필요한 만큼만 추가로 사용하고 성능적으로도 거어어어어의 손실이 없습니다.
> 
> 이미지: 이미지는 컨테이너 실행에 필요한 파일과 설정값등을 포함하고 있는 것으로 상태값을 가지지 않고 변하지 않습니다. 컨테이너는 이미지를 실행한 상태라고 볼 수 있고 추가되거나 변하는 값은 컨테이너에 저장됩니다. 같은 이미지에서 여러개의 컨테이너를 생성할 수 있고 컨테이너의 상태가 바뀌거나 컨테이너가 삭제되더라도 이미지는 변하지 않고 그대로 남아있습니다. ... (중략)... 새로운 서버가 추가되면 미리 만들어 놓은 이미지를 다운받고 컨테이너를 생성만 하면 됩니다. 한 서버에 여러개의 컨테이너를 실행할 수 있고, 수십, 수백, 수천대의 서버도 문제없습니다. [출처](https://subicura.com/2017/01/19/docker-guide-for-beginners-1.html)
> 



### Docker의 장점

도커를 한동안 빡세게 써보니 차암~ 좋은 시스템이다. 

- 다수의 여러가지 운영체제를 효율적으로 동시에 돌릴 수 있다.
- 미리 CPU, storage, memory를 할당하지 않고 여러 시스템이 유연하게 자원을 나눠가진다. 
- 새로운 이미지나 컨테이너를 만드는게 매우 쉽고 빠르고 자원효율적이다.

동시 여러개의 웹서버를 띄우는데 많이 쓰인다지만 나한테는 ROS나 C++ 개발에 특히 유용해 보인다. 파이썬이야 가상환경으로 dependency를 쉽게 관리할 수 있지만 ROS나 C++ 라이브러리의 경우 어쩔수 없이 시스템에 설치해야 하는데 이런 저런 패키지를 설치하다보면 리눅스가 걸레가 되기 마련이라 반년만 지나도 포맷하고 싶은 욕구가 솟아오른다. 호스트 시스템에는 최소한의 패키지만 설치해 놓고 새로운 프로젝트를 시작할 때마다 전용 도커 이미지를 만들면 호스트 시스템을 깔끔하게 관리할 수 있다.



### Docker Image 상속과 공유

도커의 가장 멋진 점은 이미지의 상속과 공유 시스템이다. 내가 만약 Ubuntu 16.04 이미지에 내가 원하는 패키지를 설치한 `imageA`를 만들고자 할때 Ubuntu 16.04 설치부터 하지 않아도 된다. 이미 도커에서 (정확히는 Docker Hub) 제공하는 `Ubuntu 16.04` 이미지가 있고 나는 그 이미지를 다운로드 받은 후 이를 **상속**받아 거기부터 특정 패키지를 설치한 새로운 이미지 `imageA`를 만들수 있다. `imageA`에서 다른 패키지를 추가 설치한 새로운 이미지 `imageB`를 만들고자하면 `imageA`를 상속받아 추가 패키지만 설치하면 된다.  

즉 어떤 이미지를 만들던 바닥부터 설치할 필요없이 기존에 있는 이미지들을 검색 후 자신의 목적에 가장 가까운 이미지를 받아서 바로 쓰거나 아니면 거기에 세부설정을 추가하여 자신에게 꼭 맞는 이미지를 효율적으로 만들어낼 수 있다. 이때 반드시 필요한 것이 **도커 이미지들을 공유할 수 있는 Docker Hub** 시스템이다. <https://hub.docker.com/> 이곳에서 자신에게 필요한 기반 이미지를 찾으면 `docker pull`이란 명령어로 이미지를 다운받을 수 있다. 그리고 이를 상속하여 변경한 새로운 이미지를 Docker Hub에 공유할 수도 있다.  

도커 이미지에서 더욱 놀라운 점은 이미지의 설치과정을 기술한 `Dockerfile`만 공유해도 다른 곳에서 똑같은 이미지를 만들어낼 수 있다는 것이다. Docker Hub에 있는 이미지를 받을 때는 빌드된 이미지 자체를 받지만 github이나 개인간에 도커 이미지를 공유할 때는 이미지를 빌드할 수 있는 텍스트 파일인 `Dockerfile`만 받아서 직접 빌드하면 된다.  

이처럼 이미지의 상속과 공유시스템으로 원하는 이미지를 편리하게 만들어 낼 수 있으니 너도나도 도커를 쓰는것 같다. 다만 초기에 개념을 이해하고 도커 명령어와 `Dockerfile` 작성법을 익히려면 시간이 좀 필요하다.



## Docker 설치

설치 방법은 공식 홈페이지에 잘 나와있다.
[우분투 설치](https://docs.docker.com/v17.09/engine/installation/linux/docker-ce/ubuntu/)
[윈도우 설치](https://docs.docker.com/v17.09/docker-for-windows/install/)

내가 설치한 Ubuntu 18.04 기준으로 요약하면 다음과 같다.
```bash
# remove old versions
$ sudo apt remove docker*
$ sudo apt autoremove

# install required pakages
$ sudo apt update
$ sudo apt install apt-transport-https ca-certificates curl software-properties-common

# add docker's official GPG key
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
# verify the key
$ sudo apt-key fingerprint 0EBFCD88
pub   4096R/0EBFCD88 2017-02-22
      Key fingerprint = 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid                  Docker Release (CE deb) <docker@docker.com>
sub   4096R/F273FCD8 2017-02-22

# add repository
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
		$(lsb_release -cs) stable"
$ sudo apt update
$ sudo apt install docker-ce
```



## Docker 사용법

### 사용자 등록
Docker의 설치 파일이나 이미지, 컨테이너 등이 시스템에서 실제 위치하는 곳은 시스템 경로인 `/var/lib/docker`이다. 그래서 docker 명령어는 `sudo`로 실행해야 한다. 하지만 docker 사용자 그룹에 현재 사용자를 추가하면 `sudo`를 생략해도 된다. Docker 설치 후에는 아래 커맨드를 실행하자. 그리고 **로그아웃이나 시스템 재시작**을 해야 효과가 나타난다.
```bash
# use docker without sudo
sudo usermod -aG docker $USER
```

### Base Image 받기

일단 Docker Hub에서 base image를 받아보자. 예를 들어 Ubuntu 14.04 LTS 이미지를 받고 싶다면 `ubuntu:14.04`를 내려받으면 된다. `ubuntu`는 이미지 이름이고 `14.04`는 tag다. tag를 붙이지 않으면 기본 tag인 `latest`가 자동으로 붙어서 최신 버전이 설치되므로 꼭 정확한 tag를 붙여줘야 한다.
``` bash
# pull base image
docker pull ubuntu:14.04
```



### 이미지 만들기

Base image를 바탕으로 내가 원하는 것이 설치된 새로운 이미지를 만들기 위해서는 `Dockerfile`을 작성 후 `docker build`를 실행해야 한다. 

#### Dockerfile 작성
다음은 Ubuntu 14.04을 기반으로 설치과정을 정의한 `setup.sh`를 실행하는  Dockerfile 이다. 첫 번째 줄만 필수이고 나머진 필요한대로 작성하면 된다.
```
# Ubuntu 14.04를 상속
FROM ubuntu:14.04

# 이미지 빌드 중엔 CLI 환경이 사용자 입력을 받을 수 없는 환경임을 알려줌
ENV DEBIAN_FRONTEND=noninteractive
# 도커 이미지 내부 폴더 만들기
RUN mkdir -p /work/share
WORKDIR /work
COPY ros_setup.sh /work
RUN chmod a+x setup.sh \
	&& ./setup.sh
```

내가 필요한 설치 과정을 모두 `RUN` 명령어를 이용해 작성하자면 Dockerfile이 너무 지저분해질 거 같아서 따로 스크립트로 만들었다. 하지만 보통은 RUN 뒤에 명령어를 길게 쓰는 게 일반적이다. 여기서 쓰인 Dockerfile의 instruction들은 다음과 같다.

- FROM: 상속 받을 base image 지정
- ENV: 환경변수 설정
- WORKDIR: 작업공간 설정, 컨테이너에 들어갔을 때 시작 위치
- COPY \<host path\> \<image path\>: 호스트의 파일 또는 폴더를 이미지로 복사
- RUN \<command\>: 명령어 (command) 실행

Dockerfile을 작성하는 자세한 방법은 아래 링크들을 확인하길 바란다.

- 공식 홈페이지 (기본 명령어 설명 잘 됨): <https://docs.docker.com/develop/develop-images/dockerfile_best-practices/>
- 각 instruction에 대한 한글 설명(추천): <https://rampart81.github.io/post/dockerfile_instructions/>
- 간단하게 쓰는 방법: <https://www.codementor.io/aviaryan/writing-your-first-dockerfile-7e0rjhual>
- 잘 쓰는 방법: <https://rock-it.pl/how-to-write-excellent-dockerfiles/>

#### 이미지 빌드

Dockerfile를 작성했으면 이를 이미지로 빌드해 보자. 여기서도 마찬가지로 이름과 태그를 지정해주자. `-t` 옵션을 주면 tag를 지정할 수 있다. Dockerfile이 있는 위치에서 아래 명령을 실행한다. 완료가 되면 기본 Ubuntu 14.04에 내가 추가한 설치와 설정까지 포함된 **도커 이미지**가 만들어지게 된다.
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
- `-v` 혹은 `--volume`:  `-v <host path>:<container path>:<authority>` 옵션을 주면 호스트의 디렉토리를 컨테이너와 공유할 수 있다. 세 번째 authority 항목은 옵션인데 `ro`를 입력하면 read-only 가 되어 컨테이너에서 디렉토리 내용을 읽기만 가능하고 `rw`는 read-and-write 로서 읽고 쓰기가 가능하다. 지정하지 않으면 `rw`가 기본값이다. <https://docs.docker.com/storage/volumes/#use-a-read-only-volume>
- `--name`: 컨테이너의 이름을 지정한다. 이 옵션을 주지 않으면 도커가 임의로 이름을 짓기 때문에 나중에 컨테이너를 다시 실행할 때 이름을 찾는 번거로움이 있으므로 이름을 지정해주는 것이 좋다. 

옵션들을 조합하여 나는 아래와 같이 컨테이너를 실행하였다. `roscore`가 작동하는지 확인해보고 컨테이너를 종료하고 빠져나올 때는 `exit`를 치면 된다.
```bash
$ docker run --name orbslam -it -v /home/ian/workplace/docker/trusty-ros-full/share:/work/share trusty-ros-full:0.2 bash
root@95793a452c6d:/work# roscore
root@95793a452c6d:/work# exit
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

도커 이미지를 빌드하다가 실패하면 이름이 \<none\>으로 되어있는 ~~찌꺼기~~ 이미지가 남는데 이때 tag되지 않은 이미지 삭제 명령어가 유용하다.

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


### 유의사항 (ROS)

컨테이너 안에서 ROS Indigo를 설치하는데 ROS 서버가 뭔가 작업중인지 자꾸 `apt update`를 할 때마다 `Hash sum mismatch`라는 에러가 난다. 오래된 버전이라 관리를 제대로 안해주는건지... 구글에 "Hash sum mismatch"라고 치면 주로 `rm -rf /var/lib/apt/lists/*` 라든가 `apt clean` 명령어를 쳐보라고 하는데 효과가 없었다. 시스템 내부 문제면 저걸로 해결이 되겠지만 서버자체에 문제가 있으면 해결이 되기를 기다리는 수밖에 없다. 며칠 지나면 문제 없다가 다시 생기기도 하고... 암튼 저걸로 시간을 많이 버렸는데 저 에러가 나면 너무 애쓰지 말고 기다려보길 바란다.



## GUI 사용법

도커가 차암~ 좋긴 좋은데 한가지 아쉬운점은 GUI를 쓸수 없다는 점이다. 기본에 충실한 개발자님들은 그런거 필요없다고 하시겠지만 나같은 날라리 사용자는 `ls` 보다는 노틸러스로 보는게 편하다. 게다가 내가 도커에서 돌리고 싶은 프로그램이 GUI 요소를 포함할 때가 많다. 그래서 다방면으로 구글링한 결과 ROS에서 제공하는 가이드가 다양한 방법을 알려준다.

<http://wiki.ros.org/docker/Tutorials/GUI>



### A. xhost를 이용한 X server 연결

다음은 `ubuntu:18.04` 이미지로부터 GUI가 가능한 `test-gui`라는 컨테이너를 실행하는 커맨드다. 두 커맨드의 순서는 상관없다. `docker run`을 실행하면 컨테이너 내부로 들어가므로 `xhost`를 미리 실행하던가 아니면 `docker run` 이후에 다른 터미널을 열어 `xhost`를 실행해도 된다.

```bash
$ xhost +local:
$ docker run -it \
	--name test-gui \
	--env="DISPLAY" \
	--volume="/tmp/.X11-unix:/tmp/.X11-unix:rw" \
	ubuntu:18.04 bash
```

해석을 해보자. 리눅스의 GUI는 리눅스 커널 자체에서 처리하지 않고 X server를 띄워놓고 이를 통해 화면에 무언가를 표시한다. 그말인즉슨 네트워크에 연결된 다른 컴퓨터의 프로그램도 내 컴퓨터의 X server에 연결만 하면 내 컴퓨터 화면에 띄울수 있다는 소리가된다. ~~사실 자세한 원리는 나도 잘 모른다.~~   

첫 번째 커맨드인 `xhost +local:`는 X server에 대한 로컬 유저의 접근을 허가한다. `xhost`라는 명령어가 무엇인지 찾아봤는데 매뉴얼의 설명은 이렇다.

> The xhost program is used to add and delete host names or user names to the list allowed to make connections to the X server.
>
> xhost는 X server에 접근가능한 호스트 이름이나 유저 이름을 추가하거나 지우는 명령어다.

xhost의 활용법은 이 블로그에 잘 나와있다. <http://egloos.zum.com/potato1004/v/9290287>  

블로그마다 `+local:` 뒤에 root를 붙이라는데도 있고 docker를 붙이라는데도 있는데 원리는 모르지만 내가 실험해보니 `+local:`처럼 아무것도 안써도 되고 뒤에 아무거나 붙여도 상관없이 GUI는 작동한다. `xhost +family:name` 형식으로 지정하는 것인데 family만 local로 지정하면 된다.

두 번째 `docker run~~` 커맨드는 호스트의 `DISPLAY` 환경변수를 컨테이너로 전달하고 호스트의 X11 unix socket을 컨테이너에 마운트한다. ~~무슨 말인지 나도 모르겠다.~~

GUI가 되는지 확인하기 위해 컨테이너에서 파일 탐색기인 nautilus를 설치하고 실행해보자.

```
root@1c8c4b907073:/# apt update
root@1c8c4b907073:/# apt install -y nautilus
root@1c8c4b907073:/# nautilus
```

GUI 테마가 좀 다르긴 하지만 실행되는 것을 볼 수 있을 것이다. 실행하면 /root/.config/nautilus 디렉토리를 만들어달라는 메시지 팝업이 뜨는데 `mkdir -p /root/.config/nautilus`로 디렉토리를 만들어주면 더이상 뜨지 않는다.  

다른 방법도 있는 것 같은데 좀 더 복잡하고 내가 했을때 안되는 것도 있어서 나는 이렇게 쓰고 있다.



### B. OpenGL 어플리케이션 활용

그런데 A의 방법으로 모든 GUI 어플리케이션을 사용할 수 있는 것은 아니다. OpenGL 등의 GPU 하드웨어를 이용하는 어플리케이션은 실행이 안된다. 기본적으로 docker에서 GPU 접근이 안되기 때문이다. OpenGL을 활용한 어플리케이션을 실행해보면 실행되지 않는다.

```
root@1c8c4b907073:/# apt install mesa-utils
root@1c8c4b907073:/# glxgears
libGL error: No matching fbConfigs or visuals found
libGL error: failed to load driver: swrast
X Error of failed request:  BadValue (integer parameter out of range for operation)
  Major opcode of failed request:  155 (GLX)
  Minor opcode of failed request:  3 (X_GLXCreateContext)
  Value in failed request:  0x0
  Serial number of failed request:  45
  Current serial number in output stream:  47
```



#### Update!! (2019.09)

도커 컨테이너에서 GPU 자원에 접근하려면 얼마 전까지는 `nvidia-docker2`를 설치했어야 하는데 이제는 도커 자체에서 NVIDIA GPU에 접근을 할 수 있게 됐다. (아마 8월에 업데이트가 된것 같다.) 도커 버전이 19.03 이상이라면 가능하다. 설치법은 아래 링크에 자세히 설명돼있다.

<https://github.com/NVIDIA/nvidia-docker/wiki/Installation-(Native-GPU-Support)>

그런데 혹시라도 NVIDIA driver가 설치가 안돼있다면 드라이버부터 최신 버전으로 설치한다.

```bash
sudo apt-add-repository ppa:graphics-drivers/ppa
sudo apt remove nvidia*
sudo apt autoremove
sudo ubuntu-drivers autoinstall
```

도커만 설치한다고 되는건 아니고 `nvidia-container-toolkit`을 설치해줘야 한다.

```bash
# Add the package repositories
$ distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
$ curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
$ curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list

$ sudo apt update && sudo apt install -y nvidia-container-toolkit
$ sudo systemctl restart docker
```

과거엔 GPU를 쓰기 위해선 `docker` 대신  `nvidia-docker`를 썼어야 했는데 이제는 `docker run --gpus ~~`처럼 기본 도커의 옵션으로 들어간다.



#### NVIDIA 도커 이미지

모든 도커 이미지에서 GPU를 쓸 수 있는건 아니고 GPU를 쓸 수 있게 설정해놓은 전용 도커 이미지가 있어야 한다. 일반 이미지에서도 `Dockerfile` 을 잘 만들면 가능한 것 같긴하다. ([링크](https://github.com/NVIDIA/nvidia-docker/issues/136)) 그냥 우분투 이미지로 OpenGL을 쓸거라면 NVIDIA에서 제공한 `nvidia/opengl` 이미지를 사용하는 것이 편하다. CUDA를 쓸거라면 `nvidia/cuda` 이미지를 써야한다. 이미지 태그는 여기서 확인할 수 있다.

<https://hub.docker.com/r/nvidia/opengl>

<https://hub.docker.com/r/nvidia/cuda>

<https://hub.docker.com/r/nvidia/cudagl>

Ubuntu 18.04를 기반으로 OpenGL이 가능한 이미지는 `nvidia/opengl:1.1-glvnd-devel-ubuntu18.04` 이다. 다음 명령어로 내려 받을 수 있다.

```
$ docker pull nvidia/opengl:1.1-glvnd-devel-ubuntu18.04
```

이제 컨테이너를 만들어 실행해보자.  `nvidia/opengl` 이미지에 `--gpus` 옵션만 추가하면 된다. 사용한다. 이때도 GUI를 쓰기 위해서는 `xhost +local:` 명령어를 미리 써야 하는데 로그인 후 한 번만 실행하면 된다.

```bash
# xhost +local:
$ docker run --gpus all -it \
	--name test-gl-gui \
	--env="DISPLAY" \
	--volume="/tmp/.X11-unix:/tmp/.X11-unix:rw" \
	nvidia/opengl:1.1-glvnd-devel-ubuntu18.04 bash
```

컨테이너에서 nautilus와 mesa-utils를 설치하고 실행해보자.

```
root@572be97f593a:/# apt update
root@572be97f593a:/# apt install -y nautilus mesa-utils
root@572be97f593a:/# nautilus
root@572be97f593a:/# glxgears
```

당연히 nautilus는 실행될 것이고 지금까지 잘 따라왔다면 glxgears를 실행했을 때 빨녹파의 화사한 톱니바퀴가 돌아가는 것이 보일것이다.

---



지금까지 내가 공부한 도커의 사용법을 정리해보았다. 도커를 익숙하게 쓰기까지 (자세히는 아직 잘 모르지만) 2주 정도의 힘든 시간이 필요했다. 도커에서 소스를 빌드했는데 GUI가 안떠서 에러가 나고 GUI 쓰는 법을 겨우 알아냈더니 OpenGL에서 에러가 나고... 어쨌든 이런저런 문제들을 해결하고 나니 호스트 시스템을 더럽히지 않으면서 깔고 싶은 것을 마음껏 깔아보고 맘에 안들면 지워버릴 수 있게 되어 잘 쓰고 있다. **도커 만세!**

