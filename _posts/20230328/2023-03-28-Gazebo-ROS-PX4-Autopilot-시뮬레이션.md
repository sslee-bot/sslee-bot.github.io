---
title: Gazebo ROS PX4 Autopilot 시뮬레이션
# date: 2022-11-05 13:30:00 +/-TTTT
categories: [PX4 Autopilot]
tags: [ROS, ROS1, Gazebo, PX4, PX4 Autopilot, MAVROS, C++, Simulation]
---

Gazebo에서 PX4 Autopilot 을 시뮬레이션하는 방법을 소개한다.
[PX4 User Guide](https://docs.px4.io/main/en/) 안에 포함된 내용 중에서 중요하거나 보충 설명이 필요한 것 위주로 설명한다.

> 기본적인 Offboard 컨트롤 시뮬레이션 방법만을 다루었으며, QGroundControl 관련 내용은 다루지 않았다.
{: .prompt-warning }

## Prerequisite
PC 세팅 환경은 다음과 같다.

* OS: Ubuntu 20.04
* ROS: ROS1 Noetic
* Gazebo version: 11.11

ROS 및 Gazebo 는 설치된 상태라고 가정한다. (`ros-noetic-desktop-full`)

PX4 Autopilot 사용 시의 Offboard control 에 대한 전체적인 이해가 필요하므로 관련 가이드 문서[^offboard-control-guide]를 참고하는 것이 좋다.

## ROS-Gazebo 시뮬레이션
PX4 User Guide 에서는 시뮬레이션 시 ROS 와 Gazebo 를 연동하여 진행하는 방식을 추천하고 있다.
특히 Gazebo classic 을 사용한 시뮬레이션 방법은 따로 문서[^gazebo-classic-sim]가 정리되어있다.

위 링크에서 전체적인 설정 순서가 설명되어 있지만, 기본적인 시뮬레이션을 위해 필요한 내용은 크게 다음 세 가지이다.

* PX4-Autopilot 패키지 클론(clone) 및 의존성 패키지 설치
* MAVROS 설치
* 예제 코드

### PX4-Autopilot 패키지
* 참고 문서: Ubuntu Development Environment 문서의 ROS/Gazebo Classic 섹션[^ubuntu-ros-gazebo]

PX4 Autopilot 범용 패키지를 클론한다. 본 문서 작성일 기준으로 패키지의 최신 커밋 이름은 `d532578` 이다.

```bash
git clone https://github.com/PX4/PX4-Autopilot.git --recursive
```

이후 의존성 패키지 설치를 위해 클론한 패키지 내에 있는 스크립트를 다음과 같이 실행한다.

```bash
# 패키지를 클론한 경로에서,
bash ./PX4-Autopilot/Tools/setup/ubuntu.sh --no-sim-tools --no-nuttx
```
> 스크립트를 실행하였을 때 numpy 버전 미달 등의 오류가 발생한다면 다음과 같이 버전을 업그레이드한다.
```bash
sudo pip install numpy --upgrade
```
{: .prompt-info }

기타 의존성 패키지를 설치한다. (아마 이미 설치되어 있을 것이다)

```bash
sudo apt-get install protobuf-compiler libeigen3-dev libopencv-dev -y
```

### MAVROS 설치
* 참고 문서: ROS with MAVROS Installation Guide[^mavros-guide]

쉽고 빠른 설치를 위해 source installation 대신 binary installation 을 진행한다.
위 문서에서 mavros 관련 패키지 설치 명령어가

```bash
sudo apt-get install ros-kinetic-mavros ros-kinetic-mavros-extras
```

로 안내되어 있는데, ROS Kinetic 버전에만 호환될 뿐더러 모든 MAVROS 관련 패키지를 설치하지 않으므로 다음 명령어를 대신 사용하는 것을 추천한다.

```bash
sudo apt install ros-${ROS_DISTRO}-mavros'*'
```

> 환경변수 `ROS_DISTRO`는 `/opt/ros/[ROS dist 이름]/setup.bash (또는 .zsh)` 파일을 source 했다면 정상적으로 등록되었을 것이다. 예를 들어, 사용 중인 ROS 의 distribution 이 Noetic 이라면 이 환경변수의 값은 `noetic` 이 된다.
{: .prompt-info }

이후에는 문서와 동일하게 `install_geographiclib_datasets.sh` 을 다운로드하고 실행한다.

```bash
wget https://raw.githubusercontent.com/mavlink/mavros/master/mavros/scripts/install_geographiclib_datasets.sh
sudo bash ./install_geographiclib_datasets.sh
```

### 예제 코드
여기서는 PX4 User Guide 에서 제공하는 C++ 예제 문서[^cpp-example]를 바탕으로 시뮬레이션을 직접 진행하는 내용을 다룬다.

시뮬레이션에 사용되는 ROS 노드를 구현한 C++ 코드(`offb_node.cpp`)는 예제 문서에 나타나 있는데, `CMakeLists.txt` 파일은 사용자가 알아서 작성하도록 안내되어 있다.

코드 실행을 위해 워크스페이스 폴더(`px4_ws`)를 생성하고, 다음과 같이 디렉토리를 구성하였다.

```
px4_ws
└── src
    └── px4_offboard
        ├── CMakeLists.txt
        ├── include
        │   └── px4_offboard
        ├── package.xml
        └── src
            └── offb_node.cpp
```

여기서 `CMakeLists.txt` 와 `package.xml` 은 다음과 같이 작성하였다.

<details>
<summary>CMakeLists.txt 작성 예시 펼치기/접기</summary>

{% highlight cmake %}

cmake_minimum_required(VERSION 3.0.0)
project(px4_offboard)

## Use C++14
if(NOT CMAKE_CXX_STANDARD)
    set(CMAKE_CXX_STANDARD 14)
endif()
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE "Release")
endif()

## By adding -Wall and -Werror, the compiler does not ignore warnings anymore,
## enforcing cleaner code.
add_definitions(-Wall -Werror)

## Find catkin macros and libraries
find_package(catkin REQUIRED COMPONENTS
    roscpp
    std_msgs
    sensor_msgs
    geometry_msgs
    joint_state_controller
    robot_state_publisher

    mavros_msgs
)

## Find system libraries
find_package(Eigen3 REQUIRED)
find_package(Boost REQUIRED)

## catkin specific configuration
catkin_package(
    INCLUDE_DIRS include ${EIGEN3_INCLUDE_DIR}
    CATKIN_DEPENDS roscpp std_msgs sensor_msgs geometry_msgs mavros_msgs
)

## Specify additional locations of header files
include_directories(
    include
    ${catkin_INCLUDE_DIRS}
    ${EIGEN3_INCLUDE_DIRS}
    ${Boost_INCLUDE_DIRS}
)

## Libraries, executables
# add_library(${PROJECT_NAME} ~~)
# target_link_libraries(${PROJECT_NAME} ${catkin_LIBRARIES})
# add_dependencies(${PROJECT_NAME} ${catkin_EXPORTED_TARGETS})

add_executable(offb_node src/offb_node.cpp)
target_link_libraries(offb_node ${catkin_LIBRARIES})
add_dependencies(offb_node ${catkin_EXPORTED_TARGETS})

{% endhighlight %}

</details>

<details>
<summary>package.xml 작성 예시 펼치기/접기</summary>

{% highlight xml %}

<?xml version="1.0"?>
<package format="2">
    <name>px4_offboard</name>
    <version>1.0.0</version>
    <description>px4 offboard control</description>
    <maintainer email="physism@gmail.com">Sang Su Lee</maintainer>
    <license>None</license>
    <author email="physism@gmail.com">Sang Su Lee</author>

    <buildtool_depend>catkin</buildtool_depend>

    <build_depend>eigen</build_depend>
    <build_depend>boost</build_depend>

    <build_depend>roscpp</build_depend>
    <build_depend>std_msgs</build_depend>
    <build_depend>sensor_msgs</build_depend>
    <build_depend>geometry_msgs</build_depend>
    <build_depend>joint_state_controller</build_depend>
    <build_depend>robot_state_publisher</build_depend>
    <build_depend>mavros_msgs</build_depend>

    <exec_depend>roscpp</exec_depend>
    <exec_depend>std_msgs</exec_depend>
    <exec_depend>sensor_msgs</exec_depend>
    <exec_depend>geometry_msgs</exec_depend>
    <exec_depend>joint_state_controller</exec_depend>
    <exec_depend>robot_state_publisher</exec_depend>
    <exec_depend>mavros_msgs</exec_depend>
</package>

{% endhighlight %}

</details>

<br>

> 위 예시에서는 추가 개발을 염두하고 헤더파일을 저장할 `include` 폴더를 만들고, `CMakeLists.txt` 에도 인클루드 설정을 해두었다.
{: .prompt-info }

위와 같이 모든 파일을 준비하고 `px4_ws` 경로에서 `catkin_make` 또는 `catkin build` 명령어로 빌드를 진행한다.

> `catkin build` 명령어가 사용 불가능하다면 다음과 같이 패키지를 설치한다.
```bash
sudo apt install python3-catkin-tools
```
{: .prompt-tip }

### 시뮬레이션
* 참고 문서: ROS with Gazebo Classic Simulation[^simulation]

다음 순서에 따라 진행하면 예제 코드가 동작하는 것을 시뮬레이션에서 확인할 수 있다.

* MAVROS launch 파일 실행
* Gazebo 실행
* 예제 코드 (ROS 노드) 실행

먼저 ROS 토픽, 서비스를 사용하기 위해 다음과 같이 launch 파일을 실행한다.

```bash
roslaunch mavros px4.launch fcu_url:="udp://:14540@127.0.0.1:14557"
```

그 후 다음과 같이 명령어를 입력하여 Gazebo 를 실행한다.

```bash
# 클론한 PX4-Autopilot 패키지 경로에서,
DONT_RUN=1 make px4_sitl_default gazebo-classic
# 이후에 시뮬레이션을 다시 진행하려는 경우 여기서부터 시작
source Tools/simulation/gazebo-classic/setup_gazebo.bash $(pwd) $(pwd)/build/px4_sitl_default
export ROS_PACKAGE_PATH=$ROS_PACKAGE_PATH:$(pwd)
export ROS_PACKAGE_PATH=$ROS_PACKAGE_PATH:$(pwd)/Tools/simulation/gazebo-classic/sitl_gazebo-classic
roslaunch px4 posix_sitl.launch
```

> 위에서 make 명령어 실행 시 gstreamer 관련 의존성 문제로 `FAILED` 오류가 뜨며 실패하는 경우 다음과 같이 설치한다.
```bash
sudo apt-get install libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev
```
{: .prompt-tip}

추가로 터미널을 열고 위에서 예제 코드를 컴파일하여 얻은 ROS 노드를 실행한다.

```bash
# px4_ws/devel/setup.bash (또는 .zsh) 를 source 한 후,
rosrun px4_offboard offb_node
```

실행하면 예제 문서에 있는 것처럼 arming, takeoff 진행 후 2m 높이까지 떠오르는 것을 확인할 수 있다.

> 컴파일(make) 이후에 PX4-Autopilot 패키지를 다른 경로로 이동시키는 경우 시뮬레이션 실행에 실패할 수 있다. 이런 경우 다음과 같이 조치한다.
* `~/.ros` 폴더를 삭제한다. (내부에 필요한 파일이 있는 경우 따로 저장)
* 패키지 경로에서 `make clean` 명령어 실행
* Gazebo 시뮬레이션 실행 과정 다시 진행
{: .prompt-warning}

## 기타
C++ 예제 문서[^cpp-example]에 다음과 같은 설명이 있다.

> PX4 has a timeout of 500ms between two Offboard commands. If this timeout is exceeded, the commander will fall back to the last mode the vehicle was in before entering Offboard mode. This is why the publishing rate must be faster than 2 Hz to also account for possible latencies. This is also the same reason why it is recommended to enter Offboard mode from Position mode, this way if the vehicle drops out of Offboard mode it will stop in its tracks and hover.

따라서 setpoint position 과 같은 Offboard command 의 타임아웃은 0.5 초임에 주의해야 한다. 이것이 지켜지지 않으면 arming, takeoff 등이 실행되지 않는다.

MAVROS 에서 사용 가능한 토픽, 서비스는 따로 위키문서[^mavros-wiki]에 정리되어있다.

시뮬레이션에 사용되는 무인기의 종류나 환경을 변경하려면 Gazebo Simulation 문서[^gazebo-sim-detailed]를 참고한다.

## 참고자료
[^offboard-control-guide]: [PX4 User Guide: Offboard Control](https://docs.px4.io/main/en/ros/offboard_control.html)
[^gazebo-classic-sim]: [PX4 User Guide: ROS with Gazebo Classic Simulation](https://docs.px4.io/main/en/simulation/ros_interface.html)
[^ubuntu-ros-gazebo]: [PX4 User Guide: Ubuntu Development Environment](https://docs.px4.io/main/en/dev_setup/dev_env_linux_ubuntu.html#ros-gazebo-classic)
[^mavros-guide]: [PX4 User Guide: ROS with MAVROS Installation Guide](https://docs.px4.io/main/en/ros/mavros_installation.html)
[^cpp-example]: [PX4 User Guide: MAVROS Offboard control example (C++)](https://docs.px4.io/main/en/ros/mavros_offboard_cpp.html)
[^simulation]: [PX4 User Guide: ROS with Gazebo Classic Simulation](https://docs.px4.io/main/en/simulation/ros_interface.html)
[^mavros-wiki]: [MAVROS 위키](http://wiki.ros.org/mavros)
[^gazebo-sim-detailed]: [Dronecode: Gazebo Simulation](https://dev.px4.io/v1.10_noredirect/en/simulation/gazebo.html)