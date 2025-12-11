# Netflix Maestro 系统架构深入分析

> **Netflix Maestro** 是 Netflix 开源的下一代通用工作流编排引擎，提供完全托管的工作流即服务（Workflow-as-a-Service, WAAS）。它服务于数千名用户，每天调度数十万个工作流和数百万个作业，即使在流量高峰期也能保持严格的 SLO。

---

## 目录

- [1. 系统概述](#1-系统概述)
- [2. 模块架构](#2-模块架构)
- [3. 核心数据模型](#3-核心数据模型)
- [4. 工作流引擎设计](#4-工作流引擎设计)
- [5. Flow 执行引擎](#5-flow-执行引擎)
- [6. 触发器系统](#6-触发器系统)
- [7. 队列与消息处理](#7-队列与消息处理)
- [8. SEL 表达式语言](#8-sel-表达式语言)
- [9. 扩展性设计](#9-扩展性设计)
- [10. 最佳实践与设计亮点](#10-最佳实践与设计亮点)

---

## 1. 系统概述

### 1.1 系统定位

Maestro 是一个**通用工作流编排器**，专为以下场景设计：

- **ETL 管道**：数据抽取、转换和加载流程
- **机器学习工作流**：模型训练和推理管道
- **A/B 测试管道**：实验数据处理和分析
- **数据迁移管道**：跨存储系统的数据移动

### 1.2 技术栈

| 技术 | 版本 | 用途 |
|------|------|------|
| Java | 21 | 核心开发语言，利用虚拟线程特性 |
| Gradle | 8.x | 构建工具 |
| Spring Boot | 3.x | Web 框架与依赖注入 |
| PostgreSQL | 17 | 持久化存储 |
| Jackson | 2.x | JSON/YAML 序列化 |
| Lombok | 1.x | 代码简化 |

### 1.3 核心特性

```
┌─────────────────────────────────────────────────────────────────┐
│                    Maestro 核心特性                              │
├─────────────────────────────────────────────────────────────────┤
│  ✓ 多种工作流模式：DAG、循环、子工作流、条件分支                    │
│  ✓ 多样化任务执行：Docker、Jupyter、Bash、SQL、Python、K8s        │
│  ✓ 高度可扩展：水平扩展支持数十万工作流                            │
│  ✓ 严格 SLO：即使流量高峰也能保持服务质量                          │
│  ✓ 信号触发：支持事件驱动的工作流启动                              │
│  ✓ 时间触发：Cron 表达式定时调度                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. 模块架构

### 2.1 整体模块结构

```
maestro/
├── netflix-sel/           # SEL 表达式语言引擎
├── maestro-common/        # 通用数据模型、验证、工具类
├── maestro-dsl/           # DSL 工作流定义与解析
├── maestro-database/      # 数据库抽象层
├── maestro-flow/          # 轻量级 Flow 执行引擎
├── maestro-engine/        # 核心工作流引擎
├── maestro-queue/         # 内部作业队列系统
├── maestro-timetrigger/   # 时间触发器实现
├── maestro-signal/        # 信号触发器与依赖
├── maestro-server/        # Spring Boot 服务端
├── maestro-aws/           # AWS 集成（SNS/SQS）
├── maestro-kubernetes/    # Kubernetes 步骤运行时
└── maestro-http/          # HTTP 步骤运行时
```

### 2.2 模块依赖关系

```
                    ┌─────────────────┐
                    │  maestro-server │  (Spring Boot 应用入口)
                    └────────┬────────┘
                             │
        ┌────────────────────┼────────────────────┐
        ▼                    ▼                    ▼
┌───────────────┐   ┌───────────────┐   ┌────────────────┐
│ maestro-engine│   │maestro-signal │   │maestro-trigger │
│  (核心引擎)    │   │  (信号系统)   │   │  (触发系统)    │
└───────┬───────┘   └───────────────┘   └────────────────┘
        │
        ▼
┌───────────────┐
│ maestro-flow  │  (轻量级 Flow 执行引擎)
└───────┬───────┘
        │
        ▼
┌───────────────┐   ┌───────────────┐   ┌───────────────┐
│maestro-common │◄──│ maestro-dsl   │   │ netflix-sel   │
│  (通用模型)    │   │ (DSL 解析)    │   │ (表达式语言)  │
└───────────────┘   └───────────────┘   └───────────────┘
```

### 2.3 各模块职责详解

| 模块 | 职责 | 核心类 |
|------|------|--------|
| `netflix-sel` | 安全表达式语言，动态参数求值 | `SEL Parser`, `SEL Evaluator` |
| `maestro-common` | 数据模型、异常、工具类、验证规则 | `Workflow`, `Step`, `StepType` |
| `maestro-dsl` | YAML/JSON 工作流定义解析 | `WorkflowParser`, `DslWorkflow` |
| `maestro-flow` | 高度优化的 Flow 执行引擎（虚拟线程） | `FlowExecutor`, `FlowActor`, `TaskActor` |
| `maestro-engine` | DAG 翻译、参数求值、步骤运行时管理 | `DagTranslator`, `MaestroTask`, `StepRuntime` |
| `maestro-queue` | 基于数据库的内部作业队列 | `MaestroQueueWorker`, `MaestroJobEvent` |
| `maestro-signal` | 信号触发、依赖、输出信号处理 | `SignalHandler`, `SignalTriggerProcessor` |
| `maestro-timetrigger` | Cron 定时触发实现 | `TimeTriggerProducer`, `TimeTriggerExecutionPlanner` |

---

## 3. 核心数据模型

### 3.1 工作流定义模型 (Workflow)

```java
// maestro-common/src/main/java/com/netflix/maestro/models/definition/Workflow.java
public class Workflow {
    private String id;                          // 工作流唯一标识
    private String name;                        // 工作流名称
    private String description;                 // 描述
    private TagList tags;                       // 标签列表
    private ParsableLong timeout;               // 超时时间（秒）
    private List<TimeTrigger> timeTriggers;     // 时间触发器
    private List<SignalTrigger> signalTriggers; // 信号触发器
    private Criticality criticality;            // 关键级别
    private Long instanceStepConcurrency;       // 实例步骤并发数
    private Map<String, ParamDefinition> params;// 参数定义
    private List<Step> steps;                   // 步骤列表
}
```

### 3.2 步骤类型 (StepType)

```java
// 支持的步骤类型
public enum StepType {
    NOOP("NoOp", true),           // 空操作步骤
    SLEEP("Sleep", true),         // 休眠步骤
    TITUS("Titus", true),         // Netflix Titus 容器步骤
    NOTEBOOK("Notebook", true),   // Jupyter Notebook 步骤
    KUBERNETES("Kubernetes", true),// Kubernetes 批处理作业
    HTTP("Http", true),           // HTTP/HTTPS 调用步骤
    JOIN("Join", false),          // 汇聚步骤
    FOREACH("foreach", false),    // 循环步骤
    WHILE("while", false),        // While 循环步骤
    SUBWORKFLOW("subworkflow", false), // 子工作流步骤
    TEMPLATE("template", false);  // 模板步骤
    
    private final boolean leaf;   // 是否为叶子节点（实际执行任务）
}
```

### 3.3 步骤接口 (Step)

```java
@JsonTypeInfo(use = JsonTypeInfo.Id.NAME, include = JsonTypeInfo.As.WRAPPER_OBJECT)
@JsonSubTypes({
    @JsonSubTypes.Type(name = "step", value = TypedStep.class),
    @JsonSubTypes.Type(name = "subworkflow", value = SubworkflowStep.class),
    @JsonSubTypes.Type(name = "foreach", value = ForeachStep.class),
    @JsonSubTypes.Type(name = "while", value = WhileStep.class),
    @JsonSubTypes.Type(name = "template", value = TemplateStep.class)
})
public interface Step {
    String getId();                                    // 步骤 ID
    String getName();                                  // 步骤名称
    Map<String, ParamDefinition> getParams();          // 参数定义
    StepType getType();                                // 步骤类型
    StepTransition getTransition();                    // 转换规则
    RetryPolicy getRetryPolicy();                      // 重试策略
    SignalDependenciesDefinition getSignalDependencies(); // 信号依赖
    SignalOutputsDefinition getSignalOutputs();        // 输出信号
}
```

### 3.4 工作流实例模型 (WorkflowInstance)

```java
public class WorkflowInstance {
    private String workflowId;              // 工作流 ID
    private long workflowInstanceId;        // 实例 ID
    private long workflowRunId;             // 运行 ID
    private String workflowUuid;            // 唯一 UUID（去重）
    private long workflowVersionId;         // 版本 ID
    private RunConfig runConfig;            // 运行配置
    private Map<String, ParamDefinition> runParams;    // 运行参数覆盖
    private Workflow runtimeWorkflow;       // 运行时工作流定义
    private Map<String, StepTransition> runtimeDag;    // 运行时 DAG
    private Initiator initiator;            // 发起者
    private Status status;                  // 状态
    private Long createTime;                // 创建时间
    private Long startTime;                 // 开始时间
    private Long endTime;                   // 结束时间
    
    public enum Status {
        CREATED(false),      // 已创建，排队中
        IN_PROGRESS(false),  // 运行中
        PAUSED(false),       // 已暂停
        TIMED_OUT(true),     // 超时（终态）
        STOPPED(true),       // 已停止（终态）
        FAILED(true),        // 失败（终态）
        SUCCEEDED(true);     // 成功（终态）
    }
}
```

### 3.5 示例工作流 JSON

```json
{
  "properties": {
    "owner": "tester",
    "run_strategy": "sequential"
  },
  "workflow": {
    "id": "sample-dag-test-1",
    "name": "Test workflow 01",
    "description": "示例 DAG 工作流",
    "params": {
      "foo": {
        "expression": "new DateTime().monthOfYear().getAsText();",
        "type": "STRING"
      }
    },
    "steps": [
      {"step": {"id": "job.1", "type": "NoOp", "transition": {"successors": {"job.5": "true", "job.2": "true"}}}},
      {"step": {"id": "job.2", "type": "NoOp", "transition": {"successors": {"job.3": "true"}}}},
      {"step": {"id": "job.3", "type": "NoOp", "transition": {"successors": {"job.4": "true"}}}},
      {"step": {"id": "job.4", "type": "NoOp", "transition": {}}},
      {"step": {"id": "job.5", "type": "NoOp", "transition": {"successors": {"job.3": "true"}}}}
    ]
  }
}
```

---

## 4. 工作流引擎设计

### 4.1 引擎架构

```
┌─────────────────────────────────────────────────────────────────────┐
│                         Maestro Engine                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────────┐ │
│  │ Workflow    │    │ DAG         │    │ Step Runtime            │ │
│  │ Handler     │───▶│ Translator  │───▶│ Manager                 │ │
│  └─────────────┘    └─────────────┘    └─────────────────────────┘ │
│         │                                        │                  │
│         ▼                                        ▼                  │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────────┐ │
│  │ Workflow    │    │ Param       │    │ Step Runtimes:          │ │
│  │ Runner      │    │ Evaluator   │    │ - NoOpStepRuntime       │ │
│  └─────────────┘    └─────────────┘    │ - SleepStepRuntime      │ │
│         │                              │ - ForeachStepRuntime    │ │
│         ▼                              │ - SubworkflowStepRuntime│ │
│  ┌─────────────────────────────────┐   │ - KubernetesStepRuntime │ │
│  │        Flow Engine              │   │ - HttpStepRuntime       │ │
│  └─────────────────────────────────┘   └─────────────────────────┘ │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 4.2 DAG 翻译器 (DagTranslator)

DAG 翻译器负责将工作流实例转换为可执行的 DAG 结构：

```java
public class DagTranslator implements Translator<WorkflowInstance, Map<String, StepTransition>> {
    
    @Override
    public Map<String, StepTransition> translate(WorkflowInstance workflowInstance) {
        // 1. 克隆工作流实例（避免修改原对象）
        WorkflowInstance instance = objectMapper.convertValue(workflowInstance, WorkflowInstance.class);
        
        // 2. 处理重启策略
        if (instance.getRunConfig().getPolicy() == RunPolicy.RESTART_FROM_INCOMPLETE
            || instance.getRunConfig().getPolicy() == RunPolicy.RESTART_FROM_SPECIFIC) {
            // 计算需要重新执行的步骤
            Map<String, StepInstance.Status> statusMap = computeStatusMap(instance);
            instance.getRunConfig().setStartStepIds(computeRestartSteps(statusMap));
        }
        
        // 3. 计算运行时 DAG
        return WorkflowGraph.computeDag(
            instance.getRuntimeWorkflow(), 
            startStepIds, 
            endStepIds
        );
    }
}
```

### 4.3 工作流图算法 (WorkflowGraph)

```java
final class WorkflowGraph {
    
    // 计算 DAG 结构
    public static Map<String, StepTransition> computeDag(
            Workflow workflow, 
            List<String> startStepIds, 
            List<String> endStepIds) {
        
        // 1. 构建步骤映射
        Map<String, Step> stepMap = buildStepMap(workflow);
        
        // 2. 处理起始和结束步骤
        if (startStepIds != null) {
            stepMap = filterReachableSteps(stepMap, startStepIds, endStepIds);
        }
        
        // 3. 构建图节点
        Map<String, GraphNode> nodeMap = computeNodeMap(workflow.getId(), stepMap);
        
        // 4. 检测循环依赖
        Checks.checkTrue(
            !containsCycleInDag(nodeMap),
            "Invalid workflow definition [%s], DAG contains cycle",
            workflow.getId()
        );
        
        return stepMap.values().stream()
            .collect(Collectors.toMap(Step::getId, Step::getTransition));
    }
    
    // 使用拓扑排序检测循环
    private static boolean containsCycleInDag(Map<String, GraphNode> nodeMap) {
        Queue<String> queue = new ArrayDeque<>();
        // 入度为 0 的节点入队
        nodeMap.forEach((stepId, node) -> {
            if (node.parents.isEmpty()) {
                queue.add(stepId);
            }
        });
        
        int visited = 0;
        while (!queue.isEmpty()) {
            String stepId = queue.remove();
            visited++;
            // 移除边，更新入度
            for (String childId : nodeMap.get(stepId).children.keySet()) {
                nodeMap.get(childId).parents.remove(stepId);
                if (nodeMap.get(childId).parents.isEmpty()) {
                    queue.add(childId);
                }
            }
        }
        return visited != nodeMap.size();
    }
}
```

### 4.4 工作流运行器 (WorkflowRunner)

```java
public class WorkflowRunner {
    private final MaestroFlowDao flowDao;
    private final FlowOperation flowOperation;
    private final WorkflowTranslator translator;
    private final WorkflowHelper workflowHelper;
    
    public void run(WorkflowInstance instance, String uuid) {
        if (instance.getStatus() == WorkflowInstance.Status.CREATED) {
            if (uuid.equals(instance.getWorkflowUuid())) {
                // 去重检查
                if (!flowDao.existFlowWithSameKeys(
                        IdHelper.deriveGroupId(instance), 
                        instance.getWorkflowUuid())) {
                    String executionId = runWorkflowInstance(instance);
                    LOG.info("Run workflow instance {} with execution_id [{}]",
                        instance.getIdentity(), executionId);
                }
            }
        }
    }
    
    String start(WorkflowInstance instance) {
        return flowOperation.startFlow(
            IdHelper.deriveGroupId(instance),           // 分组 ID
            instance.getWorkflowUuid(),                  // 流程 ID
            IdHelper.deriveFlowRef(instance),            // 引用
            translator.translate(instance),              // 翻译后的 FlowDef
            workflowHelper.createWorkflowSummaryFromInstance(instance)
        );
    }
}
```

### 4.5 步骤运行时接口 (StepRuntime)

```java
public interface StepRuntime {
    
    /**
     * 步骤启动逻辑
     * - 至少执行一次保证
     * - 需要幂等实现
     */
    default Result start(WorkflowSummary workflowSummary, Step step, StepRuntimeSummary runtimeSummary) {
        return Result.of(State.DONE);
    }
    
    /**
     * 步骤执行逻辑
     * - 周期性调用（轮询模式）
     * - 异常时触发重试
     */
    default Result execute(WorkflowSummary workflowSummary, Step step, StepRuntimeSummary runtimeSummary) {
        return Result.of(State.DONE);
    }
    
    /**
     * 步骤终止逻辑
     * - 工作流终止时调用
     */
    default Result terminate(WorkflowSummary workflowSummary, StepRuntimeSummary runtimeSummary) {
        return Result.of(State.STOPPED);
    }
    
    // 运行时状态
    enum State {
        CONTINUE,      // 继续当前状态
        DONE,          // 完成，进入下一状态
        USER_ERROR,    // 用户错误
        PLATFORM_ERROR,// 平台错误
        FATAL_ERROR,   // 致命错误
        STOPPED,       // 已停止
        TIMED_OUT;     // 超时
    }
}
```

---

## 5. Flow 执行引擎

### 5.1 设计理念

Flow 执行引擎是 Maestro 的核心创新之一，具有以下特点：

1. **高度优化**：专为 Maestro 引擎设计，非通用 DAG 引擎
2. **虚拟线程**：利用 Java 21 虚拟线程简化并发处理
3. **Actor 模型**：借鉴 Actor 模型和数据流编程模型的优点
4. **内存优化**：任务可在单机内存中协同处理

### 5.2 Actor 层次结构

```
┌─────────────────────────────────────────────────────────────────┐
│                      FlowExecutor (单例)                         │
│  - 管理所有 GroupActor                                           │
│  - 周期性维护（心跳、所有权检查）                                   │
│  - 声明过期组的所有权                                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │ GroupActor 1 │  │ GroupActor 2 │  │ GroupActor N │          │
│  │  (分组管理)   │  │  (分组管理)   │  │  (分组管理)   │          │
│  └──────┬───────┘  └──────────────┘  └──────────────┘          │
│         │                                                       │
│  ┌──────┴───────────────────────────────────────────┐          │
│  │                                                  │          │
│  │  ┌───────────┐  ┌───────────┐  ┌───────────┐    │          │
│  │  │ FlowActor │  │ FlowActor │  │ FlowActor │    │          │
│  │  │  (流程)   │  │  (流程)   │  │  (流程)   │    │          │
│  │  └─────┬─────┘  └───────────┘  └───────────┘    │          │
│  │        │                                         │          │
│  │  ┌─────┴─────────────────────────────┐          │          │
│  │  │ TaskActor  TaskActor  TaskActor   │          │          │
│  │  │  (任务)     (任务)     (任务)      │          │          │
│  │  └───────────────────────────────────┘          │          │
│  │                                                  │          │
│  └──────────────────────────────────────────────────┘          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 5.3 FlowExecutor 实现

```java
@Slf4j
public class FlowExecutor {
    private final Map<Long, Actor> groupActors = new ConcurrentHashMap<>();
    private final ReadWriteLock lock = new ReentrantReadWriteLock();
    
    // 初始化：启动周期性维护任务
    public void init() {
        maintainer.scheduleWithFixedDelay(
            this::maintenance, 
            initialDelay, 
            delay, 
            TimeUnit.MILLISECONDS
        );
    }
    
    // 周期性维护：检查所有权、声明过期组
    private void maintenance() {
        // 1. 检查当前组的所有权有效性
        groupActors.forEach((groupId, actor) -> {
            if (actor.validUntil() < System.currentTimeMillis()) {
                // 所有权失效，终止 JVM 以进行协调
                Runtime.getRuntime().halt(Constants.INVALID_OWNERSHIP_EXIT_CODE);
            }
        });
        
        // 2. 尝试声明新的组（如果当前节点组数未达上限）
        if (groupActors.size() < maxGroupNumPerNode) {
            FlowGroup group = context.claimGroup();
            if (group != null) {
                groupActors.computeIfAbsent(group.groupId(), 
                    id -> Actor.startGroupActor(group, context));
            }
        }
    }
    
    // 启动新流程
    public String startFlow(long groupId, String flowId, String reference, 
                           FlowDef flowDef, Map<String, Object> flowInput) {
        Actor groupActor = getOrCreateNewGroup(groupId);
        
        // 构建流程对象
        Flow flow = new Flow(groupId, flowId, groupActor.generation(), 
                            System.currentTimeMillis(), reference);
        flow.setInput(flowInput);
        flow.setFlowDef(flowDef);
        flow.setStatus(Flow.Status.RUNNING);
        
        // 持久化并启动
        context.saveFlow(flow);
        groupActor.post(new Action.FlowLaunch(flow, false));
        
        return flow.getFlowId();
    }
}
```

### 5.4 FlowActor 状态机

```java
final class FlowActor extends BaseActor {
    
    @Override
    void runForAction(Action action) {
        switch (action) {
            case Action.FlowStart fs -> startFlow(fs);
            case Action.FlowReconcile r -> reconcile(r);
            case Action.FlowRefresh fr -> refresh();
            case Action.FlowTaskRetry t -> retryTask(t.taskRefName());
            case Action.TaskUpdate u -> updateFlow(u.updatedTask());
            case Action.TaskWakeUp w -> wakeup(w.taskRef(), w.code());
            case Action.FlowTimeout ft -> timeoutFlow();
            case Action.FlowShutdown sd -> startShutdown(Action.TASK_SHUTDOWN);
            case Action.TaskDown td -> checkShutdown();
            default -> throw new MaestroUnprocessableEntityException(
                "Unexpected action: [%s] for flow %s", action, reference());
        }
    }
    
    // 决策逻辑：推进流程执行
    private void decide() {
        Map<String, Task.Status> taskStatusMap = buildTaskStatusMap();
        
        // 遍历任务定义，启动可执行的任务
        flow.getFlowDef().getTasks().stream()
            .map(t -> nextTask(t, taskStatusMap))
            .filter(Objects::nonNull)
            .forEach(this::startTask);
        
        // 刷新状态并尝试终止
        getContext().refresh(flow);
        tryTerminate();
    }
    
    // 尝试终止流程
    private void tryTerminate() {
        var status = flow.getMonitorTask().getStatus();
        if (status.isTerminal()) {
            if (status.isSuccessful()) {
                flow.setStatus(Flow.Status.COMPLETED);
            } else {
                flow.setStatus(Flow.Status.FAILED);
            }
            getContext().finalCall(flow);
            finalized = true;
            terminateNow();
        }
    }
}
```

### 5.5 Flow 数据模型

```java
@Data
public class Flow {
    // 不可变数据（持久化到数据库）
    private final long groupId;      // 分组 ID
    private final String flowId;     // 流程 ID
    private final long generation;   // 代数（所有权）
    private final long startTime;    // 开始时间
    private final String reference;  // 引用标识
    
    // 瞬态不可变数据
    private Map<String, Object> input;  // 输入参数
    private FlowDef flowDef;            // 流程定义
    
    // 瞬态可变数据
    private volatile Status status;     // 状态
    private Task prepareTask;           // 准备任务
    private Task monitorTask;           // 监控任务
    private final Map<String, Task> finishedTasks;  // 已完成任务
    private final Map<String, Task> runningTasks;   // 运行中任务
    
    public enum Status {
        RUNNING(false, false),
        COMPLETED(true, true),
        FAILED(true, false),
        TIMED_OUT(true, false),
        TERMINATED(true, false);
    }
}
```

---

## 6. 触发器系统

### 6.1 时间触发器

```
┌─────────────────────────────────────────────────────────────────┐
│                    Time Trigger System                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────┐    ┌─────────────────┐                    │
│  │ TimeTrigger     │    │ TimeTrigger     │                    │
│  │ Producer        │───▶│ Execution       │                    │
│  │ (生成触发计划)   │    │ Planner         │                    │
│  └─────────────────┘    └────────┬────────┘                    │
│                                  │                              │
│                                  ▼                              │
│  ┌─────────────────────────────────────────────────────┐       │
│  │              Delay Queue (延迟队列)                  │       │
│  │              - AWS SQS 实现                          │       │
│  │              - 或其他队列系统                         │       │
│  └────────────────────────┬────────────────────────────┘       │
│                           │                                     │
│                           ▼                                     │
│  ┌─────────────────────────────────────────────────────┐       │
│  │         TimeTriggerExecutionProcessor               │       │
│  │         (处理到期的触发并启动工作流)                   │       │
│  └─────────────────────────────────────────────────────┘       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 6.2 信号触发器

信号系统支持三种核心功能：

1. **信号触发（Signal Triggers）**：通过信号启动工作流
2. **信号依赖（Signal Dependencies）**：步骤等待信号就绪
3. **输出信号（Output Signals）**：步骤完成后发送信号

```
┌─────────────────────────────────────────────────────────────────┐
│                    Signal System                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────┐                      ┌─────────────────────┐  │
│  │ Signal API  │─────────────────────▶│ SignalInstanceDao   │  │
│  │ (发送信号)   │                      │ (持久化信号)        │  │
│  └─────────────┘                      └──────────┬──────────┘  │
│                                                  │              │
│       ┌──────────────────────────────────────────┘              │
│       ▼                                                         │
│  ┌─────────────────────────────────────────────────────┐       │
│  │           Signal Trigger Match Processor             │       │
│  │           (匹配信号与工作流触发条件)                   │       │
│  └────────────────────────┬────────────────────────────┘       │
│                           │                                     │
│       ┌───────────────────┼───────────────────┐                │
│       ▼                   ▼                   ▼                │
│  ┌─────────┐        ┌─────────┐        ┌─────────────┐        │
│  │ Trigger │        │ Signal  │        │ Output      │        │
│  │ Workflow│        │ Depend  │        │ Signal      │        │
│  │ (触发)  │        │ (依赖)  │        │ (输出)      │        │
│  └─────────┘        └─────────┘        └─────────────┘        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 6.3 信号使用示例

```bash
# 创建带信号触发的工作流
curl -X POST 'http://127.0.0.1:8080/api/v3/workflows' \
    -H "Content-Type: application/json" \
    -d @sample-signal-trigger-wf.json

# 发送信号触发工作流
curl -X POST 'http://127.0.0.1:8080/api/v3/signals' \
    -H "Content-Type: application/json" \
    -d '{
        "name": "signal_a", 
        "params": {
            "foo": "bar", 
            "watermark": 123456
        }
    }'
```

---

## 7. 队列与消息处理

### 7.1 内部作业队列

Maestro 使用基于数据库的内部作业队列系统：

```java
// 作业事件类型
public abstract class MaestroJobEvent {
    // 队列 ID 定义
    public static final int WORKFLOW_START_QUEUE = 1;
    public static final int WORKFLOW_INSTANCE_UPDATE_QUEUE = 2;
    public static final int STEP_INSTANCE_UPDATE_QUEUE = 3;
    public static final int NOTIFICATION_QUEUE = 4;
    public static final int WORKFLOW_DELETE_QUEUE = 5;
}

// 具体事件类型
public class StartWorkflowJobEvent extends MaestroJobEvent { }
public class InstanceActionJobEvent extends MaestroJobEvent { }
public class NotificationJobEvent extends MaestroJobEvent { }
public class DeleteWorkflowJobEvent extends MaestroJobEvent { }
```

### 7.2 队列配置

```yaml
# application.yml
maestro:
  queue:
    properties:
      1:  # WORKFLOW_START_QUEUE
        worker-num: 8
        scan-interval: 5000
      2:  # WORKFLOW_INSTANCE_UPDATE_QUEUE
        worker-num: 12
        scan-interval: 5000
      3:  # STEP_INSTANCE_UPDATE_QUEUE
        worker-num: 17
        scan-interval: 5000
      4:  # NOTIFICATION_QUEUE
        worker-num: 3
        scan-interval: 5000
      5:  # WORKFLOW_DELETE_QUEUE
        worker-num: 3
        scan-interval: 30000
        ownership-timeout: 125000
```

### 7.3 事件处理器

```java
// 事件分发器
public class MaestroJobEventDispatcher {
    private final Map<Class<?>, MaestroEventProcessor<?>> processors;
    
    public void dispatch(MaestroJobEvent event) {
        MaestroEventProcessor processor = processors.get(event.getClass());
        processor.process(event);
    }
}

// 工作流启动处理器
public class StartWorkflowJobEventProcessor implements MaestroEventProcessor<StartWorkflowJobEvent> {
    
    @Override
    public void process(StartWorkflowJobEvent event) {
        WorkflowInstance instance = instanceDao.getWorkflowInstance(event);
        workflowRunner.run(instance, event.getWorkflowUuid());
    }
}
```

---

## 8. SEL 表达式语言

### 8.1 概述

SEL（Simple Expression Language）是 Maestro 的内置表达式语言，用于动态参数求值：

- **简单**：语法遵循 Java 语言规范（JLS）子集
- **安全**：基于 Java 安全特性的权限控制
- **可靠**：严格的运行时检查（循环限制、数组大小等）

### 8.2 配置

```yaml
maestro:
  sel:
    thread-num: 3           # 执行线程数
    timeout-millis: 120000  # 超时时间
    stack-limit: 128        # 栈深度限制
    loop-limit: 25001       # 循环次数限制
    array-limit: 25001      # 数组大小限制
    length-limit: 10000     # 字符串长度限制
    visit-limit: 100000000  # 访问次数限制
    memory-limit: 100000000 # 内存限制
```

### 8.3 使用示例

```json
{
  "params": {
    "current_month": {
      "expression": "new DateTime().withZone(DateTimeZone.forID('UTC')).monthOfYear().getAsText();",
      "type": "STRING"
    },
    "calculated_date": {
      "expression": "new DateTime(1569018000000).plusDays(7).toString('yyyy-MM-dd');",
      "type": "STRING"
    },
    "is_weekend": {
      "expression": "new DateTime().dayOfWeek().get() >= 6;",
      "type": "BOOLEAN"
    }
  }
}
```

---

## 9. 扩展性设计

### 9.1 Kubernetes 集成

```java
// Kubernetes 运行时执行器接口
public interface KubernetesRuntimeExecutor {
    String launch(KubernetesJobConfig config);
    JobStatus getStatus(String jobId);
    void terminate(String jobId);
}

// Fabric8 客户端实现
public class Fabric8RuntimeExecutor implements KubernetesRuntimeExecutor {
    private final KubernetesClient client;
    
    @Override
    public String launch(KubernetesJobConfig config) {
        Job job = new JobBuilder()
            .withNewMetadata()
                .withName(config.getJobName())
                .withNamespace(config.getNamespace())
            .endMetadata()
            .withNewSpec()
                .withTemplate(config.getPodTemplate())
            .endSpec()
            .build();
        
        client.batch().v1().jobs().create(job);
        return job.getMetadata().getUid();
    }
}
```

### 9.2 AWS 集成

```yaml
# application-aws.yml
spring:
  profiles: aws

aws:
  sns:
    endpoint: http://localhost:4566
    region: us-east-1
  sqs:
    endpoint: http://localhost:4566
    region: us-east-1
```

### 9.3 自定义步骤运行时

```java
// 实现自定义步骤运行时
public class CustomStepRuntime implements StepRuntime {
    
    @Override
    public Result start(WorkflowSummary workflowSummary, Step step, 
                       StepRuntimeSummary runtimeSummary) {
        // 自定义启动逻辑
        return Result.of(State.DONE);
    }
    
    @Override
    public Result execute(WorkflowSummary workflowSummary, Step step, 
                         StepRuntimeSummary runtimeSummary) {
        // 自定义执行逻辑
        if (isCompleted()) {
            return Result.of(State.DONE);
        }
        return Result.of(State.CONTINUE, 5000L); // 5秒后再次检查
    }
    
    @Override
    public Map<String, ParamDefinition> injectRuntimeParams(
            WorkflowSummary workflowSummary, Step step) {
        // 注入运行时参数
        return Map.of("custom_param", ParamDefinition.of("value"));
    }
}
```

---

## 10. 最佳实践与设计亮点

### 10.1 设计亮点

#### 10.1.1 虚拟线程优化

Maestro 利用 Java 21 的虚拟线程特性，大大简化了并发处理：

```java
// Flow 执行引擎使用虚拟线程
public class FlowExecutor {
    // 轻量级线程，可创建大量并发任务
    private final ExecutorService virtualThreadExecutor = 
        Executors.newVirtualThreadPerTaskExecutor();
}
```

#### 10.1.2 分组管理

通过分组实现负载均衡和故障隔离：

```
节点 A                节点 B                节点 C
┌─────────┐          ┌─────────┐          ┌─────────┐
│ Group 1 │          │ Group 2 │          │ Group 3 │
│ Group 4 │          │ Group 5 │          │ Group 6 │
│ Group 7 │          │ Group 8 │          │ Group 9 │
└─────────┘          └─────────┘          └─────────┘

- 每个节点管理多个组
- 组内流程共享状态
- 组间完全隔离
- 故障时自动重新分配
```

#### 10.1.3 至少一次语义

所有步骤执行保证至少执行一次，要求幂等实现：

```java
public interface StepRuntime {
    /**
     * 至少执行一次保证
     * - 故障恢复时可能重复执行
     * - 实现必须幂等
     */
    default Result start(WorkflowSummary summary, Step step, 
                        StepRuntimeSummary runtime) {
        // 幂等实现示例
        if (alreadyStarted(runtime)) {
            return Result.of(State.CONTINUE);
        }
        doStart();
        return Result.of(State.DONE);
    }
}
```

### 10.2 性能优化建议

1. **合理设置并发**
```yaml
engine:
  configs:
    internal-worker-num: 80  # 根据 CPU 核心数调整
```

2. **调整队列工作线程**
```yaml
maestro:
  queue:
    properties:
      3:  # 步骤更新队列（最繁忙）
        worker-num: 17  # 增加工作线程
```

3. **优化数据库连接池**
```yaml
engine:
  configs:
    workflow-database-connection-pool-size-max: 8
    workflow-database-connection-pool-idle-min: 8
```

### 10.3 监控指标

Maestro 使用 Netflix Spectator 进行指标收集：

```java
public class MaestroMetricRepo implements MaestroMetrics {
    private final Registry registry;
    
    public void counter(String name, Class<?> clazz, String... tags) {
        registry.counter(name, tags).increment();
    }
    
    public void timer(String name, long duration, Class<?> clazz) {
        registry.timer(name).record(duration, TimeUnit.MILLISECONDS);
    }
}
```

关键监控指标：

| 指标名 | 描述 |
|--------|------|
| `num_of_running_flows` | 运行中的流程数 |
| `num_of_finished_flows` | 已完成的流程数 |
| `num_of_wakeup_flows` | 唤醒的流程数 |
| `step_initialize_delay` | 步骤初始化延迟 |
| `num_of_groups` | 当前节点管理的组数 |

---

## 11. 步骤状态机详解

### 11.1 步骤状态定义

```java
public enum Status {
    // 初始化阶段
    NOT_CREATED(false, false, false, false),  // 未创建（占位符）
    CREATED(false, false, false, false),       // 已创建
    INITIALIZED(false, false, false, false),   // 已初始化
    
    // 等待阶段
    PAUSED(false, false, true, false),         // 暂停（断点）
    WAITING_FOR_SIGNALS(false, false, true, false),   // 等待信号
    EVALUATING_PARAMS(false, false, true, false),     // 参数求值中
    WAITING_FOR_PERMITS(false, false, true, false),   // 等待标签许可
    
    // 执行阶段
    STARTING(false, false, false, false),      // 启动中
    RUNNING(false, false, true, false),        // 运行中
    FINISHING(false, false, false, false),     // 完成中
    
    // 成功终态
    DISABLED(true, true, false, false),        // 已禁用
    UNSATISFIED(true, true, false, false),     // 条件不满足
    SKIPPED(true, true, false, false),         // 已跳过
    SUCCEEDED(true, true, false, false),       // 成功
    COMPLETED_WITH_ERROR(true, true, false, false),  // 带错误完成
    
    // 失败终态（可重试）
    USER_FAILED(true, false, true, true),      // 用户错误（自动重试）
    PLATFORM_FAILED(true, false, true, true),  // 平台错误（自动重试）
    TIMEOUT_FAILED(true, false, true, true),   // 超时失败（自动重试）
    
    // 失败终态（不可重试）
    FATALLY_FAILED(true, false, true, false),  // 致命失败
    INTERNALLY_FAILED(true, false, true, false), // 内部错误
    STOPPED(true, false, false, false),        // 已停止
    TIMED_OUT(true, false, false, false);      // 超时终止
}
```

### 11.2 步骤状态转换图

```
                                ┌─────────────────────────────────────────────────────────────┐
                                │                     步骤状态机                               │
                                └─────────────────────────────────────────────────────────────┘

                                              ┌───────────────┐
                                              │  NOT_CREATED  │ ◄─── 占位符任务（条件不满足时）
                                              └───────────────┘
                                                     │ RESTART action
                                                     ▼
┌─────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                        正常执行流程                                               │
├─────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                 │
│    ┌─────────┐    ┌─────────────┐    ┌─────────┐    ┌──────────────────┐                       │
│    │ CREATED │───▶│ INITIALIZED │───▶│ PAUSED  │───▶│ WAITING_FOR_     │                       │
│    │ (创建)  │    │  (初始化)   │    │ (暂停)  │    │ SIGNALS (等信号) │                       │
│    └─────────┘    └─────────────┘    └────┬────┘    └────────┬─────────┘                       │
│                                           │ resume            │ signals ready                   │
│                                           └───────────────────┤                                 │
│                                                               ▼                                 │
│    ┌─────────┐    ┌─────────────┐    ┌──────────────────┐    ┌──────────────────┐             │
│    │FINISHING│◄───│   RUNNING   │◄───│     STARTING     │◄───│ EVALUATING_PARAMS│             │
│    │ (完成中)│    │   (运行中)  │    │     (启动中)     │    │   (参数求值)     │             │
│    └────┬────┘    └──────┬──────┘    └──────────────────┘    └────────┬─────────┘             │
│         │                │                     ▲                      │ params evaluated       │
│         │                │                     │                      ▼                        │
│         │                │              ┌──────────────────┐                                   │
│         │                │              │ WAITING_FOR_     │                                   │
│         │                │              │ PERMITS (等许可) │                                   │
│         │                │              └──────────────────┘                                   │
│         │                │                                                                     │
└─────────┼────────────────┼─────────────────────────────────────────────────────────────────────┘
          │                │
          │                │
          ▼                │
    ┌─────────────┐        │              ┌─────────────────────────────────────────────────────┐
    │  SUCCEEDED  │        │              │                    失败处理流程                      │
    │   (成功)    │        │              ├─────────────────────────────────────────────────────┤
    └─────────────┘        │              │                                                     │
                           │              │  ┌─────────────┐        ┌─────────────────────┐    │
    ┌─────────────┐        ├─────────────▶│  │ USER_FAILED │───────▶│ 达到重试限制?       │    │
    │ UNSATISFIED │        │ user error   │  │ (用户错误)  │        │ ├─ 否: 自动重试     │    │
    │ (条件不满足)│        │              │  └─────────────┘        │ └─ 是: FATALLY_FAILED│   │
    └─────────────┘        │              │                         └─────────────────────┘    │
                           │              │                                                     │
    ┌─────────────┐        ├─────────────▶│  ┌───────────────┐      ┌─────────────────────┐   │
    │   SKIPPED   │        │ platform err │  │PLATFORM_FAILED│─────▶│ 达到重试限制?       │    │
    │  (已跳过)   │        │              │  │ (平台错误)    │      │ ├─ 否: 自动重试     │    │
    └─────────────┘        │              │  └───────────────┘      │ └─ 是: FATALLY_FAILED│   │
                           │              │                         └─────────────────────┘    │
    ┌─────────────┐        ├─────────────▶│  ┌───────────────┐      ┌─────────────────────┐   │
    │  DISABLED   │        │ timeout      │  │ TIMEOUT_FAILED│─────▶│ 达到重试限制?       │    │
    │  (已禁用)   │        │              │  │ (超时失败)    │      │ ├─ 否: 自动重试     │    │
    └─────────────┘        │              │  └───────────────┘      │ └─ 是: TIMED_OUT    │    │
                           │              │                         └─────────────────────┘    │
    ┌──────────────────┐   │              │                                                     │
    │COMPLETED_WITH_   │◄──┤ failure mode │  ┌───────────────────┐                             │
    │ERROR (带错误完成)│   │ = IGNORE     │  │ INTERNALLY_FAILED │ ◄── Maestro 内部错误        │
    └──────────────────┘   │              │  │   (内部失败)      │                             │
                           │              │  └───────────────────┘                             │
    ┌─────────────┐        │              │                                                     │
    │   STOPPED   │◄───────┴──────────────│  ┌───────────────────┐                             │
    │  (已停止)   │  STOP/KILL action     │  │  FATALLY_FAILED   │                             │
    └─────────────┘                       │  │   (致命失败)      │                             │
                                          │  └───────────────────┘                             │
    ┌─────────────┐                       │                                                     │
    │  TIMED_OUT  │◄──────────────────────│  ┌───────────────────┐                             │
    │  (超时终止) │  workflow timeout     │  │     TIMED_OUT     │                             │
    └─────────────┘                       │  │    (超时终止)     │                             │
                                          │  └───────────────────┘                             │
                                          └─────────────────────────────────────────────────────┘
```

### 11.3 MaestroTask 执行核心逻辑

```java
// MaestroTask.doExecute() 状态机实现
private boolean doExecute(Flow flow, Task task, WorkflowSummary workflowSummary,
                         Step stepDefinition, StepRuntimeSummary runtimeSummary) {
    boolean doneWithExecute = false;
    while (!doneWithExecute) {
        switch (runtimeSummary.getRuntimeState().getStatus()) {
            
            // ===== 初始化阶段 =====
            case NOT_CREATED:
                return true;  // 占位符，保持非终态
                
            case CREATED:
                // 初始化步骤：检查条件、超时、依赖等
                doneWithExecute = initialize(flow, task, stepDefinition, 
                                            workflowSummary, runtimeSummary);
                break;
                
            case INITIALIZED:
                // 检查是否需要暂停（断点调试）
                if (stepBreakpointDao.createPausedStepAttemptIfNeeded(...)) {
                    runtimeSummary.markPaused(tracingManager);
                } else {
                    runtimeSummary.markWaitSignal(tracingManager);
                }
                break;
            
            // ===== 等待阶段 =====
            case PAUSED:
                if (stepBreakpointDao.shouldStepResume(...)) {
                    runtimeSummary.markWaitSignal(tracingManager);
                } else {
                    doneWithExecute = true;  // 继续暂停
                }
                break;
                
            case WAITING_FOR_SIGNALS:
                if (signalsReady(workflowSummary, runtimeSummary)) {
                    runtimeSummary.markEvaluateParam(tracingManager);
                } else {
                    doneWithExecute = true;  // 继续等待
                }
                break;
                
            case EVALUATING_PARAMS:
                doneWithExecute = evaluateParams(flow, task, stepDefinition, 
                                                workflowSummary, runtimeSummary);
                break;
                
            case WAITING_FOR_PERMITS:
                if (permitsReady(flow, workflowSummary, runtimeSummary)) {
                    runtimeSummary.markStarting(tracingManager);
                    task.setStartTime(runtimeSummary.getRuntimeState().getStartTime());
                } else {
                    doneWithExecute = true;  // 继续等待
                }
                break;
            
            // ===== 执行阶段 =====
            case STARTING:
                doneWithExecute = stepRuntimeManager.start(workflowSummary, 
                                                          stepDefinition, runtimeSummary);
                break;
                
            case RUNNING:
                doneWithExecute = stepRuntimeManager.execute(workflowSummary, 
                                                            stepDefinition, runtimeSummary);
                break;
                
            case FINISHING:
                // 验证输出参数和工件
                outputDataManager.validateAndMergeOutputParamsAndArtifacts(runtimeSummary);
                // 发送输出信号
                if (initializeAndSendOutputSignals(flow, stepDefinition, 
                                                  workflowSummary, runtimeSummary)) {
                    runtimeSummary.markTerminated(StepInstance.Status.SUCCEEDED, tracingManager);
                }
                break;
            
            // ===== 终态处理 =====
            case UNSATISFIED:
            case DISABLED:
            case SKIPPED:
            case SUCCEEDED:
            case COMPLETED_WITH_ERROR:
                evaluateNextConditionParams(flow, stepDefinition, runtimeSummary);
                doneWithExecute = true;
                break;
                
            case FATALLY_FAILED:
                // 处理失败模式
                if (FailureMode.IGNORE_FAILURE == stepDefinition.getFailureMode()) {
                    runtimeSummary.markTerminated(
                        StepInstance.Status.COMPLETED_WITH_ERROR, tracingManager);
                } else if (FailureMode.FAIL_IMMEDIATELY == stepDefinition.getFailureMode()) {
                    terminateAllSteps(flow, workflowSummary, stepDefinition.getId());
                }
                doneWithExecute = true;
                break;
                
            case USER_FAILED:
            case PLATFORM_FAILED:
            case TIMEOUT_FAILED:
            case INTERNALLY_FAILED:
            case STOPPED:
            case TIMED_OUT:
                doneWithExecute = true;
                break;
        }
    }
    return false;
}
```

### 11.4 重试机制详解

```java
// 步骤重试信息
public static class StepRetry {
    private long errorRetries;         // 用户错误重试次数
    private long errorRetryLimit;       // 用户错误重试上限
    private long platformRetries;       // 平台错误重试次数
    private long platformRetryLimit;    // 平台错误重试上限
    private long timeoutRetries;        // 超时错误重试次数
    private long timeoutRetryLimit;     // 超时错误重试上限
    private long manualRetries;         // 手动重启次数
    private boolean retryable;          // 是否可由系统重试
    private RetryPolicy.Backoff backoff;// 退避策略
    
    // 计算下次重试延迟
    public int getNextRetryDelay(Status status) {
        if (status == Status.USER_FAILED) {
            return backoff.getNextRetryDelayForUserError(errorRetries);
        } else if (status == Status.PLATFORM_FAILED) {
            return backoff.getNextRetryDelayForPlatformError(platformRetries);
        } else if (status == Status.TIMEOUT_FAILED) {
            return backoff.getNextRetryDelayForTimeoutError(timeoutRetries);
        }
        throw new MaestroInvalidStatusException("Invalid status for retry: " + status);
    }
}
```

重试流程：

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                            步骤重试流程                                       │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ┌─────────────┐     失败      ┌─────────────────────┐                     │
│   │   RUNNING   │──────────────▶│ USER_FAILED /       │                     │
│   │   (运行中)  │               │ PLATFORM_FAILED /   │                     │
│   └─────────────┘               │ TIMEOUT_FAILED      │                     │
│                                 └──────────┬──────────┘                     │
│                                            │                                 │
│                                            ▼                                 │
│                              ┌─────────────────────────────┐                │
│                              │    检查重试次数是否达到限制   │                │
│                              └──────────────┬──────────────┘                │
│                                             │                                │
│                          ┌──────────────────┴──────────────────┐            │
│                          │                                     │            │
│                          ▼ 未达到限制                           ▼ 已达到限制  │
│              ┌───────────────────────┐              ┌───────────────────┐   │
│              │   计算退避延迟时间     │              │   FATALLY_FAILED  │   │
│              │   (指数退避 + 随机)    │              │   或 TIMED_OUT    │   │
│              └───────────┬───────────┘              └───────────────────┘   │
│                          │                                                   │
│                          ▼                                                   │
│              ┌───────────────────────┐                                      │
│              │     等待退避时间       │                                      │
│              │  (FlowTaskRetry 调度) │                                      │
│              └───────────┬───────────┘                                      │
│                          │                                                   │
│                          ▼                                                   │
│              ┌───────────────────────┐                                      │
│              │    创建新的步骤尝试    │                                      │
│              │   (stepAttemptId++)   │                                      │
│              └───────────┬───────────┘                                      │
│                          │                                                   │
│                          ▼                                                   │
│              ┌───────────────────────┐                                      │
│              │       CREATED         │                                      │
│              │      (重新开始)       │                                      │
│              └───────────────────────┘                                      │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

## 12. API 接口概览

### 12.1 工作流 API

| 方法 | 路径 | 描述 |
|------|------|------|
| POST | `/api/v3/workflows` | 创建/更新工作流 |
| GET | `/api/v3/workflows/{id}/versions/latest` | 获取最新版本 |
| POST | `/api/v3/workflows/{id}/versions/latest/actions/start` | 启动工作流 |
| DELETE | `/api/v3/workflows/{id}` | 删除工作流 |

### 12.2 工作流实例 API

| 方法 | 路径 | 描述 |
|------|------|------|
| GET | `/api/v3/workflows/{id}/instances/{instanceId}/runs/{runId}` | 获取实例详情 |
| POST | `/api/v3/workflows/{id}/instances/{instanceId}/actions/stop` | 停止实例 |
| POST | `/api/v3/workflows/{id}/instances/{instanceId}/actions/restart` | 重启实例 |

### 12.3 步骤实例 API

| 方法 | 路径 | 描述 |
|------|------|------|
| GET | `.../steps/{stepId}/attempts/{attemptId}` | 获取步骤尝试详情 |
| POST | `.../steps/{stepId}/actions/restart` | 重启步骤 |
| POST | `.../steps/{stepId}/actions/skip` | 跳过步骤 |
| POST | `.../steps/{stepId}/actions/kill` | 终止步骤 |

### 12.4 信号 API

| 方法 | 路径 | 描述 |
|------|------|------|
| POST | `/api/v3/signals` | 发送信号 |
| GET | `/api/v3/signals/{name}/instances/{instanceId}` | 获取信号实例 |
| GET | `/api/v3/signals/{name}/instances/latest` | 获取最新信号实例 |

---

## 13. 高级主题

### 13.1 Foreach 循环步骤

Foreach 步骤支持并行循环执行，适用于批量数据处理：

```json
{
  "foreach": {
    "id": "batch-process",
    "loop_params": {
      "items": {
        "expression": "[1, 2, 3, 4, 5]",
        "type": "LONG_ARRAY"
      }
    },
    "concurrency": 3,
    "steps": [
      {"step": {"id": "process-item", "type": "NoOp"}}
    ]
  }
}
```

配置参数：

```yaml
stepruntime:
  foreach:
    loop-batch-limit: 50       # 循环批次大小
    insert-batch-limit: 10     # 插入批次大小
    get-rollup-batch-limit: 100 # 汇总获取批次大小
```

### 13.2 子工作流步骤

子工作流支持工作流复用和分层设计：

```json
{
  "subworkflow": {
    "id": "call-subwf",
    "workflow_id": "child-workflow-id",
    "version": "latest",
    "params": {
      "input_param": {
        "expression": "parent_param",
        "type": "STRING"
      }
    }
  }
}
```

配置参数：

```yaml
stepruntime:
  subworkflow:
    always-pass-down-param-names: []  # 总是传递的参数名列表
```

### 13.3 While 循环步骤

While 步骤支持条件循环，适用于轮询场景：

```json
{
  "while": {
    "id": "poll-until-ready",
    "condition": {
      "expression": "iteration_count < 10 && !data_ready",
      "type": "BOOLEAN"
    },
    "max_iterations": 100,
    "steps": [
      {"step": {"id": "check-status", "type": "Http"}}
    ]
  }
}
```

### 13.4 标签许可系统 (Tag Permits)

标签许可用于控制资源并发访问：

```yaml
stepruntime:
  enable-tag-permit: true
  tag-permit-task:
    batch-size: 512           # 批次大小
    clean-up-interval: 60000  # 清理间隔（毫秒）
    scan-interval: 30000      # 扫描间隔（毫秒）
```

使用示例：

```json
{
  "step": {
    "id": "resource-limited-step",
    "type": "Titus",
    "tags": [
      {"name": "gpu_cluster", "permit_count": 5}
    ]
  }
}
```

### 13.5 断点调试

Maestro 支持步骤级断点，用于调试复杂工作流：

```bash
# 创建断点
curl -X POST 'http://127.0.0.1:8080/api/v3/breakpoints' \
    -H "Content-Type: application/json" \
    -d '{
        "workflow_id": "my-workflow",
        "step_id": "step-to-debug"
    }'

# 恢复执行
curl -X POST '.../breakpoints/{id}/actions/resume'
```

---

## 14. 部署与运维

### 14.1 Docker 部署

```yaml
# docker-compose.yml
version: '3.8'
services:
  maestro:
    build: .
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=prod
      - JDBC_URL=jdbc:postgresql://db:5432/maestro
    depends_on:
      - db
  
  db:
    image: postgres:17
    environment:
      - POSTGRES_DB=maestro
      - POSTGRES_USER=maestro
      - POSTGRES_PASSWORD=maestro
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

### 14.2 Kubernetes 部署

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: maestro
spec:
  replicas: 3
  selector:
    matchLabels:
      app: maestro
  template:
    metadata:
      labels:
        app: maestro
    spec:
      containers:
      - name: maestro
        image: maestro:latest
        ports:
        - containerPort: 8080
        env:
        - name: JAVA_OPTS
          value: "-XX:+UseZGC -Xmx4g"
        - name: ENGINE_CONFIGS_FLOW_ENGINE_ADDRESS
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        resources:
          requests:
            memory: "2Gi"
            cpu: "1000m"
          limits:
            memory: "4Gi"
            cpu: "2000m"
        livenessProbe:
          httpGet:
            path: /actuator/health
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 5
```

### 14.3 生产环境配置建议

```yaml
# application-prod.yml
spring:
  application:
    name: maestro-prod

server:
  port: 8080

maestro:
  queue:
    properties:
      1:
        worker-num: 16      # 增加工作流启动处理器
        scan-interval: 3000
      2:
        worker-num: 24      # 增加实例更新处理器
        scan-interval: 3000
      3:
        worker-num: 32      # 步骤更新是最繁忙的队列
        scan-interval: 2000
  
  sel:
    thread-num: 8           # 增加 SEL 线程
    timeout-millis: 180000  # 增加超时
    loop-limit: 50001       # 增加循环限制

engine:
  configs:
    workflow-database-connection-pool-size-max: 32
    workflow-database-connection-pool-idle-min: 16
    internal-worker-num: 160  # 增加内部工作线程
    
logging:
  level:
    com.netflix: WARN  # 生产环境减少日志
```

### 14.4 监控告警

建议监控的关键指标：

| 指标 | 阈值建议 | 告警级别 |
|------|----------|----------|
| `num_of_running_flows` | > 10000 | Warning |
| `step_initialize_delay_p99` | > 30s | Critical |
| `queue_message_lag` | > 1000 | Warning |
| `workflow_failure_rate` | > 5% | Critical |
| `database_connection_pool_usage` | > 80% | Warning |

---

## 15. 故障排查

### 15.1 常见问题

#### 问题 1：工作流启动超时

**症状**：工作流创建后长时间处于 CREATED 状态

**排查步骤**：
1. 检查队列 1 (WORKFLOW_START_QUEUE) 的积压
2. 检查 Flow 执行器的分组所有权
3. 查看日志中的重试错误

**解决方案**：
```yaml
# 增加工作流启动处理器线程
maestro:
  queue:
    properties:
      1:
        worker-num: 24
```

#### 问题 2：步骤长时间等待信号

**症状**：步骤处于 WAITING_FOR_SIGNALS 状态

**排查步骤**：
1. 确认信号是否已发送
2. 检查信号匹配条件
3. 验证信号参数格式

**调试命令**：
```bash
# 查看信号实例
curl -X GET 'http://127.0.0.1:8080/api/v3/signals/{name}/instances/latest'

# 查看步骤信号依赖
curl -X GET '.../steps/{stepId}/attempts/latest'
```

#### 问题 3：分组所有权丢失

**症状**：JVM 因 INVALID_OWNERSHIP_EXIT_CODE 终止

**原因**：
- 数据库连接问题
- 心跳超时
- 时钟偏移

**预防措施**：
```yaml
engine:
  configs:
    maintenance-delay-millis: 5000  # 减少维护间隔
```

### 15.2 日志分析

关键日志模式：

```
# 工作流启动
INFO  - Run a workflow instance [wf-id][1][1] with an internal execution_id [xxx]

# 步骤状态变更
INFO  - Execute a step instance [wf-id][1][1][step.1] with status [RUNNING]

# 信号匹配
INFO  - Signal [signal-name] matched workflow [wf-id] trigger

# 错误重试
WARN  - Retry it as getting a runtime error
```

---

## 总结

Netflix Maestro 是一个设计精良的工作流编排系统，其核心优势包括：

### 架构亮点

| 特性 | 描述 |
|------|------|
| **虚拟线程** | 利用 Java 21 虚拟线程，大幅提升并发处理能力 |
| **分组管理** | 通过分组实现负载均衡和故障隔离 |
| **Actor 模型** | 简化并发控制，提高代码可维护性 |
| **状态机驱动** | 清晰的状态转换，便于理解和调试 |
| **至少一次语义** | 保证任务执行的可靠性 |

### 性能数据（Netflix 生产环境）

- 支持 **数十万** 并发工作流
- 每日处理 **数百万** 作业
- 保持严格的 **SLO** 即使在流量高峰

### 适用场景

1. **ETL 管道**：数据抽取、转换、加载
2. **机器学习工作流**：模型训练和推理管道
3. **A/B 测试管道**：实验数据处理
4. **数据迁移**：跨存储系统数据移动
5. **定时任务**：基于 Cron 的调度执行
6. **事件驱动流程**：基于信号的工作流触发

### 技术选型建议

| 场景 | 推荐 |
|------|------|
| 需要复杂 DAG 支持 | ✅ 推荐 |
| 需要 Foreach/While 循环 | ✅ 推荐 |
| 需要信号触发 | ✅ 推荐 |
| 需要高并发调度 | ✅ 推荐 |
| 简单定时任务 | ⚠️ 可能过重 |
| 实时流处理 | ❌ 不适合 |

---

## 参考资源

- [Maestro GitHub](https://github.com/Netflix/maestro)
- [Netflix Tech Blog - Maestro](https://netflixtechblog.com/maestro-netflixs-workflow-orchestrator-ee13a06f9c78)
- [100X Faster: How We Supercharged Maestro](https://netflixtechblog.com/100x-faster-how-we-supercharged-netflix-maestros-workflow-engine-028e9637f041)
- [Maestro Python SDK](https://github.com/jun-he/maestro-python)
- [Slack 社区](https://join.slack.com/t/maestro-oss/shared)

---

*文档版本：1.0*
*生成时间：2024年*
*基于 Maestro 源码深度分析*

