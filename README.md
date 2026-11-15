# Roboduck

基于 MicroDuck 生态的 HD1910M 双足机器鸭研究项目，围绕可复现硬件、强化学习运动控制、MuJoCo 仿真与 sim-to-real 验证展开。

> **当前状态：**项目处于硬件装配、电气接入和上游技术调研阶段。主仓库用于集成参考资料与管理后续自研成果；参考运行、训练和硬件实现分别位于对应子模块中，规划中的功能不代表已经完成实机验证。

## 硬件方案

Roboduck 采用与 MicroDuck HD1910M 版本一致的主要配置：

| 子系统     | 配置                                             |
| ---------- | ------------------------------------------------ |
| 主控       | Radxa ZERO 3W（RK3566）                          |
| 执行器     | Feetech HD1910M / HD-1910-C001 × 15              |
| 姿态传感器 | BNO08x，独立 I²C4 接口                           |
| 舵机通信   | URT-2 / CH343 USB-UART，FT-SCS 总线              |
| 电源       | 2S 电池为舵机供电，经 DC-DC 降压后为主控提供 5 V |
| 运动形式   | 10 个腿部关节、4 个颈/头部关节、1 个独立嘴部关节 |

## 项目目标

- 建立可复现的 BOM、接线、装配、标定和实机验收流程。
- 复现并验证 50 Hz 策略控制、安全停止、行走与技能切换基线。
- 基于实测数据校准执行器与机器人模型，开展 PPO、域随机化和 ONNX 部署研究。
- 探索实测驱动的数字孪生、电压/温度/负载感知控制、自动标定和安全技能恢复。
- 在基础运动可靠后，再扩展视觉、语音和自主交互。

## 仓库结构

| 路径                                 | 内容                                                                                |
| ------------------------------------ | ----------------------------------------------------------------------------------- |
| `vendor/microduck/microduck`         | Pollen Robotics MicroDuck 主线：Rust 运行时、控制安全、感知交互、仿真接入与发布体系 |
| `vendor/microduck/microduck-hd1910m` | HD1910M 变体：Radxa、FT-SCS、BNO08x、标定、参考策略、部署与仿真集成                 |
| `vendor/microduck/microduck_rl`      | Pollen MicroDuck 强化学习环境、PPO 训练、执行器模型与策略导出                       |
| `vendor/xgoduck/xgoduck_hardware`    | XGO-Duck 制造资料：BOM、STL/3MF、装配指南、PCBA 与打印文档                          |
| `vendor/xgoduck/xgoduck_rl`          | XGO-Duck 的 Feetech 1910 强化学习环境与任务适配                                     |
| `vendor/referred-projects.md`        | 上游项目、训练仓库和官方网站索引                                                    |
| `plans/`                             | 本地调研与路线图；该目录目前由 `.gitignore` 排除，不随主仓库发布                    |

## 获取代码

使用 SSH 克隆并初始化全部子模块：

```bash
git clone --recursive git@github.com:Cle2ment/roboduck.git
cd roboduck
```

已有克隆缺少子模块时：

```bash
git submodule update --init --recursive
```

本仓库没有统一的一键构建命令。请根据任务进入对应子模块，并先阅读其中的 `AGENTS.md`、`README.md` 和 `CONTRIBUTING.md`。

## 从哪个项目开始

| 任务                                    | 首选入口                                                                  |
| --------------------------------------- | ------------------------------------------------------------------------- |
| Roboduck 接线、舵机 ID、零位/HOME、部署 | [HD1910M README](vendor/microduck/microduck-hd1910m/README.md)            |
| HD1910M 仿真与训练约定                  | [HD1910M 训练说明](vendor/microduck/microduck-hd1910m/docs/training.md)   |
| 控制架构、服务边界与安全设计            | [Pollen 架构说明](vendor/microduck/microduck/docs/design/architecture.md) |
| Pollen 运行时开发与验证                 | [Pollen CONTRIBUTING](vendor/microduck/microduck/CONTRIBUTING.md)         |
| PPO、BAM、域随机化与 ONNX 导出          | [Pollen microduck_rl](vendor/microduck/microduck_rl/README.md)            |
| 采购、打印、装配与 PCBA                 | [XGO-Duck 硬件说明](vendor/xgoduck/xgoduck_hardware/README_CN.md)         |
| Feetech 1910 任务与执行器训练参考       | [XGO-Duck RL](vendor/xgoduck/xgoduck_rl/README.md)                        |

三个硬件变体共享部分机械和策略概念，但运行时、ID、零位、IMU、电气和模型参数并不通用。尤其是 **Pollen 的 XL330、HD1910M 的 Radxa + FT-SCS、XGO-Duck 的 UNO Q + STM32** 方案，不应混用部署命令或标定文件。

## 实机安全

- 以 HD1910M README 的**校正版接线表**为准；仓库保留的原始接线初稿存在端子错误。
- 舵机使用 7.4～8.4 V 电源；主控必须经过降压获得稳定 5 V，禁止将电池电压直接接入 ZERO 3W。
- 首次测试时支撑机器人，先进行只读设备检查，再逐步验证 HOME、停止和低幅度动作。
- 每台机器人独立测量关节零位、方向和 IMU 安装变换；示例标定不得直接用于运动。
- 仿真通过不等于实机通过；发布结论应区分固定基座测试、自由仿真和实机验收。

## 来源与许可

Roboduck 参考并保留以下上游项目的归属：

- Pollen Robotics [microduck](https://github.com/pollen-robotics/microduck) 与 [microduck_rl](https://github.com/pollen-robotics/microduck_rl)
- HD1910M [microduck-hd1910m](https://github.com/zhuoligetu123/microduck-hd1910m)
- Luwu Dynamics [xgoduck_hardware](https://github.com/LuwuDynamics/xgoduck_hardware) 与 [xgoduck_rl](https://github.com/LuwuDynamics/xgoduck_rl)

主仓库当前没有统一 LICENSE。代码、模型、网格、权重、图片和 App 的许可范围并不相同；复用或再分发前，请逐项查看子模块的 `LICENSE`、`THIRD_PARTY.md`、`UPSTREAM.md` 和 `LICENSING.md`。公开下载不代表已获得全部资产的商用或再分发授权。
