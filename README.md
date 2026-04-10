# NOS3 全部 20 个场景功能汇总文档

> **NOS3（NASA Operational Simulator for Space Systems）**  
> NASA 航天系统运行模拟器，提供飞行软件（FSW）、地面软件（GSW）与硬件仿真器的全链路端到端仿真环境。  
> 文档来源：https://nos3.readthedocs.io/en/latest/  
> 场景顺序依据官方文档目录（共 20 个 Scenario）

---

## 目录

| # | 场景名称 | 核心主题 |
|---|---------|---------|
| 1 | [Scenario - Installation](#1-scenario---installation) | 安装部署 |
| 2 | [Scenario - Demonstration](#2-scenario---demonstration) | 系统总览演示 |
| 3 | [Scenario - cFS](#3-scenario---cfs) | 核心飞行系统 |
| 4 | [Scenario - COSMOS](#4-scenario---cosmos) | 地面软件操作 |
| 5 | [Scenario - Commissioning](#5-scenario---commissioning) | 航天器在轨投运 |
| 6 | [Scenario - Nominal Operations](#6-scenario---nominal-operations) | 标称运行/过顶操作 |
| 7 | [Scenario - ADCS Walkthrough](#7-scenario---adcs-walkthrough) | 姿控系统详解 |
| 8 | [Scenario - Device Fault in Science Mode](#8-scenario---device-fault-in-science-mode) | 科学模式设备故障 |
| 9 | [Scenario - Patching an App or Table](#9-scenario---patching-an-app-or-table) | 在轨打补丁/更新表格 |
| 10 | [Scenario - Commanding ADCS in Science Mode](#10-scenario---commanding-adcs-in-science-mode) | 科学模式中控制 ADCS |
| 11 | [Scenario - Simulator Expansion](#11-scenario---simulator-expansion) | 仿真器扩展 |
| 12 | [Scenario - Command Encryption](#12-scenario---command-encryption) | 指令加密 |
| 13 | [Scenario - Low Power](#13-scenario---low-power) | 低电量处置 |
| 14 | [Scenario - Rapid Tumbling](#14-scenario---rapid-tumbling) | 快速翻滚异常 |
| 15 | [Scenario - Sample Debug with GDB](#15-scenario---sample-debug-with-gdb) | GDB 调试 |
| 16 | [Scenario - Unit Test Creation](#16-scenario---unit-test-creation) | 单元测试创建 |
| 17 | [Scenario - Flight Build](#17-scenario---flight-build) | 飞行软件交叉编译构建 |
| 18 | [Scenario - Ground Operations with Random Errors](#18-scenario---ground-operations-with-random-errors) | 含随机错误的地面操作 |
| 19 | [Scenario - Constellation with Lunar Focus](#19-scenario---constellation-with-lunar-focus) | 月球星座场景 |
| 20 | [Scenario - GPS Spoofing](#20-scenario---gps-spoofing) | GPS 欺骗攻击 |

---

## 1. Scenario - Installation

**目标：** 在目标平台上成功安装 NOS3 运行环境

### 功能说明
介绍两种安装路径：Option A（VirtualBox 虚拟机方式，适合 Windows 用户）和 Option B（原生 Linux 方式，性能更好）。

### 操作方式

| 功能 | 指令 / 工具 / 操作 |
|------|-------------------|
| 克隆仓库 | `git clone https://github.com/nasa/nos3.git` |
| 拉取全部子模块 | `git submodule update --init --recursive` |
| 修改虚拟机配置（CPU/内存/磁盘）| 编辑仓库根目录下的 `./Vagrantfile` |
| 创建并启动虚拟机 | `vagrant up`（需安装 Git 2.47+ / Vagrant 2.4.3+ / VirtualBox 7.1.6+） |
| 安装 Docker（Option B / Linux 原生）| 添加 Docker apt 源 → `sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin` |
| 创建 docker 用户组（避免 sudo）| `sudo groupadd docker` → `sudo usermod -aG docker $USER` → `sudo reboot now` |
| 安装 Python 环境 | `sudo apt install python3-pip python3-venv python3-dev` → `python3 -m venv .venv` |
| 初始化 NOS3（下载容器、安装依赖、启动 Igniter GUI）| `make prep` |
| 修复 Windows 换行符问题 | 在 `nos3/` 目录执行：`find . -type f -print0 \| xargs -0 dos2unix` |
| 卸载 NOS3 | `make uninstall` |

---

## 2. Scenario - Demonstration

**目标：** 演示 NOS3 的完整 FSW + GSW + 仿真器端到端交互，作为其他场景的入门基础

### 功能说明
展示从启动仿真到发送指令、接收遥测、与仿真硬件交互、手动操控 ADCS 的完整流程。

### 操作方式

| 功能 | 指令 / 工具 / 操作 |
|------|-------------------|
| 启动 NOS3 | 终端进入仓库根目录，执行 `make launch` |
| 打开 COSMOS 地面软件 | 点击 NOS3 Launcher 窗口左上角 **COSMOS 按钮** → 点击弹出对话框中的 **OK** |
| 确认航天器模式 | COSMOS **Packet Viewer** → Target: `GENERIC_ADCS` → 遥测点: `MODE`（默认进入 SUNSAFE） |
| 启用 RADIO 遥测下行 | **Command Sender** → Target: `CFS` → 指令: `TO_ENABLE_OUTPUT`（默认参数 `DEST_IP=radio_sim, DEST_PORT=5011`） |
| 对比 DEBUG 与 RADIO 接口 | 打开第二个 Packet Viewer → 选 `CFS_RADIO` 包，与 `CFS`（debug）比较 |
| 对 Sample 载荷发送 NOOP | Command Sender → Target: `SAMPLE` → 指令: `SAMPLE_NOOP_CC` |
| 确认 NOOP 执行成功 | Packet Viewer → `SAMPLE` → `SAMPLE_HK_TLM` → 确认 `CMD_COUNT` 递增 |
| 切换到仿真器终端标签 | 终端窗口右侧下拉箭头 → 选择 `sc_1 - Sample Sim` |
| 通过仿真总线桥直接控制仿真器 | Command Sender → Target: `SIM_CMD_BUS_BRIDGE` → 指令: `SAMPLE_SIM_SET_STATUS`，参数: `status=5` |
| 切换 ADCS 为被动模式（手动控制执行器）| Command Sender → `GENERIC_ADCS` → `GENERIC_ADCS_SET_MODE_CC`，`GNC_MODE=PASSIVE` |
| 手动给反应轮施加力矩 | Command Sender → `GENERIC_REACTION_WHEEL` → `GENERIC_RW_SET_TORQUE_CC` |
| 恢复 ADCS 太阳安全模式 | `GENERIC_ADCS_SET_MODE_CC`，`GNC_MODE=SUNSAFE_MODE (2)` |
| 观察姿态变化 | **42 Cam 窗口**（实时旋转与太阳方向可视化） |

---

## 3. Scenario - cFS

**目标：** 深入了解核心飞行系统（cFS）架构与各应用的配置和交互

### 功能说明
以 NOS3 **最小模式（minimal mode）** 运行，逐一演示 cFS 核心应用（CI/TO、SCH、DS、FM、LC、SC）的作用及表格配置方法，并跟踪端到端指令/遥测数据流。

### cFS 核心应用

| 应用缩写 | 全称 | 功能 |
|---------|------|------|
| CI | Command Ingest | 接收上行指令，解包为 CCSDS Space Packets 并发布到软件总线 |
| TO | Telemetry Output | 收集软件总线消息，打包为遥测帧（CCSDS TM 等）下行 |
| SCH | Scheduler | 100Hz 定时触发任务（遥测采集、应用唤醒），可设延迟/偏移 |
| DS | Data Storage | 将软件总线数据包存储到文件 |
| FM | File Manager | 文件/目录的创建、删除、查询 |
| LC | Limit Checker | 遥测限值监控，触发 Actionpoint（AP）执行 RTS |
| SC | Stored Commands | 管理 ATS（绝对时间序列）和 RTS（相对时间序列） |
| TBL | Table Services | 管理应用表格镜像，支持在轨上传更新 |

### 操作方式

| 功能 | 指令 / 工具 / 操作 |
|------|-------------------|
| 切换为最小配置模式 | 编辑 `cfg/nos3-mission.xml`：将 `<sc-1-cfg>` 改为 `spacecraft/sc-minimal-config.xml` |
| 清理旧产物并启动 | `make clean && make launch` |
| 观察 cFS 启动序列 | 终端 `sc_1 - NOS3 Flight Software` 窗口：PSP 初始化 → cFE 启动 → ES 确定启动状态（POR）→ 各模块加载 |
| 加载顺序来源文件 | `cfe_es_startup.scr`（启动脚本） |
| 配置 SCH 调度表 | 编辑 SCH 表格文件，设置 slot 时序、延迟、偏移参数后上传 |
| 配置 LC 限值表 | 编辑 LC 的 WDT（Watch-point Definition Table）和 ADT（Action-point Definition Table） |
| 使用 ATS 绝对时间序列 | 通过 COSMOS 上传 ATS 表格，在设定的 UTC 时间自动发送指令 |
| 使用 RTS 相对时间序列 | 通过 COSMOS 上传 RTS 表格，由 LC Actionpoint 触发执行 |
| 跟踪数据流 | Command Sender 发指令 → CI 接收发布到 SB → SCH 驱动 TO 下行 → Packet Viewer 显示 |
| 动态加载/卸载应用（无需重启）| 通过 COSMOS 发送 ES（Executive Services）相关指令 |

---

## 4. Scenario - COSMOS

**目标：** 深入掌握 COSMOS 地面软件的各工具使用，并创建自动化测试脚本

### 功能说明
详细演示 COSMOS 客户端-服务器架构：Command and Telemetry Server 作为通信核心，各工具作为客户端连接并操控 Targets（被控对象）。

### COSMOS 核心组件

| 组件 | 功能 | 启动方式 |
|------|------|---------|
| Command and Telemetry Server | 通信核心，管理与所有 Targets 的 UDP/TCP 接口 | 点击 **COSMOS 按钮** 自动启动 |
| Command Sender | 搜索并发送指令 | 随 COSMOS 自动打开 |
| Packet Viewer | 实时遥测可视化，支持自动更新和单位换算 | 随 COSMOS 打开；或 Launcher 第三行第一列 |
| Script Runner | 执行 Ruby 自动化脚本 | NOS3 Launcher 中打开 |
| Test Runner | 组织并运行测试套件，生成测试报告 | NOS3 Launcher 中打开 |
| Telemetry Grapher | 遥测值时序图形化展示 | NOS3 Launcher 中打开 |

### 操作方式

| 功能 | 指令 / 工具 / 操作 |
|------|-------------------|
| 启动 NOS3 | `make clean && make launch` |
| 打开 COSMOS | NOS3 Launcher 左上角 **COSMOS 按钮** |
| 自定义 Launcher 布局 | 编辑 `./gsw/cosmos/config/tools/launcher/launcher.txt` |
| 查看遥测值详情（含 Hex、数据类型、单位）| Packet Viewer → **右键**某数值 → **View Details** |
| 绘制遥测历史时序图 | Packet Viewer → **右键**某数值 → **Graph** |
| 鼠标悬停查看注释 | Packet Viewer → 鼠标悬停在数值上 → 底部状态栏显示说明 |
| 判断数据新鲜度 | 粉色 = 无数据或数据过时；正常颜色 = 数据有效 |
| 创建并运行自动化测试 | Script Runner → File→Open → 选 `.rb` 脚本 → **Start** |
| 监控上行指令记录 | 保持 **Command and Telemetry Server** 可见：记录所有上行指令，最快感知链路中断 |
| 区分 RADIO 与 DEBUG 接口 | `_RADIO` 后缀 = 模拟真实无线链路；无后缀 = 模拟直接调试接线 |
| 查看文本日志/内存转储/事件消息 | COSMOS **Log** 工具 |

---

## 5. Scenario - Commissioning

**目标：** 模拟卫星发射入轨后的初始投运（commissioning）流程

### 功能说明
航天器发射后以安全模式（Safe Mode）启动，需由操作员逐项检查每个物理组件，确认其正常工作后，方可过渡到标称运行和科学模式。使用 NOS3 的自动化 commissioning 脚本完成投运验证全流程。

### 操作方式

| 功能 | 指令 / 工具 / 操作 |
|------|-------------------|
| 启动 NOS3（航天器以 Safe Mode 启动）| `make launch` |
| 打开 COSMOS | NOS3 Launcher → **COSMOS 按钮** |
| 启用遥测下行 | Command Sender → `CFS_RADIO` → `TO_ENABLE_OUTPUT` |
| 运行投运自动化脚本 | COSMOS Script Runner → File→Open → 选择 `gsw/cosmos/config/targets/MISSION/procedures/commissioning.rb` → **Start** |
| 逐组件发送 NOOP 确认存活 | 对每个组件（EPS、ADCS、Sample 等）发送 `<APP>_NOOP_CC`，确认 `CMD_COUNT` 递增 |
| 检查 EPS 健康状态 | Packet Viewer → `GENERIC_EPS` → 查看 `BATT_VOLTAGE`（需 > 24V）、各开关状态 |
| 逐个测试硬件组件开关 | Command Sender → `GENERIC_EPS` → EPS 开关指令，逐一开启各组件 |
| 验证 ADCS 传感器/执行器 | 切换 ADCS 各模式（Sunsafe/BDOT），确认 ADCS 遥测正常 |
| 验证 Sample 仪器响应 | Command Sender → `SAMPLE` → `SAMPLE_ENABLE_CC` 后查看 Sample 遥测 |
| 确认投运完成，切换至科学模式就绪 | Command Sender → `MGR_RADIO` → `MGR_SET_MODE_CC`，选择 `SCIENCE`（投运完成后才可执行） |
| 查看动力学状态 | **42 Cam 窗口**（姿态）和 **42 Map 窗口**（轨道位置） |

---

## 6. Scenario - Nominal Operations

**目标：** 演示真实卫星过顶通信（Pass）的完整标称操作流程

### 功能说明
模拟卫星过顶 8–10 分钟的全流程：建立无线链路 → 确认健康状态 → 切换科学模式 → 文件下行（CFDP 协议），体验与真实在轨操作一致的操作环境。

### 操作方式

| 功能 | 指令 / 工具 / 操作 |
|------|-------------------|
| 启动 NOS3 | `make launch` |
| 打开 COSMOS | NOS3 Launcher → **COSMOS 按钮** |
| 运行标称过顶自动化脚本 | COSMOS Script Runner → File→Open → `gsw/cosmos/config/targets/MISSION/procedures/nominal_ops.rb` → **Start** |
| 连接 RADIO 遥测接口 | Command Sender → Target: `CFS_RADIO` → `TO_ENABLE_OUTPUT` |
| 确认系统存活（NOOP）| 对各组件发送 `<APP>_NOOP_CC`，观察 `CMD_COUNT` 递增 |
| 检查 EPS 电池电量 | Packet Viewer → `GENERIC_EPS` → 确认 `BATT_VOLTAGE` > 24V |
| 切换至科学模式 | Command Sender → Target: `MGR_RADIO` → `MGR_SET_MODE_CC`，选 `SCIENCE` |
| 确认科学模式生效 | Packet Viewer → `MGR_RADIO` → `SPACECRAFT_MODE` 变为 `SCIENCE (3)` |
| 配置科学数据收集区域 | Command Sender → MGR_RADIO → `MGR_SET_CONUS_CC` / `MGR_SET_ALASKA_CC` / `MGR_SET_HAWAII_CC` |
| 下行文件（CFDP 协议）| Command Sender → Target: `CFDP_RADIO` → 指令 `CFDP_SEND_FILE`，设置 `DSTFILENAME='/tmp/nos3/data/dummy.txt'` |
| 确认文件传输进行中 | Packet Viewer → Target: `CFDP` → 包: `CFDP_ENGINE_HK` → 查看 `ENG_PDUSRECEIVED` 递增、`ENG_INPROGRESSTRANS=1` |
| 确认文件传输完成 | 同上 → `ENG_DOWN_LASTFILEDOWNLINKED` 显示目标文件名、`ENG_TOTALSUCCESSTRANS` 增加 1 |
| 结束脚本（模拟过顶结束）| Script Runner → 点击 **Go** 按钮（脚本不自动断开） |
| 观察轨道与动力学 | **42 Cam**（姿态）+ **42 Map**（过美国上空的轨迹） |

---

## 7. Scenario - ADCS Walkthrough

**目标：** 深入理解 NOS3 姿态确定与控制系统（ADCS）的传感器融合机制与四种工作模式

### 功能说明
ADCS 是 NOS3 中的传感器融合组件，读取多个传感器的软件总线数据并控制执行器，而非直接驱动某一个硬件。它订阅 MAG、FSS、CSS、IMU、RW、Torquers、StarTracker 等组件的消息后进行姿态解算和控制。

### ADCS 四种工作模式

| 模式 | 说明 |
|------|------|
| Passive（被动）| 关闭 ADCS 自动控制，操作员手动控制执行器 |
| Sunsafe（太阳安全）| 利用 FSS/CSS 感知太阳方向，控制反应轮和磁力矩器对日定向充电 |
| Inertial（惯性）| 利用 StarTracker + IMU 数据，控制反应轮和磁力矩器保持惯性姿态 |
| BDOT（消旋）| 利用 IMU + 磁强计数据，驱动反应轮和磁力矩器将旋转速率降为零 |

### 操作方式

| 功能 | 指令 / 工具 / 操作 |
|------|-------------------|
| 启动仿真 | `make launch` |
| 切换 ADCS 工作模式 | Command Sender → `GENERIC_ADCS` → `GENERIC_ADCS_SET_MODE_CC`，设置 `GNC_MODE` 参数 |
| 进入被动模式 | `GNC_MODE=PASSIVE` |
| 进入太阳安全模式 | `GNC_MODE=SUNSAFE_MODE (2)` |
| 进入惯性模式 | `GNC_MODE=INERTIAL` |
| 进入消旋模式 | `GNC_MODE=BDOT` |
| 手动给反应轮施加力矩 | Command Sender → `GENERIC_REACTION_WHEEL` → `GENERIC_RW_SET_TORQUE_CC` |
| 查看传感器融合源码（数据接收）| `/nos3/components/generic_adcs/fsw/cfs/src/generic_adcs_ingest.c` |
| 查看姿控算法源码 | `/nos3/components/generic_adcs/fsw/cfs/src/generic_adcs_adac.c` |
| 查看执行器指令构建源码 | `/nos3/components/generic_adcs/fsw/cfs/src/generic_adcs_output.c` |
| 查看 ADCS 订阅初始化 | `/nos3/components/generic_adcs/fsw/cfs/src/generic_adcs_app.c`（`Generic_ADCS_AppInit` 函数） |
| 可视化姿态 | **42 Cam 窗口**（实时旋转和太阳方向） |
| 可视化轨道 | **42 Map 窗口**（地球投影位置） |

---

## 8. Scenario - Device Fault in Science Mode

**目标：** 在科学数据收集过程中发现并处置设备故障（Fault Detection & Correction）

### 功能说明
模拟航天器在 Science Mode（科学模式）下 Sample 仪器或其他组件出现故障时，LC（Limit Checker）自动检测故障、触发 Actionpoint，并由 SC（Stored Commands）执行 RTS 序列进行自主故障处置，同时演示地面操作员如何通过遥测识别并手动响应。

### 操作方式

| 功能 | 指令 / 工具 / 操作 |
|------|-------------------|
| 启动 NOS3 并进入科学模式 | `make launch` → COSMOS 打开 → `MGR_SET_MODE_CC` 选 `SCIENCE` |
| 通过仿真总线桥注入设备故障 | Command Sender → `SIM_CMD_BUS_BRIDGE` → `SAMPLE_SIM_SET_STATUS`，设 `status=5`（错误状态） |
| 观察 LC 告警触发 | Command and Telemetry Server Event Log → 查看 LC Actionpoint 被触发的事件消息 |
| 查看 LC 告警颜色 | Packet Viewer 中限值超越的遥测点变红（red limit） |
| 确认 RTS 自动执行了故障响应 | 查看 FSW 终端日志，观察 RTS 序列自动关闭故障设备的指令记录 |
| 手动发送恢复指令 | Command Sender → `SAMPLE` → `SAMPLE_DISABLE_CC`（关闭故障仪器） |
| 清除 LC 告警 | Command Sender → `LC` → `LC_SET_AP_STATE_CC`（重置 Actionpoint 状态） |
| 恢复科学模式 | 确认设备正常后重新启用 Sample → 发送 `SAMPLE_ENABLE_CC` |
| 参考 FDC 配置文档 | `STF_QuickLook`（LC WDT/ADT 表格和 RTS 关系汇总表） |

---

## 9. Scenario - Patching an App or Table

**目标：** 演示如何在不重启飞行软件的情况下，通过地面上传补丁（应用更新或表格更新）

### 功能说明
利用 cFS 的 TBL（Table Services）、ES（Executive Services）及 CFDP 文件传输协议，实现在轨动态更新应用表格（如 LC 表、RTS 表、SCH 表等），甚至热加载新版本的 `.so` 共享库，无需重启整个飞行软件系统。

### 操作方式

| 功能 | 指令 / 工具 / 操作 |
|------|-------------------|
| 启动 NOS3 | `make launch` |
| 修改需要更新的表格文件 | 在地面编辑对应 `.c`/`.tbl` 文件（如 LC WDT、RTS 表格） |
| 重新编译生成更新后的表格二进制 | `make` 或针对该组件执行 cmake/make 构建 |
| 通过 CFDP 上传表格文件到飞行软件 | Command Sender → `CFDP_RADIO` → `CFDP_SEND_FILE`，将新文件上传至 `/cf/` 目录 |
| 确认文件传输完成 | Packet Viewer → `CFDP_ENGINE_HK` → `ENG_TOTALSUCCESSTRANS` 增加 |
| 通过 TBL 应用加载新表格 | Command Sender → `TBL` → `TBL_LOAD_CC`，指定上传的文件路径 |
| 激活（validate + activate）新表格 | Command Sender → `TBL` → `TBL_VALIDATE_CC` → `TBL_ACTIVATE_CC` |
| 确认表格已生效 | Packet Viewer → `TBL_HK_TLM` → 查看 `LAST_UPDATED_TBL` 和版本信息 |
| 动态加载新版本应用（.so 文件）| Command Sender → `ES` → `ES_LOAD_APP_CC`，指定新 `.so` 文件路径 |
| 查看应用加载成功 | 查看 FSW 终端日志中应用初始化成功的消息 |

---

## 10. Scenario - Commanding ADCS in Science Mode

**目标：** 响应科学需求变更，修改 RTS 表格以在科学模式下实现新的指向四元数

### 功能说明
模拟真实任务中科学团队要求将科学采集时的姿态指向更改为四元数 `[0, 0, 0, 1]`（惯性指向）的场景。需修改 RTS 表 30/31/32（进入 Active Science）和 27/29/33/34/35（退出 Active Science），同时开启星跟踪器（EPS Switch 1 = 0xAA）和 StarTracker App。

### 操作方式

| 功能 | 指令 / 工具 / 操作 |
|------|-------------------|
| 确定需修改的 RTS 表格范围 | 参考 `STF_QuickLook` 文档 或 搜索 Sample Switch/App Toggle 相关指令 |
| 确定进入 Active Science 的 RTS | 表格 30、31、32 |
| 确定退出 Active Science 的 RTS | 表格 27、29、33、34、35 |
| 修改 RTS 表格（增加指向四元数指令）| 编辑 RTS 表格文件，新增 `GENERIC_ADCS_SET_INERTIAL_CMD` 设置目标四元数为 `[0,0,0,1]` |
| 开启 EPS 开关（为 StarTracker 供电）| 在 RTS 中加入 `GENERIC_EPS_SET_SWITCH`，Switch 1 = `0xAA`（170） |
| 开启 StarTracker App | 在 RTS 中加入 `GENERIC_STAR_TRACKER_ENABLE_CC` |
| 上传修改后的 RTS 表格 | 通过 CFDP 上传 → TBL 加载并激活 |
| 测试新指向方案 | `make launch` → COSMOS → MGR 进入 Science Mode → 进入 CONUS 区域 |
| 用 Telemetry Grapher 验证结果 | 打开 **EPS_TEST** 预设 → 添加 StarTracker `Enabled` 值和 EPS Switch 1 状态 → 确认进入 Science Active 时两者均切换到高位（使能） |
| 验证退出科学模式时恢复 | 离开 CONUS 或手动退出 Science Mode → 确认 Switch 1 和 StarTracker 恢复低位（禁用） |
| 测试低电量自动退出边界 | 通过 SIM_CMD_BUS_BRIDGE 拉低 EPS 电量至 60% 以下，验证系统自动切换回 Science Passive |

---

## 11. Scenario - Simulator Expansion

**目标：** 扩展 Sample 仿真器，从 42 动力学仿真器获取并传递额外的环境数据

### 功能说明
演示 NOS3 中数据流全链路：**42 → 仿真器（Sim）→ 飞行软件（FSW）→ 遥测下行 → COSMOS 可视化**。通过将 Sample 仿真器的数据提供者切换为 42 的 SAMPLE_42_PROVIDER，使其获取轨道动力学数据并传递给飞行软件，最终在 COSMOS 中查看新增遥测点。

### 操作方式

| 功能 | 指令 / 工具 / 操作 |
|------|-------------------|
| 查看 42 提供的所有数据（开启输出）| 启动 NOS3 后，在 **42 终端窗口** 中开启 42 输出（turn on 42 output）|
| 切换 Sample 数据提供者为 42 | 编辑 Sample 仿真器配置文件：注释掉 `SAMPLE_PROVIDER` 部分，取消注释 `SAMPLE_42_PROVIDER` 部分 |
| 配置 42 IPC 输出（将数据路由到 Sample）| 编辑 `cfg/InOut/Inp_IPC.txt`：在文件末尾增加 11 行配置，`filename=SAMPLE.42`，`host port=4242`，`echo to stdout=TRUE` |
| 重新构建 | `make` |
| 启动并验证新增数据 | `make launch` → COSMOS Packet Viewer → `SAMPLE` → `SAMPLE_HK_TLM` → 查看新增的 42 数据遥测点 |
| 扩展飞行软件（接收新数据字段）| 修改 Sample FSW 源码（`sample_app.c` 等）添加新数据字段的接收和处理逻辑 |
| 扩展 COSMOS 遥测定义 | 编辑 GSW 目录下的 COSMOS 遥测定义文件，添加新遥测点定义 |
| 重新构建并验证完整链路 | `make && make launch` → Packet Viewer 查看新遥测点实时更新 |

---

## 12. Scenario - Command Encryption

**目标：** 演示使用 CryptoLib 对上行指令实施加密/认证保护

### 功能说明
使用 CryptoLib（基于 CCSDS SDLS-EP 协议，gcrypt 实现）在 NOS3 中对地面到飞行软件的上行指令帧进行加密或认证，演示明文模式、认证模式、加密模式之间的切换与验证方法。

### 操作方式

| 功能 | 指令 / 工具 / 操作 |
|------|-------------------|
| 配置 NOS3 启用 CryptoLib | 编辑 NOS3 配置文件（`cfg/nos3-mission.xml`），启用 CryptoLib 组件选项 |
| 以明文模式运行并发送指令 | `make launch` → COSMOS Command Sender 正常发送指令（无加密） |
| 切换至认证模式 | 修改 CryptoLib 配置参数（`crypto_config`），重新构建启动 |
| 切换至加密模式 | 更新密钥配置文件，配置 gcrypt 参数，重新构建启动 |
| 观察指令帧格式变化 | Command and Telemetry Server → 查看 Bytes Tx（帧结构因加密而变化） |
| 验证飞行软件正确解密并执行 | FSW 终端日志中确认 CryptoLib 认证/解密成功，指令 `CMD_COUNT` 正常递增 |
| 参考 CryptoLib 文档 | GitHub 仓库：`nasa/CryptoLib`（包含详细使用、配置、测试说明） |

---

## 13. Scenario - Low Power

**目标：** 检测低电量状态并执行降功耗应急响应，保障卫星安全

### 功能说明
模拟卫星进入地影区（eclipse）后太阳能不足、电池电压下降的情况，学习通过遥测识别低电量告警并按正确流程关闭非必要设备，待充电恢复后逐步恢复正常运行。

### 操作方式

| 功能 | 指令 / 工具 / 操作 |
|------|-------------------|
| 启动 NOS3 并监控 EPS | `make launch` → COSMOS Packet Viewer → `GENERIC_EPS` → 监测 `BATT_VOLTAGE` |
| 判断低电量触发阈值 | 观察 `BATT_VOLTAGE` 下降趋势（正常值 > 24V，60% 以下触发科学模式关闭） |
| 通过 SIM Bridge 模拟低电量 | Command Sender → `SIM_CMD_BUS_BRIDGE` → EPS 仿真器指令，强制降低电量值 |
| 观察 LC Actionpoint 自动触发 | Event Log → 查看低电量 AP 触发并执行对应 RTS（自动关科学模式） |
| 查看遥测颜色告警 | Packet Viewer 中 `BATT_VOLTAGE` 变红（超下限） |
| 手动关闭非必要组件电源 | Command Sender → `GENERIC_EPS` → EPS 开关指令，逐一关闭对应通道 |
| 观察电量充电恢复 | 42 仿真器驱动太阳矢量数据 → `BATT_VOLTAGE` 随日照区恢复上升 |
| 恢复正常运行 | 电量恢复 > 阈值后，逐步重新开启各组件电源 → 切换回标称模式 |
| 使用 Telemetry Grapher 分析电量曲线 | NOS3 Launcher → Telemetry Grapher → 选择 `GENERIC_EPS` 的 `BATT_VOLTAGE` 遥测点 |

---

## 14. Scenario - Rapid Tumbling

**目标：** 模拟并处置航天器快速翻滚异常

### 功能说明
演示航天器因外力或 ADCS 失效而发生快速翻滚时的诊断流程（通过 42 Cam 观测）以及通过 ADCS BDOT 模式自动消旋或手动控制反应轮恢复稳定的过程。

### 操作方式

| 功能 | 指令 / 工具 / 操作 |
|------|-------------------|
| 启动 NOS3 | `make launch` |
| 观察翻滚状态 | **42 Cam 窗口**：直观看到航天器快速旋转（tumbling） |
| 确认翻滚的角速度数据 | Packet Viewer → IMU / 陀螺遥测 → 查看角速度分量 |
| 切换 ADCS 为 BDOT 消旋模式 | Command Sender → `GENERIC_ADCS` → `GENERIC_ADCS_SET_MODE_CC`，`GNC_MODE=BDOT` |
| 手动给反应轮施加反向力矩 | Command Sender → `GENERIC_REACTION_WHEEL` → `GENERIC_RW_SET_TORQUE_CC`（施加负力矩） |
| 确认旋转速率逐渐归零 | Packet Viewer → IMU 角速度遥测值持续下降 |
| 恢复太阳安全模式（充电优先）| `GENERIC_ADCS_SET_MODE_CC`，`GNC_MODE=SUNSAFE_MODE (2)` |
| 确认稳定后验证姿态 | **42 Cam 窗口**：航天器停止翻滚并稳定对日定向 |

---

## 15. Scenario - Sample Debug with GDB

**目标：** 使用 GDB 调试器附加到正在运行的 NOS3 cFS 进程，进行飞行软件级源码调试

### 功能说明
演示如何将 GNU Debugger（GDB）附加到 cFS 的 Sample 应用进程，设置断点、单步执行、查看变量值，结合 COSMOS 发送指令触发断点，实现对飞行软件代码路径的精确追踪。

### 操作方式

| 功能 | 指令 / 工具 / 操作 |
|------|-------------------|
| 以调试符号构建 | 构建时加入调试标志 `-g`，避免 `-O2` 等优化选项 |
| 启动 NOS3 | `make launch` |
| 查找 cFS 进程 PID | `ps aux \| grep core-cpu1`（或对应的 cFS 进程名） |
| 附加 GDB 到进程 | `gdb -p <PID>` 或在 GDB 中 `attach <PID>` |
| 设置函数断点 | GDB: `break SAMPLE_ProcessCommandPacket`（或其他函数名） |
| 设置行号断点 | GDB: `break sample_app.c:150`（文件名:行号） |
| 继续执行直到断点 | GDB: `continue` |
| 从 COSMOS 触发断点 | Command Sender → `SAMPLE` → 发送任意指令 → GDB 命中断点暂停 |
| 单步执行（不进入调用）| GDB: `next` |
| 单步执行（进入调用）| GDB: `step` |
| 查看变量值 | GDB: `print <变量名>` 或 `print *<指针>` |
| 查看调用栈 | GDB: `backtrace`（或 `bt`） |
| 查看内存 | GDB: `x/<N>x <地址>`（以十六进制显示 N 个内存单元） |
| 退出调试并继续 | GDB: `detach` → `quit` |

---

## 16. Scenario - Unit Test Creation

**目标：** 使用 UT-Assert 框架为 NOS3 组件创建单元测试并达到代码全覆盖

### 功能说明
基于 cFS 的 UT-Assert 测试框架，在 `fsw/unit_test/` 目录下为组件编写独立的单元测试文件，验证各个指令处理函数的行为（包括正常路径和错误路径），并通过 `make code-coverage` 生成 HTML 覆盖率报告。

### 操作方式

| 功能 | 指令 / 工具 / 操作 |
|------|-------------------|
| 检查或创建 unit-test 目录 | 若 `fsw/unit_test/` 目录不存在，从 sample 组件复制或用 sample 脚本生成 |
| 编写测试函数 | 在 `.c` 测试文件中引用 `ut_assert.h`，使用 `UT_SetDataBuffer()` 设置 MsgId、FcnCode、Size |
| 设置命令码（FcnCode）| `UT_SetDataBuffer(UT_KEY(CFE_MSG_GetFcnCode), &FcnCode, sizeof(FcnCode), false)` |
| 设置命令长度（Size）| `UT_SetDataBuffer(UT_KEY(CFE_MSG_GetSize), &Size, sizeof(Size), false)`（无参数命令共享同一大小） |
| 设置消息 ID | `UT_SetDataBuffer(UT_KEY(CFE_MSG_GetMsgId), &TestMsgId, sizeof(TestMsgId), false)` |
| 构建单元测试 | `make ut`（或对应的 CMake 目标） |
| 运行单元测试（含覆盖率）| `make code-coverage`（**注意**：只能在 NOS3 直接克隆到 Linux 时使用，不支持 VirtualBox 共享文件夹） |
| 查看覆盖率报告 | 打开生成的 `coverage_report.html` 文件，查看各文件的行/分支覆盖情况 |

---

## 17. Scenario - Flight Build

**目标：** 将飞行处理器工具链集成到 NOS3 Docker 环境，实现双目标交叉编译

### 功能说明
演示如何在 NOS3 的 Docker 多阶段构建体系中添加飞行目标处理器（以 Raspberry Pi 64-bit ARM 为例）的交叉编译工具链，使同一套代码既能在 NOS3（x86 Linux）上运行，也能编译为飞行计算机的目标二进制文件。

### 操作方式

| 功能 | 指令 / 工具 / 操作 |
|------|-------------------|
| 克隆 deployment 仓库 | `git clone https://github.com/nasa-itc/deployment` |
| 编辑 Dockerfile 添加工具链 | 编辑 `./deployment/Dockerfile`，在 `nos2` 阶段后添加：`RUN apt-get install -y gcc-aarch64-linux-gnu g++-aarch64-linux-gnu` |
| 本地构建新 Docker 镜像 | 按 Dockerfile 顶部说明执行 docker build 命令 |
| 复制工具链 CMake 文件 | `cp cfg/nos3_defs/toolchain-amd64-posix.cmake cfg/nos3_defs/toolchain-arm64-posix.cmake` |
| 编辑工具链 CMake 文件 | 修改 `CMAKE_C_COMPILER=/usr/bin/aarch64-linux-gnu-gcc`，`CMAKE_CXX_COMPILER=/usr/bin/aarch64-linux-gnu-g++` |
| 更新 `targets.cmake` | 添加新目标（cpu2），注释掉目标平台缺少库的应用 |
| 构建（同时生成 NOS3 和飞行目标）| `make`（NOS3 产物在 `./fsw/build/exe/cpu1`；飞行目标在 `./fsw/build/exe/cpu2`） |
| 验证构建产物架构 | `file ./fsw/build/exe/cpu2/core-cpu1`（确认显示 ARM aarch64） |

**Dockerfile 多阶段说明：**  
`nos0`（apt/pip 基础包）→ `nos1`（NOS Engine + ITC Common）→ `nos2`（NOS3 依赖）→ `nos3`（新增飞行目标工具链）

---

## 18. Scenario - Ground Operations with Random Errors

**目标：** 在含随机错误的条件下，训练操作员识别异常遥测并正确响应

### 功能说明
模拟真实任务中可能出现的随机通信错误、遥测数据异常等情况，练习通过 COSMOS 工具链识别故障、分析原因并执行恢复操作，提升操作员在非理想条件下的应急处置能力。

### 操作方式

| 功能 | 指令 / 工具 / 操作 |
|------|-------------------|
| 启动含错误注入配置的仿真 | `make launch`（使用含随机错误注入的配置） |
| 识别遥测数据异常 | Packet Viewer → 观察变**红**（超限）或变**粉**（数据失效/过时）的遥测点 |
| 监控所有上行指令记录 | **Command and Telemetry Server** Event Log（完整记录每条上行指令，利于脚本问题排查） |
| 对遥测值进行时序图形分析 | Packet Viewer → 右键目标遥测点 → **Graph** → 观察历史趋势 |
| 对异常遥测值查看原始数据 | Packet Viewer → 右键 → **View Details**（含 Hex、数据类型、单位） |
| 重发指令（指令未响应时）| Command Sender 重新发送对应指令 |
| 运行自动化检查脚本 | Script Runner → 打开对应 `.rb` 脚本 → **Start** |
| 注入特定错误（调试验证）| Command Sender → `SIM_CMD_BUS_BRIDGE` → 仿真器错误注入指令 |
| 恢复正常运行 | 发送相关复位/重置指令，重新建立正常遥测链路 |

---

## 19. Scenario - Constellation with Lunar Focus

**目标：** 演示 NOS3 对多星座编队的仿真支持，以月球轨道任务为背景

### 功能说明
配置并运行多颗卫星组成的星座（Constellation），每颗卫星有独立的轨道参数和配置，使用 YAMCS（默认 GSW）或 COSMOS 进行多星协同操控，支持不同卫星甚至部署在不同虚拟机上的分布式运行。

### 操作方式

| 功能 | 指令 / 工具 / 操作 |
|------|-------------------|
| 配置多颗卫星 | 编辑 `cfg/nos3-mission.xml`：添加多个 `<sc-N-cfg>` 节点，链接各自的航天器配置文件 |
| 设置各卫星轨道参数 | 在各 `sc-mission-config.xml` 中设置 Orbital X、Y、Z 参数（月球轨道高度、倾角等） |
| 通过 Igniter GUI 管理星座 | `make igniter` → Configuration Tab → 独立配置每颗卫星的组件使能情况 |
| 保存多星配置 | Igniter GUI → **Save As** 按钮，保存新的星座 XML 配置文件 |
| 启动整个星座仿真 | `make launch`（自动为每颗卫星创建独立 Docker 网络容器组） |
| 使用 YAMCS 操控多星 | 打开 YAMCS Web UI → 选择对应卫星的遥测/指令通道分别操控 |
| 观察多卫星动力学 | **42 仿真器**：每颗卫星有独立的轨道和姿态数据 |
| 将 FSW 和 GSW 分离至不同 VM | 参考文档中的分离部署说明，FSW 和 GSW 在不同 VM 上独立运行 |
| 月球轨道背景配置 | 修改 42 的轨道动力学配置文件（`Inp_Sim.txt` 等），设置月球引力参数和初始轨道条件 |

---

## 20. Scenario - GPS Spoofing

**目标：** 模拟 GPS 欺骗攻击，测试飞行软件对虚假导航数据的响应与检测机制

### 功能说明
通过 NOS3 的 NOS Engine 中间件或 SIM_CMD_BUS_BRIDGE，向 GPS 仿真器注入虚假位置/时间数据，模拟在轨 GPS 欺骗攻击场景，观察飞行软件对异常导航数据的响应，并验证告警或安全切换机制。

### 操作方式

| 功能 | 指令 / 工具 / 操作 |
|------|-------------------|
| 启动 NOS3 | `make launch` |
| 查看正常 GPS 遥测基准值 | Packet Viewer → GPS 组件遥测包（位置、速度、时间） |
| 通过 SIM Bridge 注入虚假 GPS 数据 | Command Sender → `SIM_CMD_BUS_BRIDGE` → GPS 仿真器欺骗指令（修改位置/时间参数） |
| 直接修改 GPS 仿真器输出 | 通过仿真器后门接口（NOS Terminal / NOS UDP）设置虚假轨道位置和时间数据 |
| 观察 FSW 导航数据变化 | Packet Viewer → GPS 遥测包 → 确认虚假数据被 FSW 接收 |
| 检测 FSW 异常响应 | FSW 终端日志 → 观察导航异常告警（如位置突变告警、位置与姿态不一致告警） |
| 与 42 真实轨道对比 | 42 仿真器提供真实轨道数据 → 对比 GPS 欺骗数据与 42 轨道数据的偏差 |
| 验证安全模式切换机制 | 若 FSW 实现了 GPS 完整性检查，观察是否自动切换为安全模式（Safe Mode） |
| 恢复正常 GPS 数据 | 重置 GPS 仿真器输出，停止欺骗注入 |

---

## 附录：核心功能快速索引

### 🚀 系统启停与构建

| 功能 | 实现方式 |
|------|---------|
| 全套初始化 | `make prep` |
| 一键构建全部 | `make` 或 Igniter GUI → Build Tab → **Build All** |
| 清理并重新构建 | `make clean && make` |
| 启动完整仿真环境 | `make launch` |
| 仅构建飞行软件 | `make fsw` |
| 仅构建地面软件 | `make gsw` |
| 仅构建仿真器 | `make sims` |
| 打开 Igniter GUI | `make igniter` |
| 构建单元测试 | `make ut` |
| 生成代码覆盖率报告 | `make code-coverage` |
| 卸载 NOS3 | `make uninstall` |

### 📡 地面指令发送（COSMOS Command Sender）

| 功能 | Target / 指令 |
|------|--------------|
| 启用 RADIO 遥测下行 | `CFS_RADIO` → `TO_ENABLE_OUTPUT` |
| 发送 NOOP（确认存活）| `<APP>_RADIO` → `<APP>_NOOP_CC` |
| 切换航天器模式 | `MGR_RADIO` → `MGR_SET_MODE_CC`（参数：SAFE/SCIENCE 等） |
| 设置科学区域 | `MGR_RADIO` → `MGR_SET_CONUS_CC` / `MGR_SET_ALASKA_CC` / `MGR_SET_HAWAII_CC` |
| 切换 ADCS 工作模式 | `GENERIC_ADCS` → `GENERIC_ADCS_SET_MODE_CC`（GNC_MODE 参数） |
| 手动控制反应轮 | `GENERIC_REACTION_WHEEL` → `GENERIC_RW_SET_TORQUE_CC` |
| 开关 EPS 电源通道 | `GENERIC_EPS` → EPS 开关指令（Switch 0/1/... = 0xAA 开启） |
| 直接控制仿真器 | `SIM_CMD_BUS_BRIDGE` → 各仿真器专用指令 |
| 上行文件 | `CFDP_RADIO` → `CFDP_SEND_FILE` |

### 📊 遥测监视（COSMOS Packet Viewer）

| 功能 | 操作 |
|------|------|
| 查看实时遥测 | 选择 Target + Packet |
| 查看数据详情（Hex/单位）| 右键数值 → **View Details** |
| 绘制时序图 | 右键数值 → **Graph** |
| 数据状态判断 | 粉色=无数据/过时；红色=超限；正常色=有效 |
| 使用 Telemetry Grapher | NOS3 Launcher → Telemetry Grapher → 选择预设（如 EPS_TEST） |

### 🗂️ 关键配置文件

| 文件 | 用途 |
|------|------|
| `cfg/nos3-mission.xml` | 主配置：卫星数量、任务时间、各星配置链接 |
| `cfg/spacecraft/sc-mission-config.xml` | 航天器配置：启用/禁用组件和 cFS 应用 |
| `cfg/spacecraft/sc-minimal-config.xml` | 最小配置：仅 cFS 核心应用（cFS Scenario 使用） |
| `cfg/nos3_defs/targets.cmake` | cFS 构建目标定义 |
| `cfg/nos3_defs/cpuN_cfe_es_startup.scr` | cFS 启动脚本：应用加载顺序 |
| `cfg/InOut/Inp_IPC.txt` | 42 IPC 路由配置：将 42 数据路由到各仿真器 |
| `deployment/Dockerfile` | Docker 多阶段构建文件：添加飞行工具链 |
| `gsw/cosmos/config/tools/launcher/launcher.txt` | COSMOS Launcher 布局配置 |

---

*文档生成时间：2026年4月  
数据来源：https://nos3.readthedocs.io/en/latest/*
