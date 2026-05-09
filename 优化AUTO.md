### 优化 AUTO

#### 当前代码



#### 存在问题

| 编号 | 模块               | 当前设计                                       | 存在问题              | 风险分析                              | 优化建议                                                      | 优先级 |
| -- | ---------------- | ------------------------------------------ | ----------------- | --------------------------------- | --------------------------------------------------------- | --- |
| 1  | Step 设计        | Step 同时包含语义（如 SlotMapped）与执行含义             | Step 非纯描述，混入执行逻辑  | 语义与执行耦合，扩展困难，复用性差                 | 将 Step 设计为纯数据结构（Declarative），仅描述意图；执行逻辑完全下沉至 Executor     | 高   |
| 2  | RuntimeContext   | StepResult 写入 RuntimeContext，后续 Step 隐式读取  | 数据流不透明，读写关系不清晰    | 调试困难，易出现隐式依赖与状态错误                 | 引入明确的数据流机制（类似 Blackboard），定义 Step 输入/输出（inputs/outputs）   | 高   |
| 3  | TaskOrchestrator | 使用硬编码生成 std::vector<Step>                  | 流程逻辑写死，缺乏结构表达能力   | 随任务增多，if-else 爆炸，维护困难             | 引入轻量级组合结构（Sequence/Selector/Retry），替代线性 Step 列表           | 高   |
| 4  | Bridge 分发机制   | 使用 CompositeCapabilityBridge 手动分发          | 分发逻辑集中，扩展依赖修改中心代码 | 新能力接入需改动核心模块，违反开放封闭原则             | 改为注册机制（StepType → Handler 映射），支持动态扩展                      | 中   |
| 5  | Slot 映射机制     | Slot → selected_slot → executor 内硬编码 joint | 目标解析逻辑嵌入执行层       | 扩展到更多 target_source（vision/动态点）困难 | 抽象 TargetResolver 模块，统一处理目标映射（slot/vision/fixed）          | 高   |
| 6  | Context 分层     | 当前 Context 与 RuntimeContext 职责边界不清         | 状态混用              | 生命周期与作用域不明确，易产生脏数据                | 明确三层结构：全局 Context / 执行期 RuntimeContext / Step间 Blackboard | 中   |
| 7  | Step 语义表达     | StepType + 特殊字段（如 target_source）           | 表达能力有限            | 难以扩展复杂任务语义                        | 使用通用参数结构（如 key-value 或 Variant map）增强表达能力                 | 中   |
| 8  | 执行与规划边界     | Orchestrator 与 Executor 之间部分语义未完全解耦        | 规划阶段包含部分执行假设      | 架构边界模糊                            | 明确：Orchestrator 仅负责“生成计划”，Executor 负责“解释执行”               | 中   |
| 9  | 能力层调用方式     | StepExecutor 直接调用 Bridge → Capability      | 调用链较长且耦合路径固定      | 后续多能力组合困难                         | 支持 Capability 组合与复用（Composite Capability 抽象进一步增强）         | 低   |
| 10 | 可调试性          | Step 执行过程依赖隐式状态                            | 难以追踪执行路径          | 调试成本高                             | 为 Step 增加显式日志、输入输出记录、执行 trace                             | 中   |

#### 逐步修改

1. 

自动任务框架优化执行序列（按MVP拆分）
阶段一：消除“隐式逻辑”（最高优先级，必须先做）

1. 明确 RuntimeContext 数据结构（显式化状态流）

目标：消除隐式共享变量

统一定义 RuntimeContext 字段（如 selected_slot / detected_pose）

明确：

哪些字段由哪个 Step 写入

哪些 Step 读取

禁止随意新增/隐式写入字段

👉 验收标准：

所有 Step 读写字段可追踪

不再出现“某 Step 依赖历史状态但来源不明”

2. 为 Step 定义输入/输出（数据流显式化）

目标：让 Step 之间依赖关系透明

每个 Step 明确：

inputs（依赖哪些数据）

outputs（产生哪些数据）

与 RuntimeContext 对齐

👉 验收标准：

任意 Step 的数据依赖可以静态分析

可以打印执行链路（谁产生→谁使用）

3. 抽离 SlotMapped → TargetResolver（关键解耦点）

目标：消除“语义 → 控制参数”硬编码

将：

slot → joint

vision → pose

fixed → joint

统一抽象为 TargetResolver

👉 验收标准：

Executor 中不再出现 slot 相关硬编码

新增 target_source 不需要改 executor 主流程

阶段二：规范 Step 表达（控制复杂度）
4. Step 结构纯数据化（去语义污染）

目标：Step 只描述“做什么”，不描述“怎么做”

移除：

SlotMapped

特殊 enum 行为分支

改为：

StepType + 参数（如 target_source=slot）

👉 验收标准：

Step 不包含执行逻辑判断

所有执行逻辑仅存在于 Executor / Resolver

5. Step 参数统一为通用结构（增强表达能力）

目标：避免 StepType 爆炸

使用：

key-value / variant map

支持：

动态参数扩展（如速度、精度）

👉 验收标准：

新增 Step 不需要修改结构体定义

参数扩展无需改底层类型

阶段三：稳定执行层（提高可扩展性）
6. Bridge 改为注册机制（解耦能力接入）

目标：消除集中式分发

替换：

switch-case / if-else

改为：

StepType → Handler 注册表

👉 验收标准：

新 capability 接入无需修改已有代码

支持独立模块注册

7. 增强 Step 执行日志与追踪（可调试性）

目标：让系统“可观测”

每个 Step 记录：

输入

输出

执行结果

提供执行 trace（序列打印）

👉 验收标准：

可完整复现执行路径

出错时能快速定位 Step

阶段四：优化任务编排（保持简单但更清晰）
8. TaskOrchestrator 结构化（轻量组合结构）

目标：替代纯 vector<Step>

⚠️ 注意：不引入完整行为树

引入简单结构：

Sequence（顺序执行）

可选 Retry（失败重试）

👉 验收标准：

任务结构清晰（不是一长串 Step）

不增加复杂性（仍保持固定流程）

9. Context 分层（职责清晰）

目标：避免状态污染

划分：

全局 Context（机器人状态）

RuntimeContext（执行期）

Step 数据（局部）

👉 验收标准：

不同生命周期数据不混用

重启任务不会带脏数据

阶段五：能力层增强（非必须，但推荐）
10. Capability 组合能力增强（扩展性）

目标：支持复杂能力复用

支持：

capability 内部组合（如视觉+机械臂）

优化 CompositeCapabilityBridge 结构

👉 验收标准：

复杂动作不需要在 Orchestrator 拼接

能力层可以独立复用

最终执行顺序（简版）
1. RuntimeContext 显式化
2. Step 输入/输出定义
3. TargetResolver 抽象（Slot 解耦）
4. Step 纯数据化
5. Step 参数通用化
6. Bridge 注册机制
7. Step 执行日志与追踪
8. Orchestrator 结构优化（Sequence）
9. Context 分层
10. Capability 组合增强
