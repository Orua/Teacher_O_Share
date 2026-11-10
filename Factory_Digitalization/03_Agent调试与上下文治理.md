# Agent 调试与上下文治理：从表面错误追到真正发生的事

## 中文

本文把开发审计中的故障机制改写为通用案例。对象名称、事件顺序中的非必要细节和实现标识均已泛化。修复建议与已经取得的证据分开表述，不要求使用特定平台或语言。

## 1. 调试的起点是可复现的问题描述

“助手不能用”“模型很慢”都不足以定位问题。先记录具体任务、输入条件、期望结果、实际结果和时间，再确定失败发生在哪一层。

一条典型路径是：用户入口 → 执行服务 → 模型适配 → 上游模型 → 工具服务 → 业务结果。排查的目标，是找到最早偏离预期的位置。

尽量先保留失败样本，再做最小复现。一次同时换模型、网络、预算和服务版本，即使恢复成功，也难以解释原因。

## 2. 案例一：接口返回“找不到”，问题却在业务接口之前

### 观察到的现象

客户端调用一个工具入口，持续收到表示资源不存在的响应。直觉上可能怀疑接口地址、工具注册或协议兼容。

但服务端日志显示，请求在通用访问检查阶段已经被拒绝，根本没有进入工具处理逻辑。这时继续修改工具参数，没有机会改变结果。

### 怎样排除错误方向

沿请求入口检查处理顺序：访问规则在前，认证或业务处理在后。将同一时间的客户端响应与服务器拒绝日志对应起来，确认响应究竟由哪一层产生。

如果某地址来自共享转发路径，还要确认它代表单个访问者还是多个调用方。共享出口被作为单个恶意访客长期封禁，可能误伤其他正常请求。但是否属于共享出口，需要可信网络资料或实际转发证据，不能仅凭地址格式猜测。

### 配置已经修改，为什么仍然拒绝

随后出现一个新现象：持久配置中的限制项已经移除，服务仍持续拒绝。继续检查发现，配置只在第一次访问时载入内存，后续请求一直读取旧缓存。

这时可以区分三份状态：磁盘配置、进程内缓存、实际请求结果。只确认文件内容不够，刷新客户端也不会强迫服务重载配置。

短期处理应使用该服务支持的受控刷新方式；长期可以考虑版本检测、文件变化检测或明确的缓存失效机制。选择哪种方式取决于实际环境，不应把重启整台机器作为默认排障步骤。

### 如何确认结论

确认限制项已更新，确认运行进程已经读取新状态，再执行同一最小请求。访问层恢复后，仍需要保留后续认证与业务权限检查。修复误拦不等于放宽所有访问。

## 3. 案例二：模型返回了内容，任务为什么没有继续

另一类故障中，上游响应包含推理内容，却没有最终答案或工具调用。网络和上游请求都可能正常，但执行服务没有可继续处理的动作。

需要分别回答：

- 上游是否真的返回了可执行工具调用？
- 适配层是否正确解析了该调用？
- 结束事件是否与正文一致？
- 执行服务收到结果后是否继续了下一步？

只有定位到“响应缺少动作”，才有理由评估兼容重试。一个受控方案是在确认没有业务工具被执行的条件下，用适配当前模型能力的参数再请求一次；仍无有效动作则明确失败。具体参数和次数属于实现选择，不宜照搬。

若工具可能已经改变数据，重试前必须查清执行状态。补一次模型请求与重做一次业务写入，不是同一个风险。

验证时应覆盖普通回答、工具往返、流式结束和失败响应。最小问答成功，只能证明对应问答路径可用。

## 4. 案例三：长任务反复压缩，先看返回了什么

### 原先容易形成的判断

长任务频繁压缩，可能被解释为“每次对话都新建会话”“需要新的规划器”或“压缩阈值太低”。这些假设都应由证据验证，不能直接成为改造方案。

审计应先确认实际会话标识、恢复结果和新增输入。新进程不一定意味着新会话；数据库里保留完整原始请求，也不一定意味着宿主把它再次发送给模型。

### 从工具结果量找到具体异常

按工具、方法、参数、内容摘要和返回大小统计后，可以发现某些响应异常大。一个典型例子是图片被编码后塞进文本对象，而不是作为工具支持的图片内容返回。

模型需要图片，不代表需要大段编码文本。可考虑让图片使用被客户端支持的媒体内容，文本保留必要说明、标识和尺寸。这属于包装层优化，不能为了节省上下文直接删掉任务需要的视觉信息。

### 不能把所有重复读取都当成浪费

同一个工具名出现多次，可能是不同参数或新修订状态。同一测量方法也可能在检查不同对象。判断冗余至少需要比较方法、参数、版本和内容。

还有一类重复发生在压缩之后：工具说明或工作流不再完整保留，执行者为了恢复约束而重新读取。它与同一阶段无理由反复加载不同。贸然禁止补读，可能让下一步失去必要规则。

### 时间关系限制了因果结论

即使发现一次巨大的图片包装，如果此前已经发生压缩，就不能说它解释了全部压缩。较稳妥的结论是：它是一个可以单独修正的明确负担，其他成本仍需继续分析。

优化顺序因此应从明确异常开始，再减少说明中的重复表达，并保留必要恢复、最新状态读取和真实回执。没有证据时，不必先新增一套上下文管理系统。

## 5. 建立一张调试假设表

| 假设 | 需要的证据 | 什么结果会推翻它 |
|---|---|---|
| 请求没有到服务 | 入口日志与请求标识 | 服务已记录同一请求 |
| 业务接口不存在 | 路由或工具处理日志 | 请求在更早访问层被拒绝 |
| 配置修改未生效 | 持久配置与运行状态 | 进程已载入新配置但仍失败 |
| 会话每轮重建 | 实际会话标识与恢复回执 | 多轮保持同一会话且恢复成功 |
| 重复读取都是浪费 | 参数、修订和内容对比 | 状态已变化或压缩后需恢复 |
| 新代码已在运行 | 产物来源与运行身份 | 运行实例仍来自旧产物 |

AI 适合帮助整理这些证据、寻找对应代码和构造样本。人的作用是约束范围、提供业务事实，并要求结论停在证据能支持的位置。

## 6. 一次调试怎样收尾

交付说明写清现象、定位层、已排除的方向、实际修改和直接复现结果。只做了诊断，就保留为诊断结论；提出了修复方案但没安装，就不要说已恢复。

同时记录未解决的问题。例如一次响应包装修正，可以减少明确的上下文负担，却未必解决全部耗时。保留这个边界，下一轮才不会从错误前提继续开发。

---

# Agent Debugging and Context Management: Establish What Actually Happened

## English

These generalized cases preserve failure mechanisms from development reviews while removing organizational details and implementation identifiers. Evidence is distinguished from proposed remedies. No specific platform or language is required.

## 1. Begin with a reproducible problem

“Unavailable” or “slow” is not a useful diagnosis. Record the task, input conditions, expected and actual outcomes, and timing.

A typical path runs through the user entry point, execution service, model adapter, upstream model, tool service, and business result. Find the earliest departure from expected behavior. Preserve the failing sample before changing conditions. Changing the model, network, budget, and version together makes recovery difficult to explain.

## 2. Case one: a missing-resource response originates before the interface

### Observation

A tool request repeatedly receives a response suggesting that the resource does not exist. Initial suspects include the address, registration, and protocol.

Server logs reveal that a general access check rejected the request before tool handling. Changing tool arguments cannot affect a request that never reaches them.

### Eliminate the wrong layer

Inspect request-processing order and correlate client responses with server rejection records. Establish which component generated the response.

If an address belongs to shared forwarding infrastructure, determine whether it identifies one visitor or many callers. A long-lived block against a shared exit can affect unrelated requests. Establish that relationship using reliable network information or actual forwarding evidence rather than appearance alone.

### Why the configuration change did not help

The persistent block entry is removed, but rejection continues. Further inspection shows that configuration was loaded on first use and retained in memory.

Distinguish disk configuration, process cache, and actual request behavior. Reading the file is insufficient, and refreshing a client does not force the server to reload it.

Use the service's supported refresh mechanism for immediate recovery. Version checks, change detection, or explicit invalidation are possible longer-term designs. The choice depends on the environment; restarting the entire machine should not be a default diagnostic step.

### Confirm the result

Confirm the updated restriction, establish that the process loaded the new state, and rerun the same minimal request. Keep authentication and business authorization in place. Correcting a false block does not justify unrestricted access.

## 3. Case two: model output contains no actionable result

An upstream response may contain reasoning but no final answer or tool call. Network communication can succeed while the executor has no action to process.

Check whether an actionable call existed upstream, whether adaptation preserved it, whether ending events matched the content, and whether the executor continued.

Only after establishing missing action should compatibility retry be considered. One bounded design retries with parameters suited to the model after confirming that no business tool executed. Continued absence of a valid result should remain an explicit failure. Parameters and retry counts are implementation choices.

When a tool might already have changed data, establish its execution state first. Reissuing a model request and repeating a business write carry different risks.

Verify ordinary responses, tool round trips, stream endings, and failures. A successful minimal chat validates only that chat path.

## 4. Case three: repeated compaction in a long task

### Do not turn assumptions into architecture

Frequent compaction can lead to claims that each turn starts a new session, a planner is missing, or thresholds are too low. Check actual session identity, resume results, and incremental input first.

A new process does not necessarily mean a new conversation. A stored full prompt does not establish that the host resent it to the model.

### Inspect large results

Group tool output by method, arguments, content, and size. An unusually large response may contain encoded image data embedded in text rather than supported image content.

The model may need the image without needing a long encoded text representation. Use media content supported by the client and retain necessary descriptions, identifiers, and dimensions. This is a packaging improvement, not a reason to remove required visual evidence.

### Repeated reads can be necessary

Repeated tool names can represent different parameters or new revisions. Compare method, arguments, version, and content before calling a read redundant.

Some documentation is reread after compaction because the executor needs to restore full constraints. That differs from unexplained repetition within the same phase. Blocking recovery reads can remove information required for correct execution.

### Respect chronology

A large image response cannot explain compactions that occurred before it. It can be a clear, independently fixable burden while other costs remain unresolved.

Start with demonstrated anomalies, reduce duplicated explanations, and preserve recovery, current-state reads, and actual receipts. Evidence may not justify adding a new context-management subsystem.

## 5. Maintain a hypothesis table

| Hypothesis | Evidence needed | Evidence against it |
|---|---|---|
| The request never reached the service | Entry logs and request identity | The same request is recorded |
| The interface is absent | Routing or handling evidence | An earlier access check rejected it |
| Configuration did not take effect | Persistent and runtime states | New state loaded but failure remains |
| Every turn creates a new session | Session identity and resume receipts | Successful resume under the same identity |
| Every reread is waste | Arguments, revisions, and content | Changed state or necessary recovery |
| New code is active | Artifact provenance and runtime identity | Runtime still uses an older artifact |

AI can organize evidence, locate relevant logic, and construct samples. People constrain the investigation, supply business facts, and require conclusions to stay within the evidence.

## 6. Close the investigation precisely

Report the symptom, failure layer, rejected hypotheses, actual change, and direct reproduction result. A diagnosis remains a diagnosis until a remedy is applied and checked.

State remaining uncertainty. Fixing a response envelope may reduce a specific context burden without explaining all latency. Preserving that boundary prevents later work from building on a false premise.
