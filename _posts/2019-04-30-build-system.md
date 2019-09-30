---
layout: post
title:  "[C++] CMake Build System"
date:   2019-04-30 09:00:01
categories: programming
---



# 1. Linux Build System

C언어를 처음 배울 때 보통 Visual Studio에서 시작하는 경우가 많다. Visual Studio(이하 VS)는 통합개발환경(IDE)라서 VS에서 코딩도하고 컴파일 빌드 디버깅 실행 모두가 가능하다. 하지만 몇 줄짜리 "hello world" 코드를 빌드하기 위해 IDE가 꼭 필요한 것일까? 당연히 코드 편집 따로 빌드 따로 할 수 있다. 메모장에서 간단한 코드를 작성하고 VS에 설치된 컴파일러를 이용해 command line에서 빌드하는 것도 가능~~하다고~~한다. [링크](<https://docs.microsoft.com/en-us/cpp/build/walkthrough-compiling-a-native-cpp-program-on-the-command-line?view=vs-2019>) 윈도우에서야 초보자건 개발자건 모두가 VS를 쓰지만 **리눅스**에서는 보통 편집기와 컴파일러가 분리되어있다. 리눅스에서의 통합개발환경이란 VS처럼 자체적인 컴파일러까지 포함하는게 아니고 리눅스에 기본 설치된 컴파일러인 **gcc**를 잘 가져다 쓰는 코드 편집기를 말한다. IDE 마다 각자의 방식으로 소스코드 목록과 의존성을 관리하는 프로젝트 파일을 만드는데 소프트웨어를 배포할 때 특정 IDE의 프로젝트 파일을 배포하게 되면 그 소프트웨어를 받아서 쓰는 사람도 특정 IDE를 설치해야 하는 번거로움이 있다. 그래서 리눅스에서 공통적으로 사용하는 빌드 시스템이 있는데 바로  `make`와 `cmake`다. 그래서 소프트웨어을 개발할 때는 편리한 IDE를 쓰고 배포하거나 남의 소프트웨어를 가져다 쓸 때는 command line에서 리눅스 빌드툴을 써서 직접 빌드를 한다. gcc, make, cmake의 관계는 다음과 같다.

- gcc: GNU Compiler Collection의 약자로 C/C++의 컴파일러이다.
- make: 여러 단계의 gcc 명령을 `Makefile` 이라는 스크립트로 만들어 한번에 실행하게 하고 Incremental Build를 지원한다.
- cmake: 중간 단계를 일일이 지정해줘야 하는 복잡한 `Makefile`을 좀 더 편리하게 만들어준다. 이를 위해 `CMakeLists.txt`라는 스크립트를 작성해야 한다.

gcc를 직접 쓰기 불편해서 make가 생겼는데 프로젝트가 복잡해지면 Makefile 작성하기도 어려워서 cmake가 생겼단다. 이번 장에서는 gcc와 make의 간단한 사용 예시를 보고 왜 cmake를 써야하는지 이해한 다음 cmake 사용법에 대해 알아본다.  

본 포스트는 다음 참고자료를 요약하여 작성하였다. 아래 포스트들에서 연결된 추가적인 포스트까지 참고하였다.

- [[Make 튜토리얼] Makefile 예제와 작성 방법 및 기본 패턴](<https://www.tuwlab.com/27193>)
- [[CMake 튜토리얼] 1. CMake 소개와 예제, 내부 동작 원리](<https://www.tuwlab.com/27234>)



## 1.1 gcc and make

예를 들어 다음과 같은 구조를 가진 소스 코드를 빌드하고자 한다. 

![source structure](/ian-flow/assets/2019-04-24-build-system/structure.png)

소스 코드는 다음과 같다.

```cpp
// 터미널에서 실행: gedit main.cpp foo.h foo.cpp bar.h bar.cpp
// bar.h
#ifndef BAR_H
#define BAR_H

#include <iostream>
void bar();

#endif // BAR_H
// bar.cpp
#include "bar.h"
void bar() {
    std::cout << "hello bar\n";
}
// foo.h
#ifndef FOO_H
#define FOO_H

#include <iostream>
void foo();

#endif // FOO_H
// foo.cpp
#include "foo.h"
void foo() {
    std::cout << "hello foo\n";
}
// main.cpp
#include <iostream>
#include "foo.h"
#include "bar.h"
int main()
{
    std::cout << "Hello main\n";
    foo();
    bar();
    return 0;
}
```



이를 빌드하려면 다음 명령어를 실행한다. C언어를 빌드할 때는 `gcc`를 쓰고 C++을 빌드할 때는 `g++`을 쓴다. 아래 명령어는 각각의 `*.c` 파일을 컴파일하여 object 파일(`*.o`)을 만들고 이들을 묶어 실행 파일 `myapp`을 빌드한다.  

```bash
$ g++ -c -o main.o main.cpp
$ g++ -c -o foo.o foo.cpp
$ g++ -c -o bar.o bar.cpp
$ g++ -o myapp main.o foo.o bar.o
$ ./myapp
> Hello main
> hello foo
> hello bar
```

겨우 몇 개의 파일을 결합하여 빌드하는데 네 번의 명령어를 입력해야 하고 더 싫은건 코드를 수정할 때마다 같은 과정을 반복해야 한다는 것이다. 물론 `foo`만 수정하는경우 두 번째와 네 번째 명령어만 쓰면 되지만 어쨌든 매우 번거로운 과정이다. 이 과정을 쉘 스크립트로 작성하는 방법도 있지만 그러면 수정할 때마다 전체 빌드를 다시 하게 되므로 수정사항을 확인하는데 시간이 오래 걸릴 것이다.   

그래서 나온 것이 `make`다. 방금 말한대로 빌드과정을 make의 문법에 맞춰 스크립트로 작성한 후 `make` 명령어를 통해 한 번에 빌드할 수 있다. 위의 `g++` 명령어를 그대로 쓴 것과의 차이점은 Incremental Build가 가능하다는 것이다. 즉 변경한 부분만 알아서 빌드를 해준다. 다음과 같이 `Makefile`을 작성하고 `make` 명령어를 실행해보자.  

```bash
$ rm *.o *.out
$ gedit Makefile
# Makefile 작성
app.out: main.o foo.o bar.o
	g++ -o app.out main.o foo.o bar.o
main.o: foo.h bar.h main.cpp
foo.o: foo.h foo.cpp
bar.o: bar.h bar.cpp
# 닫기
$ make
$ ./myapp
> Hello main
> hello foo
> hello bar
```

자세한 Makefile 문법은 ~~나도 잘 모르니까~~ 넘어가기로 하고 파일들의 의존 관계를 기술했다는 것만 이해하고 넘어가자. 어쨌든 이렇게 Makefile을 만들고 `make`라는 실행하면 빌드가 되는데 중요한 것은 알아서 변경한 부분만 빌드해준다는 것이다. `make`를 실행한 후 한번 더 `make`를 실행해보거나 소스 파일 하나만 수정한 후 `make`를 실행해서 나오는 출력을 보면 이해할 수 있다.   

그러나 이것도 프로젝트 규모가 커지면 Makefile을 관리하는 것도 점점 버거워진다. 일단 소스 파일이 많아지면 중간 결과물 (`*.o`)도 많아지고 파일들의 의존 관계를 코드가 바뀔때마다 수정해주는 것도 번거롭고 빌드 target이 여러 개이거나 프로젝트가 다단계(?)로 이루어진 경우 등 Makefile은 쉽게 방대해지고 관리하기가 어려워진다.   

그래서 우리에게 필요한 것은? **CMake**



## 1.2 CMake

이번엔 cmake로 같은 프로젝트를 빌드해보자. 위 예제에서 cmake를 위한 설정 파일인 `CMakeLists.txt`를 한 줄만 쓰면 빌드가 된다. make와 cmake의 가장 큰 차이는 중간 단계를 생략해도 된다는 것이다. make는 중간 단계에서 생기는 object 파일들을 일일이 지정하고 소스 사이의 의존 관계도 직접 지정해줘야 하지만 cmake는 중간단계는 알아서 만들어주고 소스코드를 분석하여 의존성도 알아서 찾는다. 그래서 소스 파일과 결과물만 지정하면 된다. `ADD_EXCUTABLE()` 함수는 타겟과 그것을 빌드하는데 필요한 소스 파일을 지정한다. cmake에서 **타겟**이란 실행 파일이나 라이브러리 같은 빌드의 최종 결과물을 말한다. cmake는 실행하면 여러 파일과 디렉토리가 자동 생성되므로 `build` 디렉토리를 따로 만들어 그 안에서 실행하자.

```bash
$ rm *.o *.out
$ gedit CMakeLists.txt
# CMakeLists.txt 작성
ADD_EXECUTABLE(myapp main.cpp foo.cpp bar.cpp)
# 닫기
$ mkdir build
$ cd build
$ cmake ..
$ make
$ ./myapp
> Hello main
> hello foo
> hello bar
```

한 줄짜리 `CMakeLists.txt`로 빌드했는데 많은 것들이 생겼다. 각각의 파일을 열어 어떻게 생겼는지 구경해보자. 간단한 프로젝트 인데도 상당히 복잡한 파일들이 생성된 것을 볼 수 있다. 프로젝트가 커져도 온갖 상황에 대처할 수 있게 설계하다보니 기본 설정이 다소 많은듯 하다.

```
CMakeCache.txt
cmake_install.cmake
CMakeFiles/
Makefile
```

`CMakeCache.txt`는 빌드 설정 변수들을 모아놓은 것인데 일반 에디터에서 열어서 수정해도 되지만 `cmake-gui` 툴을 이용하면 편하고 안전하게 수정할 수 있다. 설치는 `sudo apt install cmake-qt-gui`로 하면 된다. 주로 외부 라이브러리 경로를 자동으로 못 찾을 때 이 파일에서 수동으로 지정해준 후 다시 `cmake`를 하면 잘 된다. 사실 **나머지 파일/디렉토리는 사용자가 열어볼 필요가 없다.** 사용자는 일단 `CMakeLists.txt`를 잘 작성하는 것이 중요하다. 위 예시는 간단한 예제였고 실제로는 좀 더 자세한 설정이 필요하다.



# 2 CMake 명령어

cmake의 주요 명령어를 하나씩 알아보면서 `CMakeLists.txt`를 작성해보자.

## 2.1 기본 명령어

### cmake 예약 변수

cmake에는 자동으로 지정된 변수들이 많은데 이들을 잘 알아야 원하는 빌드 설정을 하고 `CMakeLists.txt`도 쉽게 작성할 수 있다. 대부분 `CMAKE_`로 시작하며 `CMakeCache.txt`를 보면 어떤 변수들이 있는지 볼 수 있다. 그 중 중요한 몇 개만 여기에 설명하는데 아래 내용을 보다가 이 변수들이 나오면 의미를 찾아보자.

- `CMAKE_CURRENT_SOURCE_DIR`: 소스 파일들을 찾을 디렉토리 경로로 `CMakeLists.txt`가 있는 경로가 기본 값으로 들어있다. 다른 경로를 설정할 때 이 변수를 기준으로 상대 경로를 만들면 편하다.
- `CMAKE_INSTALL_PREFIX`: `make install` 할때 빌드 결과물이 복사될 경로로 `/usr/local`이 기본 값이다. 자세한 설명은 `install()` 함수 내용을 참조한다.
- `CMAKE_PREFIX_PATH`: `find_package()` 등의 명령에서 외부 프로젝트를 검색할 경로다. 어떤 프로젝트에서 다른 프로젝트의 라이브러리를 사용하고자 할 때 이 변수의 경로에서 하위 디렉토리까지 전부 검색한다.
- `CMAKE_PROJECT_NAME `: `project()`라는 명령어에서 설정할 수 있는 프로젝트의 이름이다.
- `CMAKE_BUILD_TYPE`: 빌드 형상을 지정할 수 있는 변수인데 빌드 형상은 대표적으로 `Debug`와 `Release`가 있다.
- `CMAKE_CXX_COMPILER`: c++컴파일러를 지정할 수 있다. `g++`이 기본이지만 다른 컴파일러나 `g++`의 다른 버전을 선택할 수 있다.
- `CMAKE_CXX_FLAGS`: c++ 컴파일 옵션을 지정한다.



### CMAKE_MINIMUM_REQUIRED() 최소 cmake 버전 확인

외부 라이브러리(3rd party software)를 이용하는 경우 외부 라이브러리의 `*.cmake` 파일에서 빌드 설정을 가져오는 경우가 많은데 이때 서로 `CMakeLists.txt`를 작성할 당시의 cmake 버전이 어느정도 맞아야 작동할 수 있다. 버전이 너무 다르면 cmake의 변수명이나 문법이 조금씩 다를 수 있기 때문이다. 그래서 보통 **`CMakeLists.txt`의 맨 위**에 이 명령어를 쓴다. 아래는 cmake 버전 3.0 이상을 요구하는 명령어다. 현재 시스템에 설치된 cmake가 이보다 낮으면 에러가 발생하게 된다.

```cmake
# cmake_minimum_required(VERSION <x.y.z.w>)
cmake_minimum_required(VERSION 3.0)
```



### PROJECT() 프로젝트 이름 지정

이 명령어를 통해 프로젝트의 이름을 지정하면 `CMAKE_PROJECT_NAME`라는 변수에 저장된다. 이후 프로젝트 이름과 같은 실행 파일이나 라이브러리를 만들 때 이 변수를 사용할 수 있다.

```
# project(<프로젝트 명>)
project("HelloCMake")
```



### MESSAGE() 문자열 출력

cmake를 하는 과정에서 어디까지 진행되었는지 확인하거나 변수에 어떤 값이 들어있는지 출력해서 확인하고자 할 때 쓰인다. 변수 내용을 참조할 때는 `${VAR_NAME}` 형식으로 사용한다. 변수는 따옴표 안팎에서 동일하게 사용된다.

```cmake
# message(<메시지>)
message("project name: ${CMAKE_PROJECT_NAME}")
message("install dir: ${CMAKE_INSTALL_PREFIX}")
```



### SET() 변수 정의

set() 명령어는 cmake에서 가장 많이 쓰이는 명령어로 변수를 정의한다. 값이 하나 일때는 하나만 쓰면 되고 값이 여러 개인 목록일 때는 띄어쓰기로 구분한다.

```cmake
# 변수 정의
# set(<변수명> <값>)
set(CMAKE_CXX_COMPILER g++)
set(CMAKE_BUILD_TYPE Release)
# 목록 변수 정의
# set(<목록 변수명> <항목> <항목> <항목> ...)
set(SOURCE_FILES main.cpp foo.cpp bar.cpp)
set(CMAKE_PREFIX_PATH /usr/lib /usr/local/lib)
message("set vars: ${CMAKE_BUILD_TYPE}, ${SOURCE_FILES}")
```

cmake를 실행해보면 `set vars: Release, main.cpp;foo.cpp;bar.cpp`라는 메시지를 볼 수 있다. 목록변수를 내부적으로는 `;`으로 구분해서 보관하고 있는 것이다. 만약 하나의 값에 공백(띄어쓰기)이 들어있다면 값을 따옴표("")로 묶어주면 된다.



### ADD_EXECUTABLE() 실행 파일 target 추가

빌드 결과물로 생성될 실행 파일과 실행 파일을 빌드하는데 필요한 소스 파일을 지정한다. 굳이 헤더파일을 쓰지 않아도 코드를 분석해서 알아서 헤더파일을 찾아준다. 실행파일의 이름은 `myapp`으로 하고 필요한 소스 파일 목록은 `SOURCE_FILES`라는 변수로 저장했으므로 변수로 대체한다.

```cmake
# add_executable(<실행_파일명> <소스_파일> <소스_파일> ... )
# add_executable(myapp main.cpp foo.cpp bar.cpp)
add_executable(myapp ${SOURCE_FILES})
```

일단 여기까지 쓰고 다시 빌드를 해보자. 자세한 설정이 더 들어갔지만 현재까지 결과는 같다. 지금까지 쓴 `CMakeLists.txt`는 다음과 같다. 한 줄씩 다시 보면서 의미를 떠올려보자.

```cmake
cmake_minimum_required(VERSION 3.0)
project("HelloCMake")
message("project name: ${CMAKE_PROJECT_NAME}")
message("install dir: ${CMAKE_INSTALL_PREFIX}")

set(CMAKE_CXX_COMPILER g++)
set(CMAKE_BUILD_TYPE Release)
set(SOURCE_FILES main.cpp foo.cpp bar.cpp)
set(CMAKE_PREFIX_PATH /usr/lib /usr/local/lib)
message("set vars: ${CMAKE_BUILD_TYPE}, ${SOURCE_FILES}")

add_executable(myapp ${SOURCE_FILES})
```



## 2.2 라이브러리 생성 및 활용

실제로 쓸만한 프로그램을 만들기 위해서는 보통 외부의 라이브러리도 많이 쓰고 직접 라이브러리를 만들기도 한다. 라이브러리를 만들고 활용하는 방법에 대해 알아보자. 우리가 이미 설치한 Qt5라이브러리를 활용하는 새로운 라이브러리 `libmyqt5`와 이 라이브러리를 사용하는 실행 파일 `myqt5_app`을 만들 것이다. 코드는 다음과 같다.

```cpp
// myqt5.h
#ifndef MYQT5_H
#define MYQT5_H

#include <QtCore/QtCore>
void myqt_func(const char* str);

#endif // MYQT5_H

// myqt5.cpp
#include "myqt5.h"
void myqt_func(const char* str) {
	QString qstr = str;
	qDebug() << "myqt5" << qstr;
}

// main.cpp
#include <iostream>
#include "myqt5.h"
int main()
{
	std::cout << "Hello main\n";
	myqt_func("hello myqt5");
    return 0;
}
```





일단 위에서 배운것을 토대로 기본 설정을 해보자. 한 가지 추가된 점은 `project`에서 `VERSION`이란 옵션 변수로 버전을 지정할 수 있다는 것이다.

```cmake
cmake_minimum_required(VERSION 3.0)
# 프로젝트 이름과 버전 지정
project("myqt5" VERSION 1.1)
# 컴파일러 지정
set(CMAKE_CXX_COMPILER g++)
# 빌드 형상 지정: Release
set(CMAKE_BUILD_TYPE Release)
# true로 지정하면 빌드 과정의 모든 메시지 출력
# set(CMAKE_VERBOSE_MAKEFILE true)
# 빌드 결과물을 설치할 경로 지정하고 화면에 출력
set(CMAKE_INSTALL_PREFIX ${CMAKE_CURRENT_SOURCE_DIR}/devel)
message("install prefix ${CMAKE_INSTALL_PREFIX}")
```



### ADD_COMPILE_OPTIONS() 컴파일 옵션 설정

1.2.1 예제에서는 넣지 않았지만 실제로 컴파일을 할 때는 다양한 옵션을 입력해준다. 컴파일 옵션을 입력할 수 있는 함수가 `ADD_COMPILE_OPTIONS()`인데 옵션들을 공백으로 구분하여 넣어주면 된다. 컴파일 옵션을 지정하는 다른 방법은 `CMAKE_CXX_FLAGS`를 이용하는 것이다. 이 변수에 옵션 값을 넣어도 컴파일 옵션에 들어간다.

```cmake
# add_compile_options(<option1> <option2> <option3> ...)
add_compile_options(-Wall -std=c++14 -O2 -fPIC)
# set(CMAKE_CXX_FLAGS "-Wall -std=c++14 -O2 -fPIC")
```

- `-Wall`: 컴파일 과정에서의 모든 warning 화면에 출력
- `-std=c++14`: c++14 사용
- `-O2`: 2단계 컴파일 최적화 사용
- `-fPIC`: 라이브러리를 만들 때 필요한 옵션

다양한 컴파일 옵션에 대한 설명은 아래 링크들을 참조한다.  

- [gcc 옵션 정리](https://devanix.tistory.com/169)
- [컴파일 과정 & gcc 옵션 요약](http://egloos.zum.com/program/v/1373351)
- [GCC -fPIC option](https://stackoverflow.com/questions/5311515/gcc-fpic-option)



### FIND_PACKAGE() 외부 라이브러리 검색

현재 프로젝트에서 사용할 외부 라이브러리를 검색해주는 함수다. 어떤 프로젝트를 하던 대부분 외부 라이브러리를 사용하므로 중요한 함수다. `<package-name>Config.cmake`형식의 이름을 가진 파일을 찾고 그 파일을 cmake로 실행한다. `REQUIRED` 옵션은 빌드에 필수적인 패키지란 뜻이다. `REQUIRED`가 붙었는데 찾지 못하면 에러가 난다. `Makefile`을 만들기 전 의존하는 패키지가 있는지 확인하는 용도로도 많이 쓰인다.  

패키지를 자동으로 찾지 못 한다면 사용자가 경로를 직접 지정해줘야 한다. `find_package(<package_name> PATH <path_to_package>)` 형식으로 검색할 경로를 지정할 수도 있고 그 전에 `CMAKE_PREFIX_PATH`에 경로를 추가해도 된다.

`find_package()`를 실행하면 설정 파일의 디렉토리 경로를 `<package-name>_DIR`이란 변수에 저장하고 설정 파일의 full path를 `<package-name>_CONFIG`이란 변수에 저장한다. 다음은 `Qt5Core` 라이브러리를 찾으면서 생긴 변수들을 출력하는 함수다. 

```cmake
# find_package(<package-name> [REQUIRED])
set(CMAKE_PREFIX_PATH /opt/Qt5.12.3/5.12.3/gcc_64)
find_package(Qt5Core REQUIRED)
message("=== find package(Qt5Core) generated Qt5Core_DIR=${Qt5Core_DIR}, Qt5Core_CONFIG=${Qt5Core_CONFIG}")
message("=== find package(Qt5Core) loaded Qt5Core_INCLUDE_DIRS=${Qt5Core_INCLUDE_DIRS}")
```

메시지 출력 결과는 다음과 같다. `Qt5CoreConfig.cmake`파일을 실행하여 `Qt5Core_INCLUDE_DIRS` 변수를 불러왔다. 이 변수는 이후 헤더파일을 찾는데 사용된다.

> === find package(Qt5Core) generated Qt5Core_DIR=/usr/lib/x86_64-linux-gnu/cmake/Qt5Core, Qt5Core_CONFIG=/usr/lib/x86_64-linux-gnu/cmake/Qt5Core/Qt5CoreConfig.cmake
>
> === find package(Qt5Core) loaded Qt5Core_INCLUDE_DIRS=/usr/include/x86_64-linux-gnu/qt5/;/usr/include/x86_64-linux-gnu/qt5/QtCore;/usr/lib/x86_64-linux-gnu/qt5//mkspecs/linux-g++



### INCLUDE_DIRECTORIES() 헤더 파일 경로 추가

프로젝트 코드에서 `#include <some-header.h>`를 할 때 헤더 파일을 검색할 경로를 추가한다. `gcc` 옵션의 `-I`에 해당한다. 이 프로젝트의 경우 `<QtCore/QtCore>`라는 헤더 파일이 필요한데 `QtCore`라는 디렉토리가 위치한 경로를 `INCLUDE_DIRECTORIES()`로 추가해야 컴파일러가 헤더를 찾을 수 있다.   

Note: 시스템에 설치된 패키지의 경우 `INCLUDE_DIRECTORIES()`를 하지 않아도 찾을 수 있다. `INCLUDE_DIRECTORIES()` 없이 빌드를 해보고 헤더를 찾지 못 하면 그때 추가해도 된다.

```cmake
# include_directories(<path-to-header>)
include_directories(${Qt5Core_INCLUDE_DIRS})
# include_directories("${Qt5Core_DIR}/../../../include")
# include_directories("/opt/Qt5.12.3/5.12.3/gcc_64/include")
```

위 세 개의 명령어는 같은 include 경로를 추가하는데 첫 번째는 `Qt5CoreConfig.cmake`을 실행하면서 생긴 `Qt5Core_INCLUDE_DIRS` 변수를 이용했고 두 번째는 `find_package()`를 하면서 생긴 `Qt5Core_DIR` 변수를 이용했고 세 번째는 수동으로 경로를 입력한 것이다.



### LINK_DIRECTORIES(), LINK_LIBRARIES() 라이브러리 링크

라이브러리 파일(*.so, *.a)을 프로젝트에 링크하는 함수다. 먼저 쓰고자 하는 라이브러리가 위치한 경로를 `LINK_DIRECTORIES()`로 추가해야 라이브러리를 찾을 수 있다. 이후 `LINK_LIBRARIES()`로 링크할 라이브러리를 추가하면 된다.

Note: 시스템에 설치된 패키지의 경우 `LINK_LIBRARIES()`를 하지 않아도 찾을 수 있다. `LINK_LIBRARIES()` 없이 빌드를 해보고 라이브러리를 찾지 못 하면 그때 추가해도 된다.  

Note: 라이브러리 파일명은 보통 `lib<name>.so`로 되어있다. Qt5Core 같은 경우 `libQt5Core.so`이다. `link_libraries()`를 할때는 파일명에서 앞뒤 형식을 빼고 이름만(Qt5Core) 쓴다.

```cmake
# link_directories(<path-to-library>)
# link_libraries(<library-name>)

link_directories("${Qt5Core_DIR}/../..")
# link_directories(/opt/Qt5.12.3/5.12.3/gcc_64/lib)
link_libraries(Qt5Core)
```

`LINK_DIRECTORIES()`도 `INCLUDE_DIRECTORIES()` 와 마찬가지로 `Qt5Core_DIR`로부터 상대 경로를 써도 되고 두 번째처럼 절대 경로를 써도 된다.



### ADD_LIBRARY() 라이브러리 타겟 추가

`ADD_EXECUTABLE()`이 실행파일을 만든다면 `ADD_LIBRARY()`는 라이브러리를 만드는 함수다. 타겟 라이브러리 이름과 소스 파일을 지정하면 된다. 중간에 `SHARED` 혹은 `STATIC` 옵션을 넣을 수 있다.

- SHARED: 동적 라이브러리인 `*.so` 파일을 만든다. 라이브러리의 동적 링크는 라이브러리 파일과 실행파일이 분리된 상태에서 필요에 따라 링크된다는 뜻이다.
- STATIC: 정적 라이브러리인 `*.a`를 만든다. 라이브러리의 정적 링크는 실행 파일에 라이브러리를 포함시킨다는 뜻이다.

```cmake
# add_library(<target-name> [SHARED|STATIC] <sources>)

set(OUTPUT_SHARED_LIB "${CMAKE_PROJECT_NAME}")
set(OUTPUT_STATIC_LIB "${CMAKE_PROJECT_NAME}_stat")
add_library(${OUTPUT_SHARED_LIB} SHARED myqt5.cpp)
add_library(${OUTPUT_STATIC_LIB} STATIC myqt5.cpp)
```

여기서는 동적 라이브러리인 `libmyqt5.so`와 정적 라이브러리인 `libmyqt5_stat.a` 두 개 다 만든다.



### SET_TARGET_PROPERTIES 타겟 속성 지정

시스템에 설치된 라이브러리를 보면 `lib<name>.so.<version>` 형식으로 된 파일들이 많다. 라이브러리 확장자에 버전을 넣고 `install` 할 때 같이 복사되어야 할 헤더 파일을 지정하기 위해 `SET_TARGET_PROPERTIES()`를 사용한다. 이 함수를 통해 타겟의 수십가지 속성을 지정할 수 있는데 그 목록은 [여기](https://cmake.org/cmake/help/v3.14/manual/cmake-properties.7.html#properties-on-targets)서 확인할 수 있다. 여기서는 간단히 버전과 헤더 파일만 다음과 같이 설정한다.

```cmake
set_target_properties(${OUTPUT_SHARED_LIB} PROPERTIES VERSION ${PROJECT_VERSION} PUBLIC_HEADER myqt5.h)
set_target_properties(${OUTPUT_STATIC_LIB} PROPERTIES VERSION ${PROJECT_VERSION} PUBLIC_HEADER myqt5.h)
```



### TARGET_LINK_LIBRARIES() 특정 타겟에 라이브러리 링크

앞서 나온 `LINK_LIBRARIES()`와 기능은 같은데 특정 타겟을 위해서만 라이브러리를 링크한다는 점이 다르다. 여러 타겟에서 공통으로 쓰이는 라이브러리는 `LINK_LIBRARIES()`로 링크하고 특정 타겟에서만 쓰는 라이브러리는 `TARGET_LINK_LIBRARIES()`로 링크하는 것이 좋다. 여기서는 위에서 만든 `libmyqt5.so` 를 사용하는 `myqt5_app`이라는 실행 파일을 생성하였다.

```cmake
# target_link_libraries(<target-name> <library1> <library2> ...)

set(OUTPUT_EXEC "${CMAKE_PROJECT_NAME}_app")
add_executable(${OUTPUT_EXEC} main.cpp)
target_link_libraries(${OUTPUT_EXEC} ${OUTPUT_SHARED_LIB})
```



### INSTALL() 설치 경로 설정

보통 라이브러리를 설치할 때 github에서 라이브러리 소스를 다운로드 받아 `make`를 하여 라이브러리 파일이 만들어 진 후 `make install`을 실행하여 라이브러리를 시스템에 설치한다. 설치란 별게 아니고 라이브러리를 다른 곳에서도 쓸 수 있도록 소스 파일(*.cpp)을 제외한 바이너리와 헤더 파일들을 다른 곳으로 복사하는 것을 말한다. 설치할 경로는 위에서 `CMAKE_INSTALL_PREFIX` 변수로 지정했다. 이 변수를 지정하지 않으면 기본적으로 `/usr/local`에 설치된다. `INSTALL()`은 복사할 파일의 종류에 따라 설치 경로에 대한 상대 경로를 지정한다.

```cmake
# install(TARGETS <target1> <target2> ...
        RUNTIME DESTINATION <executable-dir>
        LIBRARY DESTINATION <shared-lib-dir>
        ARCHIVE DESTINATION <static-lib-dir>
        PUBLIC_HEADER DESTINATION <header-dir>
        )
install(TARGETS ${OUTPUT_SHARED_LIB} ${OUTPUT_STATIC_LIB} ${OUTPUT_EXEC}
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib
        PUBLIC_HEADER DESTINATION include
        )
```

위 함수의 인자들을 하나씩 알아보자

- TARGETS: 설치할 타겟들을 공백으로 구분하여 입력한다.
- RUNTIME DESTINATION: 실행 파일이 복사될 디렉토리를 지정한다.
- LIBRARY DESTINATION: "SHARED" 라이브러리 파일이 복사될 디렉토리를 지정한다.
- ARCHIVE DESTINATION: "STATIC" 라이브러리 파일이 복사될 디렉토리를 지정한다.
- PUBLIC_HEADER DESTINATION: `SET_TARGET_PROPERTIES()` 함수에서 지정한 PUBLIC_HEADER 파일을 복사할 디렉토리를 지정한다.

`INSTALL()` 함수에서 지정한 경로는 모두 `CMAKE_INSTALL_PREFIX`에 대한 상대 경로이다. 파일의 종류에 따라 저장될 디렉토리의 이름은 위와 같이 관습적으로 지정되어 있으니 가급적 저대로 쓰는 것이 좋다.  

---

위와 같은 과정을 거쳐 나온 `CMakeLists.txt`는 다음과 같다. `message()` 함수나 주석(#)은 지워도 된다.  

```cmake
cmake_minimum_required(VERSION 3.0)
project("myqt5" VERSION 1.1)

set(CMAKE_CXX_COMPILER g++)
set(CMAKE_BUILD_TYPE Release)
set(CMAKE_INSTALL_PREFIX ${CMAKE_CURRENT_SOURCE_DIR}/devel)
message("=== install prefix ${CMAKE_INSTALL_PREFIX}")
add_compile_options(-Wall -std=c++14 -O2 -fPIC)

set(CMAKE_PREFIX_PATH /opt/Qt5.12.3/5.12.3/gcc_64)
find_package(Qt5Core REQUIRED)
message("=== find package(Qt5Core) generated Qt5Core_DIR=${Qt5Core_DIR}, Qt5Core_CONFIG=${Qt5Core_CONFIG}")
message("=== find package(Qt5Core) loaded Qt5Core_INCLUDE_DIRS=${Qt5Core_INCLUDE_DIRS}")
include_directories(${Qt5Core_INCLUDE_DIRS})
# include_directories("${Qt5Core_DIR}/../../../include")
# include_directories("/opt/Qt5.12.3/5.12.3/gcc_64/include")
link_directories("${Qt5Core_DIR}/../..")
# link_directories(/opt/Qt5.12.3/5.12.3/gcc_64/lib)
link_libraries(Qt5Core)

set(OUTPUT_SHARED_LIB "${CMAKE_PROJECT_NAME}")
set(OUTPUT_STATIC_LIB "${CMAKE_PROJECT_NAME}_stat")

add_library(${OUTPUT_SHARED_LIB} SHARED myqt5.cpp)
add_library(${OUTPUT_STATIC_LIB} STATIC myqt5.cpp)
set_target_properties(${OUTPUT_SHARED_LIB} PROPERTIES VERSION ${PROJECT_VERSION} PUBLIC_HEADER myqt5.h)
set_target_properties(${OUTPUT_STATIC_LIB} PROPERTIES VERSION ${PROJECT_VERSION} PUBLIC_HEADER myqt5.h)

set(OUTPUT_EXEC "${CMAKE_PROJECT_NAME}_app")
add_executable(${OUTPUT_EXEC} main.cpp)
target_link_libraries(${OUTPUT_EXEC} ${OUTPUT_SHARED_LIB})

install(TARGETS ${OUTPUT_SHARED_LIB} ${OUTPUT_STATIC_LIB} ${OUTPUT_EXEC}
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib
        PUBLIC_HEADER DESTINATION include
        )
```

