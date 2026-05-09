## ENGINEER DESIGN

> 2026 工程上位机系统设计
>  按功能展开
> - 自动组合动作

### AUTOMATIC COMPOSITE ACTION

>  需求 ：

>  半自动方式机器人，预先定义组合动作细节，只需外层调度命令便可执行

	分层结构，复用+注册，单独功能包，接口清晰轻量，数据流动显式（避免隐式），扩展性强，轻量，分层合理，内部独立依赖，不要多个层级依赖重复的一些东西
	
	分层：
	
	Task Layer: 只定义步骤序列 + 显式定义每一步 + 是否依赖上一步 + 如何接受数据 + 定义每一步规则 + 输出 atomic steps
	
	Step Layer: 只执行步骤 + 检查 inputs + 输出 atomic command + 按规则处理错误（retry \ timeout \ cancel）+ 写入 outputs + 不理解业务 Spec 
	
	Capbilities Layer: 只执行 command + 返回 outputs + 只负责命令执行，错误上报 + 不做重试

**Step 规格 ：**

 - 每步包含：
      - /id（唯一）
      - kind（arm/vision/gripper/slot/data）
      - payload（原子 command 或 data 规则）
      - inputs（显式声明：从哪个 step 的哪个输出端口来）
      - outputs（显式声明：本步产生哪些端口值）
      - timeout/retry

~~~ c++
 CommandStep {
    kind: "arm.move" | "gripper.cmd" | "vision.detect" | ...
    params: {...}          // 具体参数（step 层不解释）
    inputs: [Key]          // 依赖的上下文 key（可选）
    outputs: [Key]         // 本步产出 key（可选）
    bindings: [Binding]    // 把输入 key 绑定到 params 某个字段
    timeout, retries
  }
~~~

**ContextKey 显式数据共享**

  - ContextKey = {name, scope}
  - scope 取 task | persist
  - task 只在单任务内有效
  - persist 作为“长期记忆”，跨任务有效

**Bind 绑定：**

~~~c++
Binding {
    from: "VisionPose",
    to_param: "target_pose"
  }
~~~

  step 层只做“把值塞进 params 的指定字段”，不理解含义。

 这样上下文只是一张 Key->Value 表

  - vision 产出的 VisionPose 可以被后续多个 step 读取
  - 独立 step 没有 inputs/bindings，就直接执行

**数据共享机制：**

  - 不再用“隐式 derive”，改为端口连接
  - 输入写法示例：
    inputs: [ {from: "vision_detect", port: "pose"} ]
    outputs: [ "pose", "vector" ]
  - Step 层只按连接关系取数据，不再推导

**StepExecutor执行流程：**

    1. 读取 TaskPlan（线性或 DAG 均可，先默认线性）
    2. 校验 inputs 是否在 Context 中
    3. 根据 bindings 把输入值填充进 params
    4. 把 command 交给 capability
    5. capability 返回 outputs（或者 step 定义 outputs 已知）
    6. 写回 Context（task/persist 视 scope）
    7. 进入下一步

**Capabilities职责：**

  - 按 kind 解析 params（例如 arm.move 解析 target_pose）
  - 不读 Context，不做数据派生
  - 只执行命令，输出结果或状态

**Task编排示例：**

  **AUTO_GRAB 逻辑（显式）**

  1. vision.detect
     outputs: VisionPose@task, VisionVector@task
  2. arm.move
     inputs: VisionPose@task
     bindings: VisionPose -> params.target_pose
  3. gripper.cmd
     params: {action: "close"}
  4. arm.move
     inputs: VisionVector@task
     bindings: VisionVector -> params.target_vector
  5. arm.move
     params: {target_joints: HOME}

  **AUTO_STORE 逻辑（显式）**

  1. slot.select 

     params: {strategy: "put"}

     outputs: SlotID@task

  2. arm.move
     inputs: SlotID@task 
     bindings: SlotID -> params.target_joints

  3. gripper.cmd
     params: {action: "open"}

  4. slot.lock (这个是为了做闭环用的,这个类似于select,需要事先在task layer实现好是lock?还是unlock?)
     inputs: SlotID@task
     bindings: SlotID -> params.slot_id

这样 arm capability 完全通用，不知道 slot , step 层完全“哑化” , task 层显式定义 select/lock 策略，符合你要求

这样 Step Layer 只管执行，Task Layer 完全显式，行为可读，数据共享显式

**能力层桥接机制：**

  - 仍用注册表按 command.kind 路由
  - 每个 capability 只负责一个或一组 kind

~~~c++
 ExecuteResult run(const Command &cmd)
  void cancel()
  std::string last_error()
~~~

同步服务型：execute() 里直接发请求并返回 Succeeded/Failed

异步 action 型：
- 第一次返回 Running
- 后续 tick 检查 action 状态并返回 Succeeded/Failed

**ExecuteResult结构：**

~~~c++
  ExecuteResult {
    status: Running | Succeeded | Failed
    outputs: map<string, Value>   // 可选
    error: ErrorInfo              // Failed 时必填
  }

  ErrorInfo {
    code: ErrorCode
    message: string
    retriable: bool
    detail: string (可选) (先上 std::string 就行)
  }
~~~

执行流程：

    1. StepExecutor 发送 Command
  2. Capability 返回：
      - Running：等待下一 tick
      - Succeeded：写 outputs，进入下一步
      - Failed：返回 ErrorInfo

**Delay 处理：**



**StepExecutor 核心通用机制：**

将std::any替换成std::variant,使用核心Value机制，  using Value = std::variant<
    bool,
    int64_t,
    double,
    std::string,
    std::array<double,3>,   // vector
    std::array<double,7>,   // pose
    std::array<float,6>     // joints

这样维护了StepExecutor的核心通用机制，同时使用std::variant,
