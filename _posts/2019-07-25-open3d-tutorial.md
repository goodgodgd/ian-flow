---
layout: post
title:  "[WIP] Introduction to Open3D"
date:   2010-07-25 09:00:01
categories: research
---



# 1. What is Open3D?

Open3D는 3차원 데이터를 다루기 위한 도구들을 모은 라이브러리다. 3차원 데이터란 (RGB-)Depth 영상, point cloud, 3D voxel, mesh 등 다양한 표현들이 있다. Open3D가 제공하는 기능들 크게 보면 다음과 같다.

- 다양한 종류의 3차원 데이터를 위한 자료 구조(클래스) 제공
- 데이터 종류마다 다른 표준 파일 포맷을 읽고 쓰는 기능
- 2차원/3차원 데이터를 시각화(visualization)하는 기능
- 2차원 영상 처리 알고리즘 (필터링 등)
- 3차원 데이터를 위한 다양한 알고리즘
  - Local/Global Registration (ICP 등)
  - Normal Estimation
  - K-D Tree
  - TSDF Volume Integration
  - Pose-graph Optimization

이러한 기능은 원래 PCL(Point Cloud Library)에도 있었다. 하지만 Open3D는 이와 차별화되는 특징들이 있다. 

- 3rd Party 라이브러리에 대한 의존성 최소화: 한마디로 **설치가 쉽다.** PCL을 한 번 빌드하려면 그 전에 최소 대여섯개의 3rd Party 라이브러리를 설치하던지 직접 소스에서 빌드를 하고 PCL과 링크를 시켜줘야 했다. (이 작업에만 하루가 걸린다.) 하지만 Open3D는 3rd Party 라이브러리 수도 적고 필요한 것들은 Open3D 저장소에 포함되어 있다. 그래서 3rd Party를 따로 받을 필요 없이 Open3D 내부에서 함께 빌드하면 된다.
- 가벼움: 3rd Party 라이브러리가 적고 내부 구조를 단순하게 만들어서 라이브러리 설치 파일의 개수나 용량도 적다.
- Python/C++ 지원: Open3D의 공식 문서는 Python 기반으로 설명하고 있다. 내부적으로는 C++로 구현하여 최적화 시켰지만 Python API를 만들어 Python에서 쓰기가 더 쉽다. [(Python Documentation)](http://www.open3d.org/docs/release/) C++ API도 당연히 있지만 이건 설명하는 문서가 없고 API reference와 다양한 예제 파일들을 보며 사용법을 익혀야 한다. [(C++ API reference)](http://www.open3d.org/docs/release/cpp_api/index.html), [(C++ examples)](https://github.com/intel-isl/Open3D/tree/master/examples/Cpp) 참고로 PCL도 Python API가 있다. [(링크)](https://github.com/strawlab/python-pcl)
- C++사용의 편의성: PCL은 C++의 template 문법을 거의 모든 곳에 적용하여 최적화나 확장성은 좋지만 코딩을 하기에는 매우 불편한점이 많다. Open3D는 자주 쓰이는 타입 하나로만 한정하여 코드를 읽고 쓰기가 더 편하다.
- Cross platform: Ubuntu/Windows/MacOS 에서 모두 작동한다. 

2018년 2월에 최초 릴리즈된 프로젝트이기 때문에 오랫동안 개발된 PCL보다는 기능이 적긴하다. (segmentation, 3D keypoint/descriptor 등) 하지만 같은 기능의 알고리즘도 더 최신 알고리즘이 들어갔고 PCL에 없는 Graph Optimization이나 PointNet 등도 지원한다. 오픈 소스로서 앞으로 더 발전할 가능성이 높다.

**여담**  

Open3D 개발사는 인텔(Intel)인데 인텔에서 3차원 센서도 만들면서 관련 소프트웨어도 개발한 것 같다. 여러 사람이 개발했지만 압도적으로 많이 기여한 한 명이 있는데 박재식이라는 한국 사람이다. 카이스트 박사 후 인텔에서 일하다 최근 포스텍으로 갔다고 한다.



# 2. How to Install

Open3D를 Python으로 사용할 경우 단순히 `pip install open3d-python` 만 치면 된다.  

하지만 C++을 사용할 경우 소스코드를 <https://github.com/intel-isl/Open3D> 에서 받아 빌드를 해야한다. OpenCV나 PCL처럼 복잡하지 않고 오래 걸리지 않는다. 아래 가이드는 Ubuntu (16.04 or 18.04)를 기준으로 설명한다.



## 2.1 Build Library


1. 저장소 받기 (**--recursive** 옵션에 주의)

   ```
   git clone --recursive https://github.com/intel-isl/Open3D
   cd Open3D
   ```

2. 의존성 설치

   ```
   sudo ./util/scripts/install-deps-ubuntu.sh
   sudo apt install cmake-qt-gui
   ```

3. cmake-gui 실행

   ```
   mkdir build
   cd build
   cmake-gui ..
   ```

4. "Configure" and "Finish" 클릭

5. "BUILD_xxx" 옵션 세팅

   1. 체크해제: **PYBIND11** and **PYTHON_MODULE**
   2. 체크: **SHARED_LIBS**, **TINYFILEDIALOGS** and **EIGEN3**

6. "CMAKE_INSTALL_PREFIX" 에 라이브러리 설치 경로 지정

7. "Configure", "Generate" 누르고 cmake-gui 나가기

8. 빌드 및 설치

   ```
   make -j3
   make install
   cp lib/* /path/to/install/lib/
   ```



## 2.2 Import Open3D to QtCreator Project

QtCreator에서 프로젝트를 만든 후 `.pro` 파일에 다음 스크립트 추가

```c
INCLUDEPATH += <install path>/open3d/include \
                <install path>/open3d/include/Open3D/3rdparty/fmt/include \
                <install path>/open3d/include/Open3D/3rdparty/Eigen

LIBS += -L<install path>/open3d/lib \
        -lOpen3D
```



# 3. Open3D Tutorial (C++)

이 포스트에서 주로 다루는 것은 Open3D에 대한 C++ API를 사용하는 방법이다. Python으로 사용하는 방법은 이미 공식문서가 너무 잘되어 있어서 필요하면 언제든 보고 따라할 수 있다. [(Python Basic Tutorial)](http://www.open3d.org/docs/release/tutorial/Basic/index.html) 여기서는 공식 문서에서는 다루지 않지만 회사에서 더 많이 쓰이는 C++로 사용하는 방법에 대해 설명하고자 한다. 문서화 되진 않았지만 C++도 [API reference](http://www.open3d.org/docs/release/cpp_api/index.html)와, [예제](https://github.com/intel-isl/Open3D/tree/master/examples/Cpp)를 제공하여 사용법을 알 수 있게 해놓았다. 여기서 다루는 주제는 크게 두 가지다.

1. **File IO and Visualization**: 파일에서 데이터를 읽고 화면에서 확인 후 저장하는 방법을 익힌다.
2. **Point Cloud Registration**: point cloud를 ICP를 통해 정합해보고 RGB 정보를 이용한 ICP도 알아본다.

예제 프로그램은 다음과 같은 Qt GUI를 통해 각 기능을 하나씩 실행해 볼 수 있도록 했다.  

[그림]



## 3.1 File IO and Visualization

OpenCV에서 제공하지 않는 point cloud를 저장하는 `.pcd` 파일 형식이나 mesh를 저장하는 `.ply` 형식을 읽어주는 것만으로도 Open3D는 써볼 가치가 있다. 



###  From Depth Image





### From RGB-Depth Image



