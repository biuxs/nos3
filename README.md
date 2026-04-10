# NOS3 Scenario 功能汇总文档

> **文档说明**：本文档汇总了 NASA Operational Simulator for Space Systems (NOS3) 官方文档中所有以 "Scenario" 开头的场景页面，梳理每个场景的功能目标，以及实现这些功能所用到的具体指令、脚本、按钮和操作部件。

---

## 目录

1. [Scenario - Installation（安装部署）](#1-scenario---installation安装部署)
2. [Scenario - Demonstration（基础演示）](#2-scenario---demonstration基础演示)
3. [Scenario - COSMOS（地面软件操作）](#3-scenario---cosmos地面软件操作)
4. [Scenario - Nominal Operations（标称轨道运行）](#4-scenario---nominal-operations标称轨道运行)
5. [Scenario - cFS（核心飞行系统）](#5-scenario---cfs核心飞行系统)
6. [Scenario - ADCS Walkthrough（姿控系统详解）](#6-scenario---adcs-walkthrough姿控系统详解)
7. [Scenario - Commanding ADCS in Science Mode（科学模式姿控指令）](#7-scenario---commanding-adcs-in-science-mode科学模式姿控指令)
8. [Scenario - Simulator Expansion（仿真器扩展）](#8-scenario---simulator-expansion仿真器扩展)
9. [Scenario - Flight Build（飞行目标构建）](#9-scenario---flight-build飞行目标构建)
10. [Scenario - Unit Test Creation（单元测试创建）](#10-scenario---unit-test-creation单元测试创建)
11. [功能总览对照表](#功能总览对照表)

---

## 1. Scenario - Installation（安装部署）

### 功能目标

在不同平台上完成 NOS3 的完整安装，包括虚拟机方式（VirtualBox + Vagrant）和原生 Linux 方式（Docker）。

### 实现方式

| 功能 | 指令 / 操作 |
|------|------------|
| 克隆代码仓库 | `git clone <NOS3仓库地址>` |
| 编辑虚拟机配置（CPU、内存、磁盘） | 编辑 `./Vagrantfile` 文件（推荐：4+ CPU，16GB+ 内存，64GB+ 磁盘） |
| 启动虚拟机 | `vagrant up`（在 NOS3 目录下执行） |
| 安装 Docker（Ubuntu） | `sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin` |
| 添加 Docker GPG 密钥 | `sudo apt-get install ca-certificates curl` + `sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc` |
| 将用户加入 Docker 组 | `sudo usermod -aG docker $USER` |
| 拉取容器镜像并初始化环境 | `make prep` |
| 卸载 NOS3 | `make uninstall` |

---

## 2. Scenario - Demonstration（基础演示）

### 功能目标

提供 NOS3 的入门概览演示，展示飞行软件（FSW）、地面软件（GSW）、仿真器三者之间的交互，验证整体架构的基本运行。

### 实现方式

**启动与构建**

| 功能 | 指令 / 操作 |
|------|------------|
| 构建 NOS3 | `make` |
| 启动 NOS3（含 FSW、GSW、仿真器） | `make launch` |
| 停止 NOS3 | `make stop` |
| 启动 Igniter GUI（配置界面） | `make igniter` |

**COSMOS 地面软件操作**

| 功能 | 操作 |
|------|------|
| 打开 COSMOS 地面站 | 点击 NOS3 Launcher 窗口左上角 **COSMOS** 按钮 |
| 打开 Packet Viewer（遥测查看器） | 通过 COSMOS NOS3 Launcher 第三行第一列按钮打开 |
| 启用遥测下行输出 | Command Sender → Target `CFS` → 发送 `TO_ENABLE_OUTPUT`（参数 `DEST_IP: radio_sim`, `DEST_PORT: 5011`） |
| 通过 RADIO 接口启用遥测 | Command Sender → Target `CFS_RADIO` → 发送 `TO_ENABLE_OUTPUT` |
| 验证 FSW 和 GSW 连通性（NOOP 指令） | Command Sender → Target `CFS` → 发送 `NOOP` |
| 通过 RADIO 接口发送 NOOP | Command Sender → Target `CFS_RADIO` → 发送 `NOOP` |
| 验证 Sample 组件存活 | Command Sender → `SAMPLE SAMPLE_NOOP_CC`；观察 Packet Viewer 中 `CMD_COUNT` 增加 |
| 查看 Sample 遥测 | Packet Viewer → Target `SAMPLE` → Packet `SAMPLE_HK_TLM` |
| 查看 ADCS 姿控模式 | Packet Viewer → Target `GENERIC_ADCS` → 遥测点 `MODE` |
| 验证 RADIO 与 DEBUG 遥测一致性 | 同时打开 `CFS` 和 `CFS_RADIO` 两个 Packet Viewer 窗口进行对比 |

**姿控操作（ADCS）**

| 功能 | 操作 |
|------|------|
| 手动控制反作用轮转速 | Command Sender → `GENERIC_REACTION_WHEEL GENERIC_RW_SET_TORQUE_CC`（设置正/负扭矩值） |
| 切换 ADCS 为 Sunsafe 模式 | Command Sender → `GENERIC_ADCS GENERIC_ADCS_SET_MODE_CC`，参数 `GNC_MODE: SUNSAFE_MODE (2)` |
| 查看 42 动力学仿真图像 | 在 42 Cam 窗口中点击拖动来旋转视角 |
| 切换仿真器终端标签 | 点击终端窗口右侧下拉箭头，选择 `sc_1 - Sample Sim` |

---

## 3. Scenario - COSMOS（地面软件操作）

### 功能目标

深入介绍 COSMOS 地面软件各工具模块的用法，包括指令发送、遥测查看、自动化测试等，掌握与实际航天器相似的操作流程。

### 实现方式

**启动**

| 功能 | 指令 / 操作 |
|------|------------|
| 构建并启动（带清理） | `make clean` → `make` → `make launch` |
| 修改启动器布局 | 编辑 `./gsw/cosmos/config/tools/launcher/launcher.txt` |

**COSMOS 核心工具**

| 工具 / 功能 | 操作 |
|------------|------|
| **Command and Telemetry Server**（通信核心） | 点击 COSMOS 顶左 `COSMOS` 按钮自动打开；处理 UDP 接口与协议 |
| **Command Sender**（指令发送器） | 选择 Target → 选择指令包 → 填写参数 → 点击 **Send** 按钮 |
| 区分 RADIO/DEBUG 接口 | Target 名称带 `_RADIO` 后缀代表真实无线接口；不带后缀为直连 debug 接口 |
| **Packet Viewer**（遥测查看器） | 选择 Target 和 Packet → 实时查看遥测值；粉色=无数据或数据超时 |
| 遥测值悬停提示 | 将鼠标移到任意遥测项上，底部显示注释和说明 |
| 遥测值右键菜单 | 右键点击遥测值 → 可查看十六进制详情、数据类型、单位，或对该值绘制时间图 |
| **Test Runner**（自动化测试） | 执行按测试套件/组/用例组织的自动化操作规程，生成详细报告 |
| **Script Runner**（脚本执行） | 使用 Ruby 脚本创建自动化操作规程、调试工具和连续任务（如热控） |

---

## 4. Scenario - Nominal Operations（标称轨道运行）

### 功能目标

演示卫星在轨运行的标准操作流程，包括建立通信连接、发送任务指令（进入科学模式）、下行数据传输（CFDP 文件传输协议），以及识别异常遥测。

### 实现方式

**建立遥测连接**

| 功能 | 操作 |
|------|------|
| 启用 RADIO 下行遥测 | Command Sender → Target `CFS_RADIO` → 发送 `TO_ENABLE_OUTPUT` |
| 确认指令通道正常（NOOP） | Command Sender → Target `CFS_RADIO` → 发送 `NOOP`；观察 CMD_COUNT 增加 |
| 确认 EPS 电源系统正常 | Command Sender → `EPS` Target → 发送 `NOOP`；Packet Viewer 检查 `BATT_VOLTAGE > 24V` |

**科学模式控制**

| 功能 | 操作 |
|------|------|
| 进入科学模式 | Command Sender → Target `MGR_RADIO` → 选择 `MGR_SET_MODE_CC` → 下拉选择 `SCIENCE` → Send |
| 确认模式切换成功 | Packet Viewer → Target `MGR_RADIO` → 观察 `SPACECRAFT_MODE` 切换为 `SCIENCE` |

**文件下行（CFDP 协议）**

| 功能 | 操作 |
|------|------|
| 触发文件下行传输 | Command Sender → Target `CFDP` → 设置 `DSTFILENAME '/tmp/nos3/data/dummy.txt'` → Send |
| 监控文件传输进度 | Packet Viewer → Target `CFDP` → Packet `CFDP_ENGINE_HK` → 观察 `ENG_PDUSRECEIVED`（递增）和 `ENG_INPROGRESSTRANS == 1` |
| 确认文件传输完成 | Packet Viewer 查看 `ENG_DOWN_LASTFILEDOWNLINKED`（显示目标文件名）和 `ENG_TOTALSUCCESSTRANS`（增加 1） |

**仿真器状态查看**

| 功能 | 操作 |
|------|------|
| 查看各仿真器状态 | 终端窗口各标签页（Simulator tabs）查看各组件仿真器输出 |
| 查看 42 动力学仿真 | 42 GUI 窗口查看轨道/姿态图形 |

---

## 5. Scenario - cFS（核心飞行系统）

### 功能目标

深入理解 NOS3 中 cFS（core Flight System）的工作原理，掌握 CI/TO、DS、FM、LC、SC、SCH 等核心应用的配置与使用，实现端到端指令/遥测流程追踪。

### 实现方式

**启动**

| 功能 | 指令 / 操作 |
|------|------------|
| 以最小模式启动（仅 cFS 核心） | `make clean` → `make` → `make launch`（使用 minimal 配置） |

**cFS 核心应用操作**

| 应用 / 功能 | 说明 / 实现方式 |
|------------|----------------|
| **CI（Command Ingest）** | 接收外部指令，将帧解包为 CCSDS 空间数据包，发布到软件总线（SB） |
| **TO（Telemetry Output）** | 收集 SB 上的消息，打包为 CCSDS TM 帧下行 |
| **SCH（Scheduler）** | 默认以 100Hz 运行，周期性触发遥测收集/进程唤醒；通过 SCH 表配置延时和偏移 |
| 修改 SCH 调度表 | 编辑 SCH 表文件（设置每个 slot 的延时和偏移参数） |
| **DS（Data Storage）** | 监听 SB 遥测，按照表定义的规则创建数据文件并存储（支持降采样） |
| **FM（File Manager）** | 提供创建/删除目录和文件、查询文件系统的能力 |
| **LC（Limit Checker）** | 监视遥测点，当超限时触发告警或自动执行相关动作 |
| **SC（Stored Commands）** | 按照 RTS（Relative Time Sequence）表执行预定义的指令序列 |
| 启动脚本加载应用 | 通过 `cfg/nos3_defs/cpuN_cfe_es_startup.scr` 脚本加载各 `.so` 共享库模块 |

**关键配置文件**

| 文件 | 用途 |
|------|------|
| `cfe_es_startup.scr` | 列出所有随 cFS 启动加载的库和应用 |
| `cfg/nos3_defs/targets.cmake` | 定义构建目标和应用列表 |
| SCH 表文件 | 配置调度槽位、频率、延时和偏移 |
| DS 表文件 | 定义遥测数据存储规则和采样策略 |

---

## 6. Scenario - ADCS Walkthrough（姿控系统详解）

### 功能目标

深入了解 NOS3 的姿态确定与控制系统（ADCS）工作原理，包括传感器融合机制、各姿控模式的运作方式，以及各传感器/执行器组件的协同。

### 实现方式

**ADCS 系统工作流**

| 功能 | 实现文件 / 方式 |
|------|----------------|
| 订阅传感器遥测数据 | `generic_adcs_app.c` 中 `Generic_ADCS_AppInit` 方法订阅 MAG/FSS/CSS/IMU/RW/Torquers/ST 组件的 SB 消息 |
| 解析传感器数据（Ingest） | `generic_adcs_ingest.c`（位于 `/nos3/components/generic_adcs/fsw/cfs/src/`） |
| 计算姿控指令（ADAC） | `generic_adcs_adac.c`；调用 `AC_sunsafe` 等模式方法 |
| 发送执行器指令（Output） | `generic_adcs_output.c`；构建 CCSDS 包通过 cFS SB 发出 |
| 42 动力学配置 | 订阅 `Inp_DI.txt`、`Inp_ADAC.txt`、`Inp_DO.txt` |

**执行器操作**

| 功能 | 操作 |
|------|------|
| 控制反作用轮（Reaction Wheels） | Command Sender → `GENERIC_REACTION_WHEEL GENERIC_RW_SET_TORQUE_CC`（正/负扭矩值控制各轴旋转） |
| 磁力矩器（Magnetorquers/Torquers） | 通过 ADCS 自动驱动；用于卸载反作用轮角动量 |
| 切换 ADCS 模式 | Command Sender → `GENERIC_ADCS GENERIC_ADCS_SET_MODE_CC` |

**ADCS 模式说明**

| 模式 | 说明 |
|------|------|
| **Sunsafe 模式** | 对准太阳，确保充电；读取 FSS/CSS 太阳矢量数据，发送扭矩指令使 +x 轴对齐太阳 |
| **Inertial 模式** | 保持惯性空间中固定指向（由指向四元数参数指定） |

---

## 7. Scenario - Commanding ADCS in Science Mode（科学模式姿控指令）

### 功能目标

模拟真实任务操作场景：科学团队要求在科学数据采集期间保持特定指向四元数 `[0, 0, 0, 1]`，需要修改 RTS（Stored Commands）表来实现新的任务参数，并覆盖新的边界情况。

### 实现方式

**核心操作流程**

| 功能 | 实现方式 |
|------|---------|
| 确定需修改的 RTS 表 | 分析 LC（Limit Checker）的 watchpoint 触发逻辑，确认表 27、29、30、31、32、33、34、35 |
| 添加 Star Tracker 和 ADCS 头文件 | 在 RTS 表 `.c` 文件的 `#include` 区段添加对应头文件 |
| 添加 ADCS 惯性模式指令 | RTS 表中插入 `GENERIC_ADCS_SET_MODE_CC`（模式设为 Inertial）和惯性四元数设置指令 |
| 添加指向四元数参数 | 在 RTS 表对应段添加四元数指令定义：`[0, 0, 0, 1]` |
| 离开科学模式时恢复 Sunsafe | RTS 表中添加 `GENERIC_ADCS_SET_MODE_CC`（模式设为 Sunsafe） |
| 关闭 Star Tracker 节省功耗 | 在退出科学模式的 RTS 表中，添加禁用 Star Tracker 及其开关的指令 |
| 关闭 Sample 仪器 | 类比 Sample 组件的已有处理，加入禁用指令 |
| 测试修改效果 | `make launch` → 启动 COSMOS → 在 Command Sender 中发送进入各区域/科学模式的指令 |

---

## 8. Scenario - Simulator Expansion（仿真器扩展）

### 功能目标

演示如何将额外的 42 动力学数据（如磁场矢量 bvb）集成到 Sample 仿真器和飞行软件中，展示从 42 → 仿真器 → 飞行软件 → COSMOS 遥测的完整数据流。

### 实现方式

**开启 42 数据输出**

| 功能 | 操作 |
|------|------|
| 在 42 窗口打开数据输出 | 在 42 终端窗口手动启用 stdout 输出 |

**仿真器端扩展**

| 功能 | 操作 / 文件 |
|------|------------|
| 切换 42 数据提供者 | 编辑 `cfg/InOut/Inp_IPC.txt`：注释 `SAMPLE_PROVIDER`，取消注释 `SAMPLE_42_PROVIDER` |
| 添加 42 IPC 配置段 | 在 `Inp_IPC.txt` 文件末尾添加 11 行配置（设置 `filename: SAMPLE.42`，`host port: 4242`，`echo to stdout: TRUE`），并将第 2 行数值加 1 |
| 添加 bvb 磁场数据点 | 编辑 `components/sample/sim/src/sample_data_point.cpp`，添加 `_sample_bvb` 数组和 getter 方法 |
| 更新头文件声明 | 编辑 `components/sample/sim/inc/sample_data_point.hpp` |
| 将 bvb 数据写入输出帧 | 编辑 `components/sample/sim/src/sample_hardware_model.cpp`，在 `create_sample_data()` 中添加 bvb 字段，并将 `out_data.resize` 从 14 改为 20 |

**飞行软件端扩展**

| 功能 | 文件 |
|------|------|
| 解析新增遥测字段 | 编辑 `components/sample/fsw/shared/sample_device.c` |

**地面软件端扩展**

| 功能 | 操作 / 文件 |
|------|------------|
| 添加 bvb 遥测定义 | 编辑 `components/sample/gsw/SAMPLE/cmd_tlm/SAMPLE_TLM.txt`，在 `SAMPLE_DATA_TLM` 包中添加 bvb 遥测点 |

**重新构建与验证**

| 功能 | 指令 |
|------|------|
| 清理并重新构建 | `make clean` → `make` |
| 启动并验证 | `make launch` |
| 停止 NOS3 | `make stop` |

---

## 9. Scenario - Flight Build（飞行目标构建）

### 功能目标

演示如何在 NOS3 开发环境中同时为仿真环境（x86/amd64）和实际飞行硬件（如树莓派 ARM64）构建飞行软件，实现开发与飞行目标并行维护。

### 实现方式

**添加飞行目标工具链**

| 功能 | 操作 / 文件 |
|------|------------|
| 编辑 Dockerfile 添加交叉编译器 | 编辑 `./deployment/Dockerfile`，在适当 stage 添加：`RUN apt-get install -y gcc-aarch64-linux-gnu g++-aarch64-linux-gnu` |
| 本地构建 Docker 镜像 | 按 Dockerfile 顶部注释中的步骤执行构建命令 |

**配置 CMake 工具链**

| 功能 | 操作 / 文件 |
|------|------------|
| 复制工具链文件 | 复制 `cfg/nos3_defs/toolchain-amd64-posix.cmake` 并命名为 `toolchain-arm64-posix.cmake` |
| 设置 C 编译器 | 编辑 `CMAKE_C_COMPILER` 为 `/usr/bin/arm-linux-gnueabihf-gcc` |
| 设置 C++ 编译器 | 编辑 `CMAKE_CXX_COMPILER` 为 `/usr/bin/arm-linux-gnueabihf-g++` |

**修改 targets.cmake**

| 功能 | 文件 |
|------|------|
| 配置应用/库列表（注释不适用于飞行目标的模块） | 编辑 `cfg/nos3_defs/targets.cmake`，按需注释 `MISSION_GLOBAL_APPLIST` 中无法在飞行平台上编译的条目 |

**构建产物**

| 构建目标 | 路径 |
|---------|------|
| NOS3 仿真构建（cpu1） | `./fsw/build/exe/cpu1` |
| 飞行目标构建（cpu2） | `./fsw/build/exe/cpu2` |

---

## 10. Scenario - Unit Test Creation（单元测试创建）

### 功能目标

为 NOS3 组件创建单元测试框架，实现对飞行软件应用功能和指令的系统化验证，达到关键代码文件的全覆盖。

### 实现方式

**初始化测试目录**

| 功能 | 操作 |
|------|------|
| 检查是否存在 unit-test 目录 | 进入组件 `fsw/cfs/` 目录，运行 `ls`；确认存在 `CMakeLists.txt coveragetest/ inc/ stubs/` |
| 创建测试框架（如不存在） | 从 `sample` 组件复制 unit-test 目录，或使用 sample 脚本生成 |

**编写测试用例**

| 功能 | 方式 |
|------|------|
| 设置被测指令码 | 在测试函数中设置 `FcnCode`（即 cFS 指令功能码） |
| 设置消息 ID | `UT_SetDataBuffer(UT_KEY(CFE_MSG_GetMsgId), &TestMsgId, sizeof(TestMsgId), false)` |
| 设置指令功能码 | `UT_SetDataBuffer(UT_KEY(CFE_MSG_GetFcnCode), &FcnCode, sizeof(FcnCode), false)` |
| 设置消息长度 | `UT_SetDataBuffer(UT_KEY(CFE_MSG_GetSize), &Size, sizeof(Size), false)` |
| 无参数指令长度 | `Size = sizeof(TestMsg.Noop)` |
| 带参数指令长度 | 在顶部联合体结构中单独指定 Size |
| 断言测试结果 | `UtAssert_True(EventTest.MatchCount == 1, "SAMPLE_CMD_NOOP_INF_EID generated (%u)", ...)` |

**构建与运行测试**

| 功能 | 指令 |
|------|------|
| 构建并运行单元测试 | `make` + 相关测试构建目标（在组件 unit-test 目录下） |
| 生成代码覆盖率报告 | `make code-coverage`（仅在直接克隆到 Linux 环境时有效，**不支持** VirtualBox 共享文件夹） |
| 查看覆盖率报告 | 打开生成的 `coverage_report.html` 文件 |

---

## 功能总览对照表

| 功能类别 | 具体功能 | 实现手段 |
|---------|---------|---------|
| **环境管理** | 安装/初始化 | `make prep` |
| | 启动仿真 | `make launch` |
| | 停止仿真 | `make stop` |
| | 清理构建 | `make clean` |
| | 卸载 NOS3 | `make uninstall` |
| | 打开配置 GUI | `make igniter` |
| **遥测通信** | 启用遥测下行 | COSMOS → Command Sender → `TO_ENABLE_OUTPUT` |
| | 查看遥测数据 | COSMOS → Packet Viewer（选择 Target/Packet） |
| | 验证组件存活 | Command Sender → 发送 `NOOP` 指令 |
| **航天器控制** | 切换科学模式 | `MGR_SET_MODE_CC`（SCIENCE） |
| | 切换 ADCS 模式 | `GENERIC_ADCS_SET_MODE_CC`（SUNSAFE_MODE/Inertial） |
| | 控制反作用轮 | `GENERIC_RW_SET_TORQUE_CC`（正/负扭矩） |
| **数据传输** | 文件下行（CFDP） | Command Sender → CFDP Target → 设置目标路径 → Send |
| **仿真扩展** | 接入 42 动力学数据 | 编辑 `Inp_IPC.txt` + 仿真器/FSW/GSW 源码 |
| | 添加新遥测点 | 编辑 `SAMPLE_TLM.txt` |
| **构建配置** | 配置应用列表 | 编辑 `targets.cmake` 和 `cfe_es_startup.scr` |
| | 交叉编译飞行目标 | 编辑 `Dockerfile` + CMake 工具链文件 |
| | 选择地面软件系统 | 编辑 `cfg/nos3-mission.xml` 中的 `gsw` 参数（cosmos/openc3/yamcs/fprime） |
| **测试验证** | 编写单元测试 | 使用 UT-Assert 框架，设置 `FcnCode`/`TestMsgId`/`Size` |
| | 运行覆盖率分析 | `make code-coverage` → `coverage_report.html` |
| | 自动化操作规程 | COSMOS Test Runner / Script Runner（Ruby 脚本） |

---

