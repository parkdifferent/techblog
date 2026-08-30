# AI Agent 必读技术资料指南

> 一份面向工程师、技术负责人和研究者的精选阅读清单  
> 更新日期：2026-08-30  
> 范围：LLM Agent 的架构、推理与工具、上下文与记忆、多智能体、Coding Agent、评测、安全、互操作协议和生产化

## 先说结论

AI Agent 领域更新很快，但真正经得住时间检验的内容并不多。阅读时应把三类材料分开：

1. **奠基论文**：定义了后来反复出现的架构和问题，例如 ReAct、Reflexion、SWE-bench。
2. **生产工程复盘**：来自真实产品和大规模系统，重点看失败模式、成本、评测与运维，而不是产品功能。
3. **框架和协议文档**：时效性强，适合动手时查阅，不宜替代系统原理。

最重要的工程判断不是“该选哪个 Agent 框架”，而是：

- 任务是否真的需要 Agent，而不是一次模型调用或确定性工作流？
- 任务能否被验证，成功条件能否自动或半自动检查？
- 工具接口是否为模型设计过，权限和副作用是否受控？
- 上下文如何选择、压缩、隔离和持久化？
- 错误在长轨迹、多 Agent 和外部工具之间会怎样放大？

## 评级和筛选方法

### 评级

- **S 必读**：形成了领域共识、开创了重要范式、成为事实标准，或提供了难以替代的生产证据。
- **A 强烈推荐**：在某个关键子领域很有工程价值，内容扎实且可迁移。
- **B 按需阅读**：优秀但更偏特定框架、产品、场景，或主要具有历史价值。

评级综合考虑：来源权威性、原创贡献、业界采用、学术影响、可复现性、写作质量和截至本文更新日的时效性。引用数随数据库和时间变化很大，本文不展示容易过时的精确数字，而用论文发表场所、后续采用和基准地位作定性判断。

### 来源优先级

1. 论文原文、会议论文集、官方规范
2. 研究机构或产品团队的一手工程复盘
3. 官方技术教程与开源实现
4. 高质量独立综述

厂商文章会有产品立场。本文保留其可迁移的工程知识，并在“注意”栏标明局限，不把营销主张等同于独立证据。

## 如果只读 15 篇

按以下顺序读，可以先建立完整心智模型，再进入生产工程：

| 顺序 | 资源 | 为什么值得读 |
|---:|---|---|
| 1 | [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) | 当前最清晰的 Agent 与 Workflow 边界，以及路由、并行、编排者、评审优化等基本模式。 |
| 2 | [LLM Powered Autonomous Agents](https://lilianweng.github.io/posts/2023-06-23-agent/) | 用规划、记忆、工具三部分建立经典心智模型，适合快速补齐概念地图。 |
| 3 | [ReAct](https://arxiv.org/abs/2210.03629) | “推理、行动、观察”循环的代表性论文，影响了大量 Agent 实现。 |
| 4 | [Toolformer](https://arxiv.org/abs/2302.04761) | 理解模型如何学习何时调用工具、传什么参数、怎样利用结果。 |
| 5 | [Reflexion](https://arxiv.org/abs/2303.11366) | 用语言反馈和情景记忆改进后续尝试，是反思式 Agent 的代表作。 |
| 6 | [Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) | 把 Agent 的瓶颈从“写提示词”提升为有限上下文资源的系统管理。 |
| 7 | [MemGPT](https://arxiv.org/abs/2310.08560) | 用操作系统和分层存储视角理解长期记忆、工作记忆与上下文换入换出。 |
| 8 | [Towards a science of scaling agent systems](https://research.google/blog/towards-a-science-of-scaling-agent-systems-when-and-why-agent-systems-work/) | 用受控实验回答“什么时候多 Agent 有用”，反驳“越多越好”。 |
| 9 | [How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system) | 少见的生产级多 Agent 复盘，覆盖委派、并发、成本、评测和部署。 |
| 10 | [SWE-bench](https://arxiv.org/abs/2310.06770) | Coding Agent 的事实标准之一，把评测推进到真实仓库和真实 Issue。 |
| 11 | [SWE-agent](https://arxiv.org/abs/2405.15793) | 说明 Agent-Computer Interface 和 Harness 设计本身能显著影响能力。 |
| 12 | [Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) | 目前最实用的 Agent 评测入门之一，兼顾轨迹、结果、评分器和人工复核。 |
| 13 | [τ-bench](https://arxiv.org/abs/2406.12045) | 测量工具调用、用户交互、业务规则和多次运行一致性，比单轮正确率更接近生产。 |
| 14 | [AgentDojo](https://arxiv.org/abs/2406.13352) | 把间接提示注入放进真实工具环境，是 Agent 安全评测的核心资料。 |
| 15 | [Model Context Protocol Specification](https://modelcontextprotocol.io/specification/) | 理解 Agent 与数据、工具互操作的主流开放协议，阅读当前版本而非旧教程。 |

## 一、定义、架构与设计模式

| 评级 | 资源 | 来源 | 重点与阅读建议 |
|---|---|---|---|
| S | [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) | Anthropic Engineering, 2024 | 从 augmented LLM 出发，逐步讲解 prompt chaining、routing、parallelization、orchestrator-workers、evaluator-optimizer 和自主 Agent。先读“何时不该用 Agent”。 |
| A | [Building agents](https://developers.openai.com/tracks/building-agents) | OpenAI Developers | 当前官方学习路径，将模型、工具、知识、逻辑、Agent SDK、可观测性和评测连成完整工程流程。适合边学边做。 |
| A | [Agents guide](https://developers.openai.com/api/docs/guides/agents) | OpenAI Developers | 当前 OpenAI Agent 构建入口，适合实现时查阅。注意它是平台文档，不是独立框架比较。 |
| S | [LLM Powered Autonomous Agents](https://lilianweng.github.io/posts/2023-06-23-agent/) | Lilian Weng, 2023 | 经典独立综述，规划、记忆、工具的三分法非常有影响力。部分项目和 API 已过时，但心智模型仍有价值。 |
| A | [A Survey on Large Language Model based Autonomous Agents](https://arxiv.org/abs/2308.11432) | 学术综述 | 体系化梳理 Agent 的构建、应用和评估，适合需要论文脉络和参考文献树的读者。 |
| A | [How to Build an Agent](https://www.langchain.com/blog/how-to-build-an-agent) | LangChain, 2025 | 从具体样例、SOP、最小提示词、连接工具、测试到上线的务实路线。框架相关表述要与通用原则分开看。 |
| A | [A developer's guide to production-ready AI agents](https://cloud.google.com/blog/products/ai-machine-learning/a-devs-guide-to-production-ready-ai-agents) | Google Cloud, 2026 | 覆盖 Agent 生命周期、上下文、记忆、工具、评测、部署与治理，并链接一组更细的实战指南。 |

### 本节应带走的结论

- 固定步骤且可明确编码的任务优先用 Workflow。
- 只有在路径无法预先写死、需要根据环境反馈动态调整时，才值得引入更强自治。
- 从单 Agent 和少量工具开始，只有评测证明收益后再增加层级、多 Agent 或复杂框架。
- Agent 的核心不是“会聊天”，而是模型、环境、工具、状态、控制循环和验证器共同组成的系统。

## 二、推理、规划、工具调用与自我改进

| 评级 | 资源 | 贡献 | 注意 |
|---|---|---|---|
| S | [ReAct: Synergizing Reasoning and Acting in Language Models](https://arxiv.org/abs/2210.03629) | 将语言推理轨迹和环境行动交错执行，奠定大量工具型 Agent 的循环范式。 | 显式思维轨迹不是生产系统的必需接口，真正关键的是可迭代的行动与反馈。 |
| S | [Toolformer: Language Models Can Teach Themselves to Use Tools](https://arxiv.org/abs/2302.04761) | 研究模型自监督学习插入 API 调用，回答“何时用、用哪个、参数是什么、如何吸收结果”。 | 更偏模型训练研究，与今天的结构化 tool calling API 不完全相同。 |
| A | [MRKL Systems](https://arxiv.org/abs/2205.00445) | 早期提出语言模型路由到神经或符号专家模块，影响后来工具路由和模块化 Agent。 | 主要是架构思想和早期实验。 |
| A | [Tree of Thoughts](https://arxiv.org/abs/2305.10601) | 将推理扩展为候选状态的搜索、评估与回溯，适合理解搜索式规划。 | 额外采样和评分会显著增加延迟、成本和评测复杂度。 |
| S | [Reflexion](https://arxiv.org/abs/2303.11366) | 不更新权重，而用语言反馈和 episodic memory 改善下一次尝试。 | 反思也可能生成错误归因，必须由环境反馈或验证器约束。 |
| A | [Self-Refine](https://arxiv.org/abs/2303.17651) | 用同一模型生成、批评和改写输出，形成通用迭代优化模式。 | 自我评分不等于真实正确性，外部检查通常更可靠。 |
| A | [Writing effective tools for AI agents](https://www.anthropic.com/engineering/writing-tools-for-agents) | 从工具原型、评测、Agent 辅助优化到命名空间和返回内容，讲清 Agent-Computer Interface。 | 一手厂商经验，原则可迁移，具体示例以 Claude 工具接口为主。 |
| A | [Introducing smolagents](https://huggingface.co/blog/smolagents) | 用极简开源实现展示 tool-calling Agent 与 code Agent，并讨论何时适合使用 Agent。 | 适合动手理解，不代表 code action 在所有安全边界内都优于结构化调用。 |

## 三、上下文工程、记忆与长任务

| 评级 | 资源 | 重点 | 适合谁 |
|---|---|---|---|
| S | [Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) | 把上下文视为有限资源，讨论选择、压缩、持久化和隔离。 | 所有生产 Agent 工程师。 |
| A | [Context Engineering](https://www.langchain.com/blog/context-engineering-for-agents) | 用 write、select、compress、isolate 四类操作总结上下文工程。 | 希望获得可执行分类法的读者。 |
| S | [MemGPT: Towards LLMs as Operating Systems](https://arxiv.org/abs/2310.08560) | 分层记忆与虚拟上下文管理，启发了后来的长期记忆 Agent。 | 研究记忆架构和长期对话者。 |
| S | [Generative Agents](https://arxiv.org/abs/2304.03442) | memory stream、检索、反思和规划组成可交互模拟角色。 | 研究长期行为、社会模拟和个性化 Agent 者。 |
| A | [Voyager](https://arxiv.org/abs/2305.16291) | 自动课程、可执行技能库、环境反馈和自验证，展示开放式终身学习 Agent。 | 研究技能积累、具身环境和课程学习者。 |
| A | [Dynamic context discovery](https://cursor.com/blog/dynamic-context-discovery) | 将长工具结果、历史、Skills、MCP 工具和终端历史变成可动态发现的文件化上下文。 | Coding Agent 和 Harness 设计者。 |
| S | [Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) | 用初始化器、功能清单、进度文件、Git 历史、增量交付和端到端测试跨越多个上下文窗口。 | 长任务、代码生成和持续执行系统。 |

### 上下文工程检查表

- 不要把“能塞进窗口”误当成“应该塞进窗口”。
- 静态规则只放稳定且高频的信息，其余内容按需检索。
- 工具原始输出先过滤、结构化或落盘，避免吞噬上下文。
- 对压缩后的上下文保留任务目标、关键决策、未完成项、证据和可恢复状态。
- 长任务使用外部状态、检查点和幂等动作，不能只依赖对话历史。
- 记忆写入和记忆召回都要评测，错误记忆比没有记忆更危险。

## 四、多智能体系统

| 评级 | 资源 | 关键价值 | 局限与注意 |
|---|---|---|---|
| S | [Towards a science of scaling agent systems](https://research.google/blog/towards-a-science-of-scaling-agent-systems-when-and-why-agent-systems-work/) | 基于 180 种配置研究不同拓扑，显示多 Agent 对可并行任务有利，对强顺序依赖任务可能有害。 | 结果依赖测试任务和模型，但比“多 Agent 一定更强”的经验判断可靠。 |
| A | [How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system) | 真实研究产品中的 orchestrator-worker、上下文隔离、并行搜索、委派提示、评测和部署经验。 | 内部评测不可完全独立复现，且报告了显著 token 成本。 |
| S | [AutoGen](https://www.microsoft.com/en-us/research/publication/autogen-enabling-next-gen-llm-applications-via-multi-agent-conversation-framework/) | 对话式可组合多 Agent 框架的代表论文，COLM 2024，并获 ICLR 2024 LLM Agents Workshop Best Paper。 | 论文影响大，但实际选型要使用当前 Microsoft Agent Framework 文档，不应照搬旧 API。 |
| A | [Magentic-One](https://arxiv.org/abs/2411.04468) | 以 Orchestrator 动态规划、委派、追踪和重规划，组合网页、文件、终端和代码 Agent。 | 是具体系统设计，不是“分工越细越好”的证明。 |
| A | [How and when to build multi-agent systems](https://www.langchain.com/blog/how-and-when-to-build-multi-agent-systems) | 从上下文管理、并行化和团队边界判断是否值得拆成多 Agent。 | 带有 LangGraph 视角，重点读决策逻辑。 |
| A | [Benchmarking Multi-Agent Architectures](https://www.langchain.com/blog/benchmarking-multi-agent-architectures) | 用具体实验比较 supervisor、swarm 等常见结构，强调架构与任务特征的匹配。 | 厂商实验范围有限，不能泛化为所有模型和任务的排名。 |
| A | [CAMEL](https://arxiv.org/abs/2303.17760) | 早期探索 role-playing 和通信式 Agent 社会，为大量角色型多 Agent 工作提供起点。 | 偏研究探索，现代生产系统更需要权限、状态和评测。 |
| B | [MetaGPT](https://arxiv.org/abs/2308.00352) | 将 SOP 和软件团队角色编码进多 Agent 工作流，业界传播广。 | 适合研究结构化协作，不应把角色扮演本身视为可靠性来源。 |

### 什么时候使用多 Agent

优先考虑多 Agent：

- 子任务可独立并行，且最终能汇总或验证。
- 单个上下文窗口无法容纳全部工作材料。
- 不同子任务确实需要不同工具、权限或专门指令。
- 任务价值足以覆盖更多 token、延迟和协调成本。

优先保持单 Agent 或确定性工作流：

- 任务有强顺序依赖，前一步错误会污染所有后续步骤。
- 各 Agent 必须共享大量相同上下文。
- 无法自动判断谁的结果正确。
- 多 Agent 只是为了模拟职位名称，没有形成真正的隔离、并行或互补能力。

## 五、Coding Agent 与 Agent Harness

| 评级 | 资源 | 重点 |
|---|---|---|
| S | [SWE-bench](https://arxiv.org/abs/2310.06770) | 用真实 GitHub Issue、仓库和测试评估代码修复，改变了 Coding Agent 的评测方式。阅读时也要理解数据污染、任务有效性和 Harness 差异。 |
| S | [SWE-agent: Agent-Computer Interfaces Enable Automated Software Engineering](https://arxiv.org/abs/2405.15793) | 核心洞见是为 Agent 设计命令和反馈接口，与提升基础模型同样重要。 |
| A | [OpenHands](https://arxiv.org/abs/2407.16741) | 开源通用软件开发 Agent 平台，整合终端、代码编辑、浏览器、沙箱、多 Agent 与基准。适合研究系统实现。 |
| S | [Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) | 目前最可操作的跨上下文 Coding Agent 长任务实践之一。 |
| A | [Continually improving our agent harness](https://cursor.com/blog/continually-improving-agent-harness) | 介绍 Harness 如何针对不同模型调整、用线上与离线信号评估、从静态上下文转向动态发现。 |
| A | [Dynamic context discovery](https://cursor.com/blog/dynamic-context-discovery) | 具体展示文件、工具、聊天历史和终端输出如何按需进入上下文。 |
| A | [Expanding our long-running agents research preview](https://cursor.com/blog/long-running-agents) | 长任务中的先计划后执行、多个 Agent 交叉检查、规模与合并率等实践观察。数据来自自家产品，宜作为案例而非普遍结论。 |
| A | [Best practices for coding with agents](https://cursor.com/blog/agent-best-practices) | Rules、Skills、Hooks、规划和长循环的实用用法，适合 Cursor 用户直接落地。 |
| B | [Composer 2 Technical Report](https://cursor.com/resources/Composer2.pdf) | 训练和部署使用一致 Harness、长轨迹强化学习和真实软件任务的技术报告。更偏模型与产品体系。 |

### Coding Agent 阅读时应观察什么

- Agent 能获得哪些仓库结构、符号、测试、日志和运行时反馈？
- 编辑、搜索、执行命令的接口是否紧凑、可恢复、对模型友好？
- 测试是否真的覆盖用户可见行为，而不只是模型自己声称“已完成”？
- 任务是否使用隔离沙箱，网络、密钥、依赖安装和外部写操作如何授权？
- 基准分数来自同一模型还是不同 Harness？成本、token、超时和重试是否一致？

## 六、评测、基准与可观测性

Agent 的输出不是一段文本，而是一条可能改变环境状态的轨迹。至少要分别评测：最终任务成功、过程约束、工具调用、成本与延迟、稳定性、安全性和恢复能力。

### 评测方法

| 评级 | 资源 | 价值 |
|---|---|---|
| S | [Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) | 从任务定义、trial、transcript、outcome、grader 到 eval harness，提供生产导向的完整方法。 |
| A | [OpenAI Evals guide](https://developers.openai.com/api/docs/guides/evals) | 官方评测流程和 API 入口，适合构建数据集、评分器和持续评测。 |
| A | [A developer's guide to production-ready AI agents](https://cloud.google.com/blog/products/ai-machine-learning/a-devs-guide-to-production-ready-ai-agents) | 将 trajectory evaluation、部署阶段、日志和治理纳入 Agent 生命周期。 |

### 代表性基准

| 评级 | 基准 | 测量什么 | 使用提醒 |
|---|---|---|---|
| S | [GAIA](https://arxiv.org/abs/2311.12983) | 通用助手的推理、网页搜索、多模态和工具使用。 | 适合开放式研究 Agent，但静态题集需关注污染和答案维护。 |
| A | [AgentBench](https://arxiv.org/abs/2308.03688) | 在八类交互环境中评估长期推理、决策和指令遵循。 | 奠基性强，部分环境和模型已显年代感。 |
| S | [WebArena](https://arxiv.org/abs/2307.13854) | 在可复现真实网站中完成长程网页任务。 | 比静态问答更真实，但浏览器实现和视觉能力会显著影响成绩。 |
| S | [OSWorld](https://arxiv.org/abs/2404.07972) | 在真实桌面操作系统中完成跨应用计算机任务。 | 基础设施复杂，要严格记录环境版本和执行条件。 |
| S | [SWE-bench](https://arxiv.org/abs/2310.06770) | 真实代码仓库问题修复。 | 不要只看榜单。任务清洗、测试质量、模型污染、成本和 Harness 都会影响结果。 |
| S | [τ-bench](https://arxiv.org/abs/2406.12045) | 工具、Agent、模拟用户的多轮交互和业务政策遵循。 | `pass^k` 强调多次执行可靠性，通常比最好一次的 pass@1 更接近生产。 |
| A | [ToolSandbox](https://arxiv.org/abs/2408.04682) | 有状态工具使用、隐式状态依赖和用户交互。 | 适合评估 function calling 系统，不覆盖全部开放环境风险。 |
| S | [AgentDojo](https://arxiv.org/abs/2406.13352) | 正常任务效用与间接提示注入攻击下的安全性。 | 安全和任务能力必须一起看，防住攻击但无法完成任务同样不可用。 |

### 建立自有 Eval 的最小闭环

1. 从 20 至 50 个真实任务起步，覆盖常见请求、高风险动作和已知失败案例。
2. 记录完整轨迹：模型版本、系统指令、工具定义、调用参数、观察结果、状态变化、token、延迟和重试。
3. 最终状态优先使用确定性检查，例如数据库状态、单元测试、文件差异和 API 结果。
4. 对开放式质量使用带 rubric 的 LLM judge，但先用人工样本校准，并定期复核漂移。
5. 同时测单次成功率和重复运行一致性。Agent 的方差通常比单轮生成更重要。
6. 每次修改模型、提示、工具描述、检索、Harness 或权限策略，都运行回归评测。
7. 线上失败和人工纠正持续回流为新测试，不允许评测集长期停留在演示样例。

## 七、工具协议与互操作

| 评级 | 资源 | 应理解的边界 |
|---|---|---|
| S | [Model Context Protocol Specification](https://modelcontextprotocol.io/specification/) | MCP 标准化 Agent 应用与资源、提示和工具之间的连接。规范更新快，应使用入口页中的当前版本。它解决连接协议，不自动解决工具质量、授权和提示注入。 |
| A | [Agent2Agent Protocol Specification](https://a2a-protocol.org/latest/specification/) | A2A 面向独立 Agent 之间的发现、任务协作和内容交换。它与 MCP 的 Agent-to-tool 关注点不同，可组合而非互相替代。 |
| A | [Writing effective tools for AI agents](https://www.anthropic.com/engineering/writing-tools-for-agents) | 协议连通以后，工具描述、参数、命名空间、返回内容和错误反馈仍决定 Agent 是否会正确使用。 |
| A | [Introducing smolagents](https://huggingface.co/blog/smolagents) | 通过小型实现理解结构化工具调用和代码执行式 Agent 的差异。 |
| A | [Open-source DeepResearch](https://huggingface.co/blog/open-deep-research) | 开放实现展示 CodeAgent、网页检索、GAIA 评测和社区复现如何结合。 |

## 八、安全、权限与治理

Agent 的风险来自“模型输出被接到有副作用的软件接口”。仅靠系统提示或让模型自我审查，不足以形成安全边界。

| 评级 | 资源 | 核心价值 |
|---|---|---|
| S | [OWASP Top 10 for Agentic Applications 2026](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/) | 面向 Agent 应用的威胁分类和缓解起点，适合架构评审和安全检查表。 |
| S | [AgentDojo](https://arxiv.org/abs/2406.13352) | 在邮件、银行、旅行等工具环境中同时测正常任务和提示注入，提供可扩展安全评测。 |
| S | [Defeating Prompt Injections by Design](https://arxiv.org/abs/2503.18813) | CaMeL 将模型视为不可信组件，通过控制流、数据流、来源追踪和 capability 限制危险动作。 | 
| A | [Lessons from Defending Gemini Against Indirect Prompt Injections](https://storage.googleapis.com/deepmind-media/Security%20and%20Privacy/Gemini_Security_Paper.pdf) | Google DeepMind 的间接提示注入评测与多层防御经验，强调系统级防护。 |
| A | [NIST: Summary Analysis of Responses on AI Agent Security](https://www.nist.gov/publications/summary-analysis-responses-request-information-regarding-security-considerations-ai) | 汇总 2026 年政府、产业和研究界对 Agent 威胁、评估、身份、授权与标准的意见。它是现状分析，不是完整控制标准。 |
| A | [NIST AI Agent Standards Initiative](https://www.nist.gov/news-events/news/2026/02/announcing-ai-agent-standards-initiative-interoperable-and-secure) | 跟踪 Agent 互操作、安全、身份和授权标准工作的官方入口。 |

### 最低安全基线

- 默认最小权限，读与写分离，高风险工具单独授权。
- 不可信网页、邮件、文档和工具结果全部视为数据，不能获得与系统指令同等的控制权。
- 转账、发送消息、删除、部署、权限变更等不可逆或外部动作需要策略检查和适当的人类确认。
- 使用沙箱、网络出口限制、短期凭证、资源配额和超时，限制失控 Agent 的影响范围。
- 工具端独立校验身份、授权、参数和业务规则，不能信任模型“已经检查过”。
- 记录可审计的动作日志，但避免把密钥、隐私数据和隐藏提示完整写入日志。
- 将提示注入、工具投毒、数据外泄、越权、循环失控、记忆污染和多 Agent 信任传播纳入红队测试。

## 九、四周阅读与实践路线

### 第 1 周：建立心智模型

1. Anthropic《Building effective agents》
2. Lilian Weng《LLM Powered Autonomous Agents》
3. ReAct
4. Toolformer
5. 实践：不用框架，实现一个只有 2 个只读工具、最多 6 步、带停止条件的 Agent loop。

### 第 2 周：上下文、记忆和工具

1. Anthropic《Effective context engineering》
2. LangChain《Context Engineering》
3. MemGPT
4. Anthropic《Writing effective tools》
5. 实践：为长工具结果加入落盘、摘要、按需检索；比较完整上下文与动态上下文的正确率、成本和延迟。

### 第 3 周：多 Agent 与 Coding Agent

1. Google《Towards a science of scaling agent systems》
2. Anthropic 多 Agent Research 复盘
3. SWE-bench 与 SWE-agent
4. Anthropic 长任务 Harness 或 Cursor Harness 文章
5. 实践：只对可并行子任务增加 worker，并与单 Agent 做同模型、同预算对照实验。

### 第 4 周：评测、安全和生产化

1. Anthropic《Demystifying evals for AI agents》
2. τ-bench 与 AgentDojo
3. OWASP Agentic Top 10
4. MCP 与 A2A 规范概览
5. 实践：为前面的小 Agent 建立 30 个任务的回归集、5 个提示注入案例、权限矩阵、轨迹日志和人工审批点。

## 十、按目标选择阅读组合

| 目标 | 最小阅读组合 |
|---|---|
| 第一次构建 Agent | Building effective agents → ReAct → How to Build an Agent → OpenAI Building agents |
| 做企业流程 Agent | τ-bench → Demystifying evals → OWASP Top 10 → MCP Spec → Google production guide |
| 做 Deep Research | Anthropic multi-agent research → Open DeepResearch → GAIA → Google agent scaling |
| 做 Coding Agent | SWE-bench → SWE-agent → Anthropic long-running harness → Cursor dynamic context → AgentDojo |
| 做长期记忆与个性化 | MemGPT → Generative Agents → Context Engineering → Reflexion |
| 做多 Agent 编排 | Google agent scaling → Anthropic multi-agent research → AutoGen → Magentic-One |
| 做 Agent 平台或基础设施 | OpenHands → MCP → A2A → OpenAI Evals → NIST/OWASP 安全资料 |

## 十一、阅读论文和厂商博客的检查模板

每读一篇，至少回答以下问题：

1. **问题**：它解决的是模型能力、Agent 架构、工具接口、上下文、评测，还是基础设施问题？
2. **基线**：是否与更简单的 prompt、Workflow、单 Agent 和同预算方案比较？
3. **变量**：模型、Harness、工具、token 预算、重试次数是否被同时改变？
4. **指标**：测的是最终成功、最好一次表现、平均表现、稳定性、成本，还是主观偏好？
5. **验证**：结果由确定性规则、环境状态、人工专家还是 LLM judge 判定？
6. **失败模式**：作者是否报告失败案例、方差、权限风险和无法泛化之处？
7. **可复现性**：是否提供代码、数据、提示、工具定义、环境版本和完整轨迹？
8. **迁移性**：结论依赖特定模型或产品，还是能迁移到其他技术栈？
9. **成本**：提升是否只是来自更多 token、更多采样、更多 Agent 或更长运行时间？
10. **时效性**：论文的具体数值可能很快过时，但它提出的任务、接口或评测方法是否仍有效？

## 十二、哪些热门内容没有列为必读

- **AutoGPT、BabyAGI 等早期爆款项目**：历史意义大，但可靠性、评测和生产工程不足。可用于了解 2023 年 Agent 热潮，不建议作为当前架构模板。
- **只展示 Demo 的产品发布**：不能回答失败率、稳定性、成本、权限和安全问题。
- **只看排行榜的模型对比**：Agent 分数高度依赖 Harness、工具、重试、预算和环境版本。
- **没有简单基线的多 Agent 论文或博客**：如果没有与单 Agent、Workflow 和同 token 预算比较，很难判断收益来自架构还是更多计算。
- **只讲 Prompt、不讲环境和验证的教程**：Agent 可靠性主要是系统工程问题，不是靠一段万能提示词解决。
- **过时的框架 API 教程**：实现细节应查当前官方文档，本文优先保留更持久的原理和一手复盘。

## 十三、维护建议

建议每季度更新一次，并执行以下规则：

1. 检查 OpenAI、Anthropic、Google、Microsoft、Cursor、LangChain、Hugging Face、NIST、OWASP、MCP 和 A2A 的官方更新。
2. 对框架教程标记版本和日期；失效 API 移入历史区，不直接删除其仍有价值的设计思想。
3. 新增论文前，要求至少满足一项：顶会或权威机构、显著引用和后续工作、被主流系统采用、公开可复现、提出重要新基准。
4. 新增厂商文章前，要求包含架构细节、失败经验、评测或可迁移的工程方法，纯发布公告不收录。
5. 每次更新抽查所有链接，并记录更新时间。

## 附录：机构入口

- [OpenAI Developers](https://developers.openai.com/)
- [Anthropic Engineering](https://www.anthropic.com/engineering)
- [Google Research Blog](https://research.google/blog/)
- [Google Cloud AI & Machine Learning Blog](https://cloud.google.com/blog/products/ai-machine-learning)
- [Microsoft Research AutoGen Publications](https://www.microsoft.com/en-us/research/project/autogen/publications/)
- [Cursor Blog](https://cursor.com/blog)
- [LangChain Blog](https://www.langchain.com/blog)
- [Hugging Face Blog](https://huggingface.co/blog)
- [Hugging Face Agents Course](https://huggingface.co/learn/agents-course)
- [Model Context Protocol](https://modelcontextprotocol.io/)
- [A2A Protocol](https://a2a-protocol.org/latest/)
- [OWASP GenAI Security Project](https://genai.owasp.org/)
- [NIST AI](https://www.nist.gov/artificial-intelligence)
- [arXiv Artificial Intelligence](https://arxiv.org/list/cs.AI/recent)
- [OpenReview](https://openreview.net/)

---

这是一份“少而精、可执行”的技术阅读地图，不是 Agent 资源的百科全书。若一个新资源不能帮助读者更好地设计、验证、约束或运行 Agent，就不应仅因热度被加入主清单。
