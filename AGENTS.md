# AGENTS.md

## 项目边界

Roboduck 是基于 MicroDuck 生态的 HD1910M 双足机器鸭研究项目。主仓库当前负责子模块集成、上游出处和研究文档；没有统一构建入口，也没有独立的根级运行时。

当前硬件基线是 **Radxa ZERO 3W + 15 × Feetech HD1910M + BNO08x + URT-2/FT-SCS**。Pollen 主线使用 XL330，XGO-Duck 使用 UNO Q + STM32；三者的驱动、ID、标定、部署命令和策略参数不得互换。

用户交流与研究文档使用中文；代码、测试和提交信息遵循所在项目的既有风格。事实、推断、仿真结果、实机结果和未来规划必须分开表述。

## 仓库地图

| 路径 | 职责 | 修改前必读 |
| --- | --- | --- |
| `vendor/microduck/microduck` | Pollen Rust 运行时、控制安全、感知、仿真和发布 | 该目录 `AGENTS.md`、`CONTRIBUTING.md` |
| `vendor/microduck/microduck-hd1910m` | Roboduck 最接近的硬件接入、标定、参考策略和部署快照 | 顶层 `README.md`、`docs/training.md`、`THIRD_PARTY.md` |
| `vendor/microduck/microduck_rl` | Pollen mjlab/PPO/BAM 训练与 ONNX 导出 | 该目录 `AGENTS.md`、`README.md` |
| `vendor/xgoduck/xgoduck_hardware` | BOM、结构、打印、装配和 PCBA 资料 | `README_CN.md`、`CONTRIBUTING.md`、`LICENSING.md` |
| `vendor/xgoduck/xgoduck_rl` | XGO-Duck 的 mjlab/PPO/HLS1910 训练适配 | `README.md`；该目录没有官方测试套件 |
| `vendor/referred-projects.md` | 上游出处索引 | 保持链接与归属准确 |
| `plans/` | 本地对比、完整性审查与 roadmap | 被 `.gitignore` 排除，克隆后不一定存在 |

`microduck-hd1910m/microduck/` 和 `microduck-hd1910m/microduck_rl/` 是该快照内部的普通目录，不是主仓库的独立子模块。需要更新主线代码时修改对应独立子模块，不要把这两处内嵌副本当作同步源。

## 工作流程

1. **先确定变体和验证级别。** 明确任务属于 Roboduck 集成、Pollen 主线、HD1910M 分叉还是 XGO-Duck；声明结果仅来自文档、仿真、台架还是实机。
2. **加载更具体的约定。** 修改子模块前阅读其内部 `AGENTS.md`；嵌套规则优先于本文件。HD1910M 没有自己的 `AGENTS.md`，以其顶层 README 和训练说明为准。
3. **保持上游边界。** `vendor/` 下五个独立项目均为 Git 子模块。子模块内先确认分支和工作区，修改后按该项目流程验证、提交；回到主仓库单独更新 gitlink。不要把子模块文件当普通主仓库文件提交。
4. **最小化变更。** 根文档只做项目说明和路由，具体机制与命令由所属子模块的文档维护，避免复制后产生多份失效说明。
5. **核对完成条件。** 所有修改范围均有对应验证；文档链接有效；涉及子模块时主仓库指针与预期 commit 一致；最终检查 `git status` 和差异，不包含无关变更。

## 子模块操作

初始化：

```bash
git submodule update --init --recursive
```

开始修改前：

```bash
git submodule status
git -C <submodule-path> status --short --branch
```

更新子模块指针前，先在子模块内确认目标 commit 已完成验证，再在主仓库暂存对应路径。一次提交只包含同一目的的子模块更新；引用上游结论时记录具体 commit，避免依赖会变化的分支名。

`plans/` 是本地草稿区。不要为了提交其中的文档而擅自使用 `git add -f`；如需发布，先与用户确认是否调整 `.gitignore`。

## 文档约定

- 流程图、架构图和拓扑图统一使用 **Mermaid** 代码块，不使用 ASCII/text graph。
- 新增文档链接使用相对路径；移动文件后检查链接是否仍然有效。
- 上游功能、参考权重和 Roboduck 自有成果分开署名；不得把集成复用描述成原创算法。
- 模型张量维度相同不代表兼容。关节顺序、HOME、动作语义、几何、执行器动态、IMU 变换和标定必须一起核对。
- 代码许可证不覆盖模型、网格、权重、图片或 App。发布或许可相关修改须逐资产核查来源文件。
- 修改 Mermaid 后，在环境可用时用 `mmdc` 对每个图块执行实际渲染检查。

## 实机与网络安全

- 未经用户明确要求，不执行可能使能或驱动舵机的命令。首次动作遵循支撑、只读检查、小幅度、可随时断电的顺序。
- 使用 HD1910M README 的校正版接线表；其中的原始接线初稿有端子错误。
- 保留并备份本机标定。示例 `installation.json` 未验证，不得直接用于运动；`configure.py` 默认只读，`--allow-motion` 是显式运动授权边界。
- 同一舵机总线只允许一个控制写入者；启动前停止旧运行时，避免两个 `robotd` 同时访问硬件。
- HD1910M 默认 App/API 配置没有认证，仅限可信局域网，不暴露到公网。
- 仿真成功不能写成实机验收；如实记录未测试项目、失败案例和硬件版本。

## 验证路由

根仓库没有全量测试命令，按实际变更范围选择：

| 变更范围 | 验证入口 |
| --- | --- |
| Pollen `microduck` | 按其 `CONTRIBUTING.md` 运行 `cargo test --workspace`；工具链检查使用项目指定的 `cargo +1.94.1 check --workspace` |
| Pollen `microduck_rl` | `uv run --with pytest pytest tests/`；训练改动另跑 64 环境、5 iteration 冒烟，长训需要 CUDA |
| HD1910M | 按 `docs/training.md` 执行 Rust 协议回归、参考策略测试和仿真测试；实机验证单独记录 |
| XGO-Duck hardware | 核对 BOM/结构/板卡版本、装配入口和文档链接；硬件结论需要相应台架或实机证据 |
| XGO-Duck `xgoduck_rl` | 该发布无官方测试套件；用 `uv run list-envs` 和小规模训练冒烟验证，不能把冒烟当作策略验收 |
| 根级 Markdown | 检查本地链接、事实状态和渲染；流程图须通过 Mermaid 渲染 |

训练项目以 Python 3.12、uv 和 mjlab/MuJoCo Warp 为主。不要假定 Windows 可直接复现 Linux/CUDA 训练流程；先阅读对应 README 的平台要求。

提交和推送仅在用户要求时执行。提交前展示或核对实际变更范围，保持未跟踪的本地研究文件和无关改动不进入提交。
