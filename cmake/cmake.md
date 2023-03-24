[toc]

modern cmake: cmake3.4之后引入了很多新特性



## cmake格式

三行

```cmake
# 指定最小cmake版本要求
cmake_minimum_required(VERSION 3.9)
# 设置项目名称
project(answer)
# 可执行文件的target，以及依赖
add_executable(answer main.cpp answer.cpp)
```



生成build文件夹，以及makefile

```bash
cmake . -Bbuild
```

编译

```bash
cmake --build build
```



## library target

如果项目中可以复用的部分，可以将这部分代码拆分为llibrary，编译成`lib`库目标

使用`STATIC`指定为静态库

```cmake
ADD_LIBRARY(libhello STATIC hello.cc)

add_executable(main main.cc)

target_link_libraries(main libhello)
```



## subdirectory

功能独立的模块可以放到子目录

比如hello是一个子目录，项目的结构为：

```cmake
.                 
├── CMakeLists.txt
├── build         
├── hello
│   ├── CMakeLists.txt
│   ├── hello.cc
│   └── include
│       └── hello
│           └── hello.hh
└─── main.cc
```

外层的CMakeLists.txt：

```cmake
add_subdirectory(hello)
```

hello的CMakeLists.txt：

```cmake
add_library(libhello STATIC hello.cc)
# 给libhello库添加include目录，PUBLIC使外部使用者能看到include目录
# 链接libhello库时，这里指定的include目录会被添加到使用libhello target的include路径中
target_include_directories(libhello PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include)
```

在include `hello.hh`时，使用`#include "hello/hello.hh"`



## find_package

用来在系统中寻找已经安装的第三方库的头文件和库文件

比如如果要在libhello中使用curl：

```cmake
find_package(CURL REQUIRED)
...
# 为libhello链接curl，PRIVATE代表只有libhello可以看到libcurl，main.cc是不能include curl头文件的
target_link_libraries(libhello PRIVATE CURL::libcurl)
```



## cache string

私密的 App ID、API Key 等不应该直接放在代码里，应该做成可配置的项，从外部传入。在 CMake 中，通过 cache 变量实现：

```cmake
set(WOLFRAM_APPID 
	"" 
	CACHE STRING "WolframAlpha APPID")
if(WOLFRAM_APPID STREQUAL "")
	message(SEND_ERROR "WOLFRAM_APPID must not be empty")
endif()
...
# 将值添加到编译cc文件时的definitions列表，从而在代码中使用
target_compile_definitions(libanswer PRIVATE WOLFRAM_APPID=${WOLFRAM_APPID})
```

传入变量值：

```bash
cmake . -Bbuild -DWOLFRAM_APPID=666wf
```



## 指定C++版本

根目录的CMakeLists.txt：指定整个项目的版本是C++11

```cmake
set(CMAKE_CXX_STANDARD 11)
```

针对某个target，设置特定的版本，或者更细粒度的C++特性

```cmake
target_compile_features(libhello INTERFACE cxx_std_20)
```



## 单元测试

要使用 CTest 运行 CMake 项目的测试程序，需要在 `CMakeLists.txt` 添加一些内容：

```cmake
# CMakeLists.txt
cmake_minimum_required(VERSION 3.14) # 提高了 CMake 版本要求
project(answer)

if(CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME)
    include(CTest)
endif()
```

```cmake
# answer/CMakeLists.txt
if(BUILD_TESTING)
    add_subdirectory(tests)
endif()
```

```cmake
# answer/tests/CMakeLists.txt
add_executable(test_some_func test_some_func.cpp)
add_test(NAME answer.test_some_func COMMAND test_some_func)
```



运行libanswer模块的单元测试

```bash
ctest --test-dir build -R "^answer."
```



## 使用makefile封装

调用 CMake 命令往往需要传很多参数，并且 CMake 生成、CMake 构建、CTest 的命令都不太相同，要获得比较统一的使用体验，可以在外面包一层 Make：

```makefile
WOLFRAM_APPID :=

.PHONY: build configure run test clean

build: configure
	cmake --build build

configure:
	cmake -B build -DWOLFRAM_APPID=${WOLFRAM_APPID}

run:
	./build/answer_app

test:
	ctest --test-dir build -R "^answer."

clean:
	rm -rf build
```
