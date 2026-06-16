## 目录
- [核心概念](#核心概念)
- [CMake vs Makefile](#cmake-vs-makefile)
- [多后端适配策略](#多后端适配策略)
- [find_package 详解](#find_package-详解)
- [编写 Find 模块](#编写-find-模块)
- [CMake 基础命令](#cmake-基础命令)
- [完整项目示例](#完整项目示例)
- [常见问题](#常见问题)

---

## 核心概念

### Makefile vs CMake

| 维度 | Makefile | CMake |
|------|----------|-------|
| **角色** | 构建规则文件（菜谱） | 构建系统生成器（工厂） |
| **输入** | 手写规则 | `CMakeLists.txt` |
| **输出** | 直接执行编译 | 生成 Makefile / VS 工程 / Ninja 等 |
| **跨平台** | 不跨（依赖特定工具链） | 原生跨平台 |
| **复杂度** | 小型项目可用 | 中大型项目必备 |

**一句话总结**：Makefile 是直接告诉 make 怎么做，CMake 是告诉 cmake 如何生成 makefile（或别的构建文件）。

---

## 多后端适配策略

### 整体思路
1. CMake 检测各加速卡环境（头文件/库/运行时）
2. 设置对应的编译宏（`USE_CUDA`, `USE_MUXI`, `USE_HYGON` 等）
3. 根据激活的后端，编译对应模块 + 链接对应库

### 目录结构建议
```
project/
├── CMakeLists.txt
├── src/
│   ├── main.cpp                    # 统一入口，用宏控制调用哪个后端
│   ├── backend_cuda.cpp            # 英伟达相关
│   ├── backend_muxi.cpp            # 沐曦相关
│   ├── backend_iluvati.cpp         # 天数智芯相关
│   └── backend_hygon.cpp           # 海光相关
└── cmake/
    ├── FindMUXI.cmake              # 沐曦查找模块
    ├── FindILUVATI.cmake           # 天数智芯查找模块
    └── FindHYGON.cmake             # 海光查找模块
```

### 主 CMakeLists.txt 模板

```cmake
cmake_minimum_required(VERSION 3.15)
project(MultiBackend VERSION 1.0)

set(CMAKE_CXX_STANDARD 17)

# ============================================
# 1. 告诉 CMake 去哪里找自定义 Find 脚本
# ============================================
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

# ============================================
# 2. 检测各加速卡环境
# ============================================

# 英伟达 CUDA（CMake 内置）
find_package(CUDAToolkit QUIET)
if(CUDAToolkit_FOUND)
    message(STATUS "Found CUDA: ${CUDAToolkit_VERSION}")
    set(HAS_CUDA TRUE)
endif()

# 沐曦（自定义 FindMUXI.cmake）
find_package(MUXI QUIET)
if(MUXI_FOUND)
    message(STATUS "Found MUXI: ${MUXI_VERSION}")
    set(HAS_MUXI TRUE)
endif()

# 天数智芯 Iluvatar
find_package(ILUVATI QUIET)
if(ILUVATI_FOUND)
    message(STATUS "Found Iluvatar")
    set(HAS_ILUVATI TRUE)
endif()

# 海光 Hygon（基于 ROCm）
find_package(ROCm QUIET)
if(ROCm_FOUND)
    message(STATUS "Found Hygon/ROCm")
    set(HAS_HYGON TRUE)
endif()

# ============================================
# 3. 让用户选择或自动选择后端
# ============================================

set(BACKEND "AUTO" CACHE STRING "Choose backend: AUTO, CUDA, MUXI, ILUVATI, HYGON")

if(BACKEND STREQUAL "CUDA" AND HAS_CUDA)
    set(USE_CUDA TRUE)
elseif(BACKEND STREQUAL "MUXI" AND HAS_MUXI)
    set(USE_MUXI TRUE)
elseif(BACKEND STREQUAL "ILUVATI" AND HAS_ILUVATI)
    set(USE_ILUVATI TRUE)
elseif(BACKEND STREQUAL "HYGON" AND HAS_HYGON)
    set(USE_HYGON TRUE)
elseif(BACKEND STREQUAL "AUTO")
    if(HAS_CUDA)
        set(USE_CUDA TRUE)
    elseif(HAS_MUXI)
        set(USE_MUXI TRUE)
    elseif(HAS_ILUVATI)
        set(USE_ILUVATI TRUE)
    elseif(HAS_HYGON)
        set(USE_HYGON TRUE)
    else()
        message(FATAL_ERROR "No supported accelerator found!")
    endif()
else()
    message(FATAL_ERROR "Backend ${BACKEND} not available on this system")
endif()

# ============================================
# 4. 添加可执行文件并根据后端配置
# ============================================

add_executable(main src/main.cpp)

if(USE_CUDA)
    target_compile_definitions(main PRIVATE USE_CUDA)
    target_sources(main PRIVATE src/backend_cuda.cpp)
    target_link_libraries(main PRIVATE CUDA::cudart)
    message(STATUS "Building with NVIDIA CUDA backend")
    
elseif(USE_MUXI)
    target_compile_definitions(main PRIVATE USE_MUXI)
    target_sources(main PRIVATE src/backend_muxi.cpp)
    target_link_libraries(main PRIVATE MUXI::runtime)
    message(STATUS "Building with MUXI backend")
    
elseif(USE_ILUVATI)
    target_compile_definitions(main PRIVATE USE_ILUVATI)
    target_sources(main PRIVATE src/backend_iluvati.cpp)
    target_link_libraries(main PRIVATE ${ILUVATI_LIBRARIES})
    message(STATUS "Building with Iluvati backend")
    
elseif(USE_HYGON)
    target_compile_definitions(main PRIVATE USE_HYGON)
    target_sources(main PRIVATE src/backend_hygon.cpp)
    target_link_libraries(main PRIVATE ${ROCm_LIBRARIES})
    message(STATUS "Building with Hygon backend")
endif()
```

### main.cpp 模板

```cpp
#include <iostream>

// 后端统一抽象接口
void init_backend();
void run_compute();
void cleanup_backend();

// 各后端实现（通过宏控制）
#ifdef USE_CUDA
    #include "backend_cuda.h"
    #define CURRENT_BACKEND "CUDA"
#elif defined(USE_MUXI)
    #include "backend_muxi.h"
    #define CURRENT_BACKEND "MUXI"
#elif defined(USE_ILUVATI)
    #include "backend_iluvati.h"
    #define CURRENT_BACKEND "Iluvati"
#elif defined(USE_HYGON)
    #include "backend_hygon.h"
    #define CURRENT_BACKEND "Hygon"
#else
    #error "No backend defined!"
#endif

int main() {
    std::cout << "Initializing " << CURRENT_BACKEND << " backend..." << std::endl;
    init_backend();
    
    std::cout << "Running computation..." << std::endl;
    run_compute();
    
    cleanup_backend();
    return 0;
}
```

### 使用方式

```bash
# 自动选择第一个可用的后端
cmake -B build

# 手动指定后端
cmake -B build -DBACKEND=CUDA
cmake -B build -DBACKEND=MUXI
cmake -B build -DBACKEND=ILUVATI
cmake -B build -DBACKEND=HYGON

# 编译
cmake --build build
```

---

## find_package 详解

### 两种模式对比

| 特性 | Module 模式 | Config 模式 |
|------|------------|-------------|
| **查找脚本位置** | `Find<Package>.cmake`（你写或CMake内置） | `<Package>Config.cmake`（软件包自带） |
| **谁提供** | 使用者（你）或 CMake | 库的开发者 |
| **适用场景** | 库没有提供 Config 文件 | 现代库都会提供 |
| **灵活性** | 你可以控制查找逻辑 | 由库的安装方式决定 |

### 搜索路径优先级

```cmake
# CMake 查找 Find<Package>.cmake 的顺序：
# 1. ${CMAKE_MODULE_PATH} 中的所有目录（优先级最高）
# 2. ${CMAKE_ROOT}/Modules/（CMake 内置模块目录）
```

**重要**：CMake **不会**自动搜索项目目录下的 `cmake/` 文件夹，必须手动设置：

```cmake
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
```

### CUDA 相关模块

| 模块名 | 状态 | 说明 |
|--------|------|------|
| `FindCUDAToolkit.cmake` | ✅ **当前推荐** | CMake 3.17 引入，只查找 CUDA 工具包 |
| `FindCUDA.cmake` | ⚠️ **已废弃** | CMake 3.10 起标记为 deprecated |

```cmake
# ✅ 推荐方式
find_package(CUDAToolkit)
target_link_libraries(myapp CUDA::cudart)

# ❌ 废弃方式
find_package(CUDA)
target_link_libraries(myapp ${CUDA_LIBRARIES})
```

### find_package_handle_standard_args

这个函数负责设置 `XXX_FOUND` 变量：

```cmake
include(FindPackageHandleStandardArgs)

find_package_handle_standard_args(MUXI
    REQUIRED_VARS
        MUXI_INCLUDE_DIR    # 必须找到头文件
        MUXI_LIBRARY        # 必须找到库文件
    VERSION_VAR
        MUXI_VERSION        # 可选：版本信息
)

# 这个函数会：
# 1. 检查所有 REQUIRED_VARS 是否都有值
# 2. 设置 MUXI_FOUND = TRUE/FALSE
# 3. 打印找到/未找到的信息
# 4. 处理 REQUIRED 参数
```

---

## 编写 Find 模块

### FindMUXI.cmake 完整示例

```cmake
# cmake/FindMUXI.cmake
# 查找沐曦加速卡 SDK

# 1. 查找头文件
find_path(MUXI_INCLUDE_DIR
    NAMES 
        muxi/runtime.h
        muxi/core.h
    PATHS
        ${MUXI_ROOT}
        $ENV{MUXI_ROOT}
        /opt/muxi/include
        /usr/local/muxi/include
        /usr/include
    PATH_SUFFIXES
        include
        muxi/include
)

# 2. 查找库文件
find_library(MUXI_LIBRARY
    NAMES
        muxi_runtime
        muxi_core
    PATHS
        ${MUXI_ROOT}
        $ENV{MUXI_ROOT}
        /opt/muxi/lib
        /usr/local/muxi/lib
        /usr/lib
    PATH_SUFFIXES
        lib
        lib64
        muxi/lib
)

# 3. 查找工具（可选）
find_program(MUXI_COMPILER
    NAMES
        muxicc
        muxi-clang
    PATHS
        ${MUXI_ROOT}/bin
        $ENV{MUXI_ROOT}/bin
        /opt/muxi/bin
)

# 4. 提取版本信息（从头文件）
set(MUXI_VERSION "0.0.0")
if(MUXI_INCLUDE_DIR AND EXISTS "${MUXI_INCLUDE_DIR}/muxi/version.h")
    file(STRINGS "${MUXI_INCLUDE_DIR}/muxi/version.h" MUXI_VERSION_HEADER)
    
    foreach(line ${MUXI_VERSION_HEADER})
        if(line MATCHES "#define MUXI_VERSION_MAJOR ([0-9]+)")
            set(MUXI_VERSION_MAJOR ${CMAKE_MATCH_1})
        elseif(line MATCHES "#define MUXI_VERSION_MINOR ([0-9]+)")
            set(MUXI_VERSION_MINOR ${CMAKE_MATCH_1})
        elseif(line MATCHES "#define MUXI_VERSION_PATCH ([0-9]+)")
            set(MUXI_VERSION_PATCH ${CMAKE_MATCH_1})
        elseif(line MATCHES "#define MUXI_VERSION \"([0-9.]+)\"")
            set(MUXI_VERSION ${CMAKE_MATCH_1})
        endif()
    endforeach()
    
    if(MUXI_VERSION_MAJOR)
        set(MUXI_VERSION "${MUXI_VERSION_MAJOR}.${MUXI_VERSION_MINOR}.${MUXI_VERSION_PATCH}")
    endif()
endif()

# 5. 处理找到/未找到状态
include(FindPackageHandleStandardArgs)
find_package_handle_standard_args(MUXI
    REQUIRED_VARS
        MUXI_INCLUDE_DIR
        MUXI_LIBRARY
    VERSION_VAR
        MUXI_VERSION
)

# 6. 创建 IMPORTED 目标
if(MUXI_FOUND AND NOT TARGET MUXI::runtime)
    add_library(MUXI::runtime UNKNOWN IMPORTED)
    set_target_properties(MUXI::runtime PROPERTIES
        IMPORTED_LOCATION ${MUXI_LIBRARY}
        INTERFACE_INCLUDE_DIRECTORIES ${MUXI_INCLUDE_DIR}
    )
endif()

# 7. 标记高级变量（在 GUI 中隐藏）
mark_as_advanced(
    MUXI_INCLUDE_DIR
    MUXI_LIBRARY
    MUXI_COMPILER
)
```

### 输出变量说明

| 变量 | 含义 | 由谁设置 |
|------|------|---------|
| `MUXI_FOUND` | 是否找到 SDK | `find_package_handle_standard_args` |
| `MUXI_INCLUDE_DIRS` | 头文件路径 | 你自己设置 |
| `MUXI_LIBRARIES` | 库文件路径 | 你自己设置 |
| `MUXI_VERSION` | 版本号 | 你自己提取 |
| `MUXI::runtime` | IMPORTED 目标 | 你自己创建 |

---

## CMake 基础命令

### 变量与字符串操作

```cmake
set(MY_VAR "hello")                     # 设置变量
set(MY_LIST a b c)                      # 列表
message(STATUS "Info: ${MY_VAR}")       # 打印信息

string(TOLOWER "HELLO" lower)           # 转小写
string(REPLACE "old" "new" text "old")  # 替换
string(REGEX MATCH "[0-9]+" num "v1.2") # 正则匹配

list(APPEND my_list a b c)              # 列表追加
list(LENGTH my_list len)                # 列表长度
list(GET my_list 0 first)               # 获取元素
```

### 文件操作

```cmake
file(READ file.txt content)              # 读取文件
file(WRITE output.txt "content")         # 写入文件
file(GLOB sources src/*.cpp)             # 通配符匹配
file(MAKE_DIRECTORY ${CMAKE_BINARY_DIR}/out)  # 创建目录
```

### 控制流

```cmake
if(VAR STREQUAL "value")
    # ...
elseif(VAR MATCHES "regex")
    # ...
else()
    # ...
endif()

foreach(item ${MY_LIST})
    message("Item: ${item}")
endforeach()

foreach(i RANGE 10)           # 0 到 10
    message("i = ${i}")
endforeach()
```

### 查找命令

```cmake
find_package(CUDAToolkit REQUIRED)      # 查找包
find_path(MUXI_INCLUDE_DIR names.h)     # 查找目录
find_library(MUXI_LIBRARY libname)      # 查找库
find_program(PYTHON python3)            # 查找程序
find_file(CONFIG_FILE config.json)      # 查找文件
```

### 目标操作

```cmake
add_executable(myapp main.cpp)                     # 可执行文件
add_library(mylib STATIC lib.cpp)                  # 静态库
add_library(mylib SHARED lib.cpp)                  # 动态库

target_link_libraries(myapp PRIVATE mylib)         # 链接库
target_include_directories(myapp PRIVATE include/) # 头文件路径
target_compile_definitions(myapp PRIVATE DEBUG)    # 编译宏
target_compile_options(myapp PRIVATE -Wall -O3)    # 编译选项
```

### 属性设置

```cmake
set_target_properties(myapp PROPERTIES
    OUTPUT_NAME "my_app"
    VERSION 1.2.3
    CXX_STANDARD 17
)
```

### 包含与子目录

```cmake
include(FindPackageHandleStandardArgs)              # 包含 CMake 模块
include(cmake/MyUtils.cmake)                       # 包含自定义脚本
add_subdirectory(src)                               # 添加子目录
add_subdirectory(tests)                             # 添加测试目录
```

### 配置模板

```cmake
configure_file(config.h.in config.h @ONLY)
```

`config.h.in` 模板：
```cpp
#pragma once
#cmakedefine USE_CUDA
#cmakedefine VERSION "@PROJECT_VERSION@"
```

### 执行外部命令

```cmake
execute_process(
    COMMAND git log -1 --format=%h
    OUTPUT_VARIABLE GIT_COMMIT_HASH
    ERROR_QUIET
)

add_custom_command(
    OUTPUT generated.cpp
    COMMAND python generator.py --output generated.cpp
    DEPENDS generator.py
)

add_custom_target(run_tests
    COMMAND ./run_tests.sh
    COMMENT "Running unit tests"
)
```

### 最常用的10个命令速查表

| 命令 | 用途 | 示例 |
|------|------|------|
| `set` | 设置变量 | `set(SOURCES main.cpp)` |
| `message` | 打印信息 | `message(STATUS "Configuring...")` |
| `add_executable` | 创建可执行文件 | `add_executable(myapp main.cpp)` |
| `target_link_libraries` | 链接库 | `target_link_libraries(myapp PRIVATE m)` |
| `find_package` | 查找依赖 | `find_package(OpenCV REQUIRED)` |
| `if` | 条件判断 | `if(WIN32)` |
| `foreach` | 循环 | `foreach(f ${SOURCES})` |
| `file` | 文件操作 | `file(GLOB SOURCES src/*.cpp)` |
| `list` | 列表操作 | `list(APPEND SOURCES extra.cpp)` |
| `configure_file` | 配置模板 | `configure_file(config.h.in config.h)` |

### 调试技巧

```cmake
# 打印变量
message(STATUS "MY_VAR = ${MY_VAR}")

# 打印 CMake 内置路径
message(STATUS "CMAKE_ROOT = ${CMAKE_ROOT}")
message(STATUS "CMAKE_MODULE_PATH = ${CMAKE_MODULE_PATH}")

# 调试 find_package
set(CMAKE_FIND_DEBUG_MODE TRUE)
find_package(MUXI)
set(CMAKE_FIND_DEBUG_MODE FALSE)

# 监控变量变化
variable_watch(MY_VAR)
```

---

## 完整项目示例

### 项目结构
```
hardware_test/
├── CMakeLists.txt
├── cmake/
│   ├── FindMUXI.cmake
│   ├── FindILUVATI.cmake
│   └── FindHYGON.cmake
├── src/
│   ├── main.cpp
│   ├── backend_cuda.cpp
│   ├── backend_muxi.cpp
│   ├── backend_iluvati.cpp
│   └── backend_hygon.cpp
└── include/
    └── backend_common.h
```

### CMakeLists.txt（完整版）

```cmake
cmake_minimum_required(VERSION 3.17)
project(HardwareTest VERSION 1.0.0)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 添加自定义模块路径
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

# ============================================
# 后端检测
# ============================================

# CUDA
find_package(CUDAToolkit QUIET)
if(CUDAToolkit_FOUND)
    set(HAS_CUDA TRUE)
    message(STATUS "✅ Found CUDA ${CUDAToolkit_VERSION}")
endif()

# MUXI
find_package(MUXI QUIET)
if(MUXI_FOUND)
    set(HAS_MUXI TRUE)
    message(STATUS "✅ Found MUXI ${MUXI_VERSION}")
endif()

# ILUVATI
find_package(ILUVATI QUIET)
if(ILUVATI_FOUND)
    set(HAS_ILUVATI TRUE)
    message(STATUS "✅ Found Iluvatar")
endif()

# HYGON
find_package(HYGON QUIET)
if(HYGON_FOUND)
    set(HAS_HYGON TRUE)
    message(STATUS "✅ Found Hygon")
endif()

# ============================================
# 后端选择
# ============================================

set(BACKEND "AUTO" CACHE STRING "Backend to use")
set_property(CACHE BACKEND PROPERTY STRINGS AUTO CUDA MUXI ILUVATI HYGON)

if(BACKEND STREQUAL "CUDA" AND HAS_CUDA)
    set(USE_CUDA TRUE)
elseif(BACKEND STREQUAL "MUXI" AND HAS_MUXI)
    set(USE_MUXI TRUE)
elseif(BACKEND STREQUAL "ILUVATI" AND HAS_ILUVATI)
    set(USE_ILUVATI TRUE)
elseif(BACKEND STREQUAL "HYGON" AND HAS_HYGON)
    set(USE_HYGON TRUE)
elseif(BACKEND STREQUAL "AUTO")
    if(HAS_CUDA)
        set(USE_CUDA TRUE)
    elseif(HAS_MUXI)
        set(USE_MUXI TRUE)
    elseif(HAS_ILUVATI)
        set(USE_ILUVATI TRUE)
    elseif(HAS_HYGON)
        set(USE_HYGON TRUE)
    else()
        message(FATAL_ERROR "No backend found! Please install at least one SDK.")
    endif()
else()
    message(FATAL_ERROR "Backend ${BACKEND} not available")
endif()

# ============================================
# 构建配置
# ============================================

add_executable(hardware_test src/main.cpp)

if(USE_CUDA)
    target_compile_definitions(hardware_test PRIVATE USE_CUDA)
    target_sources(hardware_test PRIVATE src/backend_cuda.cpp)
    target_link_libraries(hardware_test PRIVATE CUDA::cudart)
    
elseif(USE_MUXI)
    target_compile_definitions(hardware_test PRIVATE USE_MUXI)
    target_sources(hardware_test PRIVATE src/backend_muxi.cpp)
    target_link_libraries(hardware_test PRIVATE MUXI::runtime)
    
elseif(USE_ILUVATI)
    target_compile_definitions(hardware_test PRIVATE USE_ILUVATI)
    target_sources(hardware_test PRIVATE src/backend_iluvati.cpp)
    target_link_libraries(hardware_test PRIVATE ${ILUVATI_LIBRARIES})
    target_include_directories(hardware_test PRIVATE ${ILUVATI_INCLUDE_DIRS})
    
elseif(USE_HYGON)
    target_compile_definitions(hardware_test PRIVATE USE_HYGON)
    target_sources(hardware_test PRIVATE src/backend_hygon.cpp)
    target_link_libraries(hardware_test PRIVATE ${HYGON_LIBRARIES})
endif()

# 公共头文件
target_include_directories(hardware_test PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/include)

# ============================================
# 安装规则
# ============================================

install(TARGETS hardware_test DESTINATION bin)

# ============================================
# 打印配置摘要
# ============================================

message(STATUS "")
message(STATUS "=== Configuration Summary ===")
message(STATUS "Backend: ${BACKEND}")
message(STATUS "Build type: ${CMAKE_BUILD_TYPE}")
message(STATUS "Install prefix: ${CMAKE_INSTALL_PREFIX}")
message(STATUS "==============================")
message(STATUS "")
```

### include/backend_common.h

```cpp
#pragma once

// 统一的测试接口
void arithmetic_latency_test_main();
void bit_logic_latency_test_main();
void memory_sequential_test_main();
void memory_random_test_main();
void intracore_transfer_test_main();
void intercore_transfer_test_main();

// GPU 测试接口（各后端实现）
void test_precision_main();
```

### src/main.cpp

```cpp
#include <iostream>
#include <string>

#include "backend_common.h"

#ifdef USE_CUDA
    #define GPU_BACKEND "NVIDIA CUDA"
#elif defined(USE_MUXI)
    #define GPU_BACKEND "MUXI"
#elif defined(USE_ILUVATI)
    #define GPU_BACKEND "Iluvatar"
#elif defined(USE_HYGON)
    #define GPU_BACKEND "Hygon"
#else
    #define GPU_BACKEND "None"
#endif

auto sc = [](const std::string& s) {
    std::cout << "\033[1;32m" << s << "\033[0m" << std::endl;
};

int main() {
    sc("========================> 1. CPU 实时运算能力测试 <============================");
    
    sc("==================> 模块一：实时计算测试用例");
    sc("用例 1.1 基础算术运算实时时延测试");
    arithmetic_latency_test_main();
    sc("用例 1.2 位逻辑运算实时时延测试");
    bit_logic_latency_test_main();
    
    sc("==================> 模块二：内存访问性能测试");
    sc("用例 2.1 内存顺序访问性能测试");
    memory_sequential_test_main();
    sc("用例 2.2 内存随机访问性能测试");
    memory_random_test_main();
    
    sc("==================> 模块三：实时传输测试用例");
    sc("用例 3.1 单核内数据传输测试");
    intracore_transfer_test_main();
    sc("用例 3.2 多核间实时数据传输测试");
    intercore_transfer_test_main();
    
    sc("========================> 2. GPU 计算精度及算力测试 <============================");
    
#if defined(USE_CUDA) || defined(USE_MUXI) || defined(USE_ILUVATI) || defined(USE_HYGON)
    sc("GPU Backend: " GPU_BACKEND);
    test_precision_main();
#else
    sc("未指定 GPU 后端，跳过 GPU 测试");
#endif
    
    return 0;
}
```

---

## 常见问题

### Q1: CMake 安装路径在哪里？

```bash
# Ubuntu/Debian 通过 apt 安装
/usr/share/cmake-<版本号>/Modules/

# 查看当前使用的路径
cmake --system-information | grep CMAKE_ROOT
```

### Q2: 如何让 CMake 找到我的 FindMUXI.cmake？

```cmake
# 必须在 find_package 之前设置
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
```

### Q3: `#ifdef A or B or C` 为什么报错？

预处理器的 `#ifdef` 不支持 `or`，必须使用：

```cpp
#if defined(USE_CUDA) || defined(USE_MUXI) || defined(USE_HYGON)
    // 正确的写法
#endif
```

### Q4: 宏是如何从 CMake 传递到 C++ 代码的？

```cmake
# CMakeLists.txt 中
target_compile_definitions(myapp PRIVATE USE_CUDA VERSION=1)
# 等价于编译时添加：-DUSE_CUDA -DVERSION=1
```

```cpp
// C++ 代码中接收
#ifdef USE_CUDA
    // 这个宏被定义了
#endif
```

### Q5: MUXI_VERSION 在哪里定义？

**你需要自己定义**。通常从头文件中提取：

```cmake
if(MUXI_INCLUDE_DIR)
    file(STRINGS "${MUXI_INCLUDE_DIR}/version.h" version_line
         REGEX "#define MUXI_VERSION")
    # 解析版本号...
    set(MUXI_VERSION "1.2.3")
endif()
```

### Q6: 如果没有 GPU 后端，如何让代码编译通过？

添加 CPU fallback：

```cmake
if(NOT HAS_CUDA AND NOT HAS_MUXI AND NOT HAS_ILUVATI AND NOT HAS_HYGON)
    set(HAS_CPU TRUE)
    set(USE_CPU TRUE)
endif()
```

```cpp
#ifdef USE_CPU
    #include "backend_cpu.h"
    void run_compute() { /* CPU 实现 */ }
#endif
```

### Q7: 如何调试 find_package 找不到包的问题？

```cmake
# 方法1：临时关闭 QUIET
find_package(MUXI)  # 会输出详细信息

# 方法2：开启调试模式
set(CMAKE_FIND_DEBUG_MODE TRUE)
find_package(MUXI)
set(CMAKE_FIND_DEBUG_MODE FALSE)

# 方法3：打印搜索路径
message(STATUS "CMAKE_MODULE_PATH = ${CMAKE_MODULE_PATH}")
```

### Q8: 如何同时支持多个后端编译？

有两种策略：

1. **编译时选择**（当前方案）：一个二进制只支持一个后端
2. **运行时选择**：编译所有后端，通过动态加载或函数指针选择

运行时选择的 CMake 配置：
```cmake
# 编译所有后端的源文件
set(ALL_SOURCES src/main.cpp)
if(HAS_CUDA)
    list(APPEND ALL_SOURCES src/backend_cuda.cpp)
    target_compile_definitions(main PRIVATE HAS_CUDA)
endif()
# ... 类似处理其他后端
```

---

## 参考资料

- [CMake 官方文档](https://cmake.org/documentation/)
- [CMake 命令列表](https://cmake.org/cmake/help/latest/manual/cmake-commands.7.html)
- [find_package 详解](https://cmake.org/cmake/help/latest/command/find_package.html)
- [编写 Find 模块指南](https://cmake.org/cmake/help/latest/manual/cmake-developer.7.html#find-modules)

---

*文档生成时间：2026-06-12*