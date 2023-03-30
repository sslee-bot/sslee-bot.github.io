---
title: Google test 실습 예시
# date: 2022-11-05 13:30:00 +/-TTTT
categories: [C++]
tags: [google test, gtest]
---

Google test를 간단히 실습해본다.

## 디렉토리 구성
```
gtest_practice/
├── CMakeLists.txt
├── googletest
│   └── (이하 생략)
└── src
    └── test.cpp
```

* [googletest 패키지 깃허브 링크](https://github.com/google/googletest)

### googletest 클론
```bash
git clone https://github.com/google/googletest.git
```

### CMakeLists.txt
```cmake
cmake_minimum_required(VERSION 3.0)
project(gtest_practice)

set(CMAKE_CXX_STANDARD 14)

set(INSTALL_GTEST OFF)
add_subdirectory(googletest)

add_executable(hellotest src/test.cpp)
target_link_libraries(hellotest gtest)
```

### src/test.cpp
```cpp
#include <gtest/gtest.h>

TEST(TestTest, EmptyTest)
{
}

int main(int argc, char **argv)
{
    ::testing::InitGoogleTest(&argc, argv);
    return RUN_ALL_TESTS();
}
```

## 빌드, 실행하기
```bash
# 상위 경로(gtest_practice)에서,
mkdir build && cd build
cmake .. && make
./hellotest  # CMakeLists.txt 에 명시된 executable 이름
```

## 참고자료
* [Basic Setup for GTest and CMake (Platform Independent)](http://mochan.info/c++/cmake/tutorial/2019/03/23/gtest-cmake.html)