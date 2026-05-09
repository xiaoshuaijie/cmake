# STM32 CMake 嵌入式开发环境教程

本仓库整理了 STM32 工程使用 CMake 构建时，在 CLion、VS Code、OpenOCD、J-Link、DAPLink、ST-Link 和 Ozone 中进行编译、下载与调试的完整流程。

内容以 Windows 环境为主，适合从 STM32CubeMX 生成 CMake 工程后，希望脱离传统 IDE 或同时使用 CLion / VS Code 做嵌入式开发的同学参考。

项目地址：<https://github.com/xiaoshuaijie/cmake>

## 项目内容

| 文档 | 说明 |
| --- | --- |
| [clion.md](./clion.md) | CLion 中配置 STM32 CMake 工程、工具链、OpenOCD 和下载器 |
| [vscode.md](./vscode.md) | VS Code 中配置 STM32 CMake 工程、插件、`launch.json` 和 Cortex-Debug |
| [ozone.md](./ozone.md) | SEGGER Ozone 的下载、许可证、J-Link / DAPLink 调试和常用视图 |
| [finally.md](./finally.md) | CLion、VS Code、Ozone 三部分合并版教程 |
| [LICENSE](./LICENSE) | 项目许可证 |

## 适用场景

- 使用 STM32CubeMX 生成 CMake 工程。
- 想在 CLion 中完成 STM32 的编译、烧录和调试。
- 想在 VS Code 中使用 STM32CubeIDE for Visual Studio Code、CMake 和 Cortex-Debug。
- 需要用 OpenOCD 配置 DAPLink、ST-Link 或 J-Link 下载调试。
- 想用 Ozone 查看变量、波形、内存、寄存器并进行日常调试。
- 需要修改 `CMakeLists.txt`，添加用户源码、头文件路径、`printf` 浮点输出或 C++ 支持。

## 环境准备

建议先准备以下工具。实际路径可以按自己的电脑调整，但配置时要保持一致。

| 工具 | 用途 |
| --- | --- |
| STM32CubeMX | 生成 STM32 初始化代码和 CMake 工程 |
| STM32CubeCLT | 提供 CMake、Ninja、Arm GCC、GDB 等命令行工具 |
| Arm GNU Toolchain | 编译 `arm-none-eabi` 目标程序 |
| OpenOCD | 通过 DAPLink、ST-Link、J-Link 下载和调试 |
| CLion | CMake 工程编辑、构建和嵌入式调试 |
| VS Code | 轻量编辑、构建、下载和 Cortex-Debug 调试 |
| Cortex-Debug | VS Code 中连接 OpenOCD / J-Link 的调试插件 |
| SEGGER Ozone | 独立图形化调试工具，适合观察变量和波形 |
| SEGGER J-Link 软件包 | J-Link 驱动、GDB Server、Ozone 支持 |
| Everything | 可选，用于快速查找 `openocd.exe`、`arm-none-eabi-gdb` 等路径 |

> 注意：OpenOCD、STM32CubeCLT、Arm GCC 等工具解压或安装后，尽量不要频繁移动目录。路径变化后，CLion、VS Code 和 Ozone 中的配置都需要同步修改。

## 推荐学习顺序

1. 用 STM32CubeMX 生成带 CMake 支持的工程。
2. 选择一种主力 IDE。
   - 使用 CLion：阅读 [clion.md](./clion.md)。
   - 使用 VS Code：阅读 [vscode.md](./vscode.md)。
3. 配置 OpenOCD，并确认 DAPLink / ST-Link / J-Link 至少一种下载器可用。
4. 成功生成 `.elf` 文件并下载到开发板。
5. 需要更强的图形化调试时，阅读 [ozone.md](./ozone.md)。
6. 根据自己的代码结构修改 `CMakeLists.txt`。

## 总体工作流

```text
STM32CubeMX 生成工程
        |
        v
选择 CLion 或 VS Code 打开 CMake 工程
        |
        v
配置 CMake / Ninja / arm-none-eabi-gcc / arm-none-eabi-gdb
        |
        v
选择 Debug 预设并完成编译
        |
        v
配置 OpenOCD、目标芯片和下载器
        |
        v
下载固件并调试
        |
        v
按需使用 Ozone 查看变量、波形和运行状态
```

## CLion 配置概要

CLion 适合已经习惯 JetBrains 工具链、希望在一个 IDE 中完成 CMake 管理、代码跳转、构建和调试的用户。

### 1. 生成并打开工程

1. 在 STM32CubeMX 中生成 CMake 工程。
2. 右键工程目录，选择使用 CLion 打开。
3. 等待 CLion 加载 CMake 配置。
4. 选择 `Debug` 预设。

详细截图见 [clion.md](./clion.md)。

### 2. 配置嵌入式开发环境

在 CLion 中进入：

```text
设置 -> 构建、执行、部署 -> 嵌入式开发
```

需要配置的重点：

- STM32CubeMX / STM32CubeCLT 路径。
- OpenOCD 路径。
- CMake 路径，建议选择 STM32CubeCLT 自带的 `cmake.exe`。
- Ninja 路径，建议选择 STM32CubeCLT 自带的 `ninja.exe`。
- C 编译器：`arm-none-eabi-gcc.exe`。
- C++ 编译器：`arm-none-eabi-c++.exe`。

### 3. 配置 OpenOCD 运行项

CLion 中可通过运行配置选择 OpenOCD。配置时需要关注：

- `openocd.exe` 路径。
- OpenOCD `scripts` 目录。
- 下载器配置文件，例如：
  - `interface/cmsis-dap.cfg`
  - `interface/stlink.cfg`
  - `interface/jlink.cfg`
- 目标芯片配置文件，例如：
  - `target/stm32f1x.cfg`
  - `target/stm32f4x.cfg`
  - `target/stm32f2x.cfg`

不同芯片需要选择对应的 target 文件，不要直接照抄示例中的芯片型号。

## VS Code 配置概要

VS Code 适合轻量化开发，也适合配合 Cortex-Debug 做 OpenOCD / J-Link 调试。

### 1. 推荐插件

在扩展面板中安装：

- STM32CubeIDE for Visual Studio Code
- Cortex-Debug
- CMake

可选参考：

- XRobot 相关资料。
- OpenOCD 安装教程。
- Cortex-Debug 的 Live Watch 使用说明。

### 2. 加载工程并编译

1. 使用 VS Code 打开 STM32CubeMX 生成的工程目录。
2. 等待顶部出现 CMake 配置选择。
3. 选择 `Debug` 预设。
4. 等待右下角提示加载工程。
5. 执行构建，确认可以生成 `.elf` 文件。

### 3. 工具链选择

生成工程通常默认使用 GCC 工具链：

```json
"toolchainFile": "${sourceDir}/cmake/gcc-arm-none-eabi.cmake"
```

如果只使用 VS Code，可以根据项目需要改用 Clang：

```json
"toolchainFile": "${sourceDir}/cmake/starm-clang.cmake"
```

如果 CLion 和 VS Code 同时使用，建议保持 GCC 工具链，减少两边配置差异。

### 4. Cortex-Debug 示例配置

以下是一个 OpenOCD 调试配置示例，需要按自己的工程名、OpenOCD 路径、GDB 路径和芯片型号修改。

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "cwd": "${workspaceRoot}",
      "executable": "build/Debug/your_project.elf",
      "name": "Debug with OpenOCD",
      "request": "launch",
      "type": "cortex-debug",
      "servertype": "openocd",
      "configFiles": [
        "interface/cmsis-dap.cfg",
        "target/stm32f4x.cfg"
      ],
      "serverpath": "E:/tools/openocd/bin/openocd.exe",
      "searchDir": [
        "E:/tools/openocd/openocd/scripts"
      ],
      "runToEntryPoint": "main",
      "showDevDebugOutput": "none",
      "gdbPath": "C:/Users/your_name/AppData/Local/stm32cube/bundles/gnu-tools-for-stm32/bin/arm-none-eabi-gdb.exe",
      "liveWatch": {
        "enabled": true,
        "samplesPerSecond": 4
      }
    }
  ]
}
```

常见需要修改的字段：

- `executable`：编译生成的 `.elf` 文件路径。
- `configFiles`：下载器和目标芯片配置。
- `serverpath`：`openocd.exe` 路径。
- `searchDir`：OpenOCD 的 `scripts` 目录。
- `gdbPath`：`arm-none-eabi-gdb.exe` 路径。

## 下载器配置说明

### DAPLink / CMSIS-DAP

常用配置：

```text
interface/cmsis-dap.cfg
target/stm32f4x.cfg
```

如果使用自定义 `daplink.cfg`，需要根据芯片 Flash 大小、目标芯片系列和 OpenOCD 版本调整。Flash 大小可以在 STM32CubeMX 芯片信息页面查看。

### ST-Link

常用配置：

```text
interface/stlink.cfg
target/stm32f4x.cfg
```

部分新版 OpenOCD 对旧写法 `hla_swd` 支持不一致。如果遇到连接失败，可以尝试使用通用 SWD 配置方式，避免依赖 HLA 写法。

### J-Link

常用配置：

```text
interface/jlink.cfg
target/stm32f4x.cfg
```

J-Link 也可以配合 SEGGER Ozone 使用。部分 J-Link 或 Ozone 功能需要许可证，具体以 SEGGER 工具提示为准。

## Ozone 使用概要

Ozone 适合在固件已经能正常构建后，用图形化方式进行深入调试。

### 1. 下载与许可证

使用前需要安装 SEGGER Ozone。以下情况可能需要申请或添加许可证：

- 商业许可证过期。
- 当前电脑没有可用许可证。
- 使用 DAPLink 或某些受限调试功能。

详细步骤见 [ozone.md](./ozone.md)。

### 2. 创建调试工程

基本流程：

1. 打开 Ozone。
2. 选择调试工具：J-Link 或 DAPLink。
3. 选择目标芯片。
4. 选择工程编译出的 `.elf` 文件。
5. 选择启动方式：
   - `Attach`：连接正在运行的目标程序。
   - `Launch` / `Download & Reset Program`：下载并复位启动程序。

### 3. 日常调试功能

Ozone 常用功能包括：

- 查看全局变量和局部变量。
- 查看寄存器和内存。
- 使用 Watch 窗口观察表达式。
- 查看数据波形。
- 控制断点、单步、继续运行。
- 对比 `Attach` 和 `Download & Reset Program` 的不同启动方式。

## CMakeLists.txt 常用修改

STM32CubeMX 生成的 CMake 工程通常需要手动加入用户源码、头文件路径和链接选项。

### 1. 手动添加源码和头文件目录

这种方式最清晰，适合文件数量不多或希望精确控制构建内容的工程。

```cmake
target_sources(${CMAKE_PROJECT_NAME} PRIVATE
    task/dr16/dr16_task.c
    bsp/log/bsp_log.c
    component/comp_utils.c
    component/comp_cmd.c
    component/comp_ahrs.c
    bsp/usart/bsp_uart.c
    modules/dr16/dev_dr16.c
    SEGGER/RTT/SEGGER_RTT.c
    SEGGER/RTT/SEGGER_RTT_printf.c
    SEGGER/RTT/SEGGER_RTT_ASM_ARMv7M.s
    tool/convert/convert.c
    ADC/drv_adc.cpp
)

target_include_directories(${CMAKE_PROJECT_NAME} PRIVATE
    task/dr16
    SEGGER
    SEGGER/Config
    SEGGER/RTT
    component
    tool
    tool/convert
    bsp
    bsp/usart
    bsp/log
    modules
    modules/dr16
    ADC
)
```

### 2. 自动扫描源码目录

这种方式适合用户代码目录较多的工程。优点是新增 `.c`、`.cpp`、`.s` 文件后不用每次手动添加；缺点是构建内容不如手动列出直观。

```cmake
set(USER_CODE_DIRS
    "${CMAKE_CURRENT_SOURCE_DIR}/task"
    "${CMAKE_CURRENT_SOURCE_DIR}/component"
    "${CMAKE_CURRENT_SOURCE_DIR}/bsp"
    "${CMAKE_CURRENT_SOURCE_DIR}/modules"
    "${CMAKE_CURRENT_SOURCE_DIR}/tool"
    "${CMAKE_CURRENT_SOURCE_DIR}/SEGGER"
    "${CMAKE_CURRENT_SOURCE_DIR}/ADC"
)

set(USER_SOURCES)
set(USER_HEADERS)

foreach(USER_CODE_DIR IN LISTS USER_CODE_DIRS)
    file(GLOB_RECURSE USER_DIR_SOURCES CONFIGURE_DEPENDS
        "${USER_CODE_DIR}/*.c"
        "${USER_CODE_DIR}/*.cpp"
        "${USER_CODE_DIR}/*.s"
        "${USER_CODE_DIR}/*.S"
    )

    file(GLOB_RECURSE USER_DIR_HEADERS CONFIGURE_DEPENDS
        "${USER_CODE_DIR}/*.h"
        "${USER_CODE_DIR}/*.hpp"
    )

    list(APPEND USER_SOURCES ${USER_DIR_SOURCES})
    list(APPEND USER_HEADERS ${USER_DIR_HEADERS})
endforeach()

list(REMOVE_DUPLICATES USER_SOURCES)
list(REMOVE_DUPLICATES USER_HEADERS)

set(USER_INCLUDE_DIRS ${USER_CODE_DIRS})
foreach(USER_HEADER IN LISTS USER_HEADERS)
    get_filename_component(USER_HEADER_DIR "${USER_HEADER}" DIRECTORY)
    list(APPEND USER_INCLUDE_DIRS "${USER_HEADER_DIR}")
endforeach()
list(REMOVE_DUPLICATES USER_INCLUDE_DIRS)

target_sources(${CMAKE_PROJECT_NAME} PRIVATE
    ${USER_SOURCES}
    ${USER_HEADERS}
)

target_include_directories(${CMAKE_PROJECT_NAME} PRIVATE
    ${USER_INCLUDE_DIRS}
)
```

### 3. 添加 `printf` 浮点支持

如果 `printf("%f")` 不能正常输出浮点数，可以添加链接选项：

```cmake
target_link_options(${CMAKE_PROJECT_NAME} PRIVATE
    LINKER:-u,_printf_float
)
```

如果工程中混用了 C++ 文件，并出现错误的 `libob.a` 依赖，也可以参考：

```cmake
list(REMOVE_ITEM CMAKE_C_IMPLICIT_LINK_LIBRARIES ob)
```

### 4. 添加 C++ 支持

如果项目中使用 `.cpp` 文件，需要启用 C++ 语言并设置标准。

```cmake
set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS ON)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
```

同时确认启用了 C、C++ 和 ASM：

```cmake
enable_language(C CXX ASM)
```

## 常见问题排查

| 问题 | 可能原因 | 处理方式 |
| --- | --- | --- |
| CLion 或 VS Code 找不到 CMake | CMake 路径未配置或环境变量未生效 | 指向 STM32CubeCLT 自带 `cmake.exe` |
| Ninja 不可用 | 未安装 Ninja 或路径错误 | 指向 STM32CubeCLT 自带 `ninja.exe` |
| 找不到 `arm-none-eabi-gcc` | 工具链未安装或路径错误 | 检查 STM32CubeCLT / Arm GCC 安装目录 |
| OpenOCD 启动失败 | `serverpath` 或 `searchDir` 配置错误 | 分别确认 `openocd.exe` 和 `scripts` 目录 |
| 下载器连接失败 | interface 或 target 配置不匹配 | 根据实际下载器和芯片系列修改 cfg |
| ST-Link 配置失败 | OpenOCD 版本与旧 HLA 写法不兼容 | 尝试使用新版通用 SWD 配置 |
| VS Code 找不到 `.elf` | `executable` 路径写错或尚未构建 | 先构建工程，再复制 `.elf` 相对路径 |
| Cortex-Debug 找不到 GDB | `gdbPath` 配置错误 | 使用 Everything 搜索 `arm-none-eabi-gdb.exe` |
| `printf` 浮点不输出 | 默认未链接浮点格式化支持 | 添加 `LINKER:-u,_printf_float` |
| C++ 文件编译或链接异常 | 未启用 C++ 或隐式库冲突 | 添加 `enable_language(C CXX ASM)` 并检查链接库 |
| Ozone 提示许可证问题 | 当前功能需要许可证 | 按 Ozone 提示申请或添加许可证 |

## 文档维护建议

- 如果只想阅读完整合并版，可以直接查看 [finally.md](./finally.md)。
- 如果只关心某个工具，优先阅读对应专题文档。
- 如果修改了工具安装路径，记得同步修改 CLion、VS Code、Ozone 中的路径配置。
- 如果换了芯片型号，必须同步修改 OpenOCD 的 `target/*.cfg`。
- 如果换了下载器，必须同步修改 OpenOCD 的 `interface/*.cfg`。

## 许可证

本项目使用 [LICENSE](./LICENSE) 中声明的许可证。

