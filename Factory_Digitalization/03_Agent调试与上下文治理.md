# Agent 调试与上下文治理：从表面错误追到真正发生的事

## 中文

本文根据访问拦截排查、模型桥接修复、会话审计和上下文优化的历史执行记录修订。业务身份与内部标识已移除，保留影响因果判断的先后顺序。数字是指定样本的观测，不是平台性能承诺；诊断、候选实现与实际发布分别表述。

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

在这次排查中，用户从持久黑名单中移除地址后，报告仍被拦截。源码显示：读取方法先检查静态集合是否非空，非空就立即返回；只有首次访问才读文件。文件时间变化没有参与失效判断。因此“文件里已删掉”与“进程仍拒绝”可以同时成立。

这时可以区分三份状态：磁盘配置、进程内缓存、实际请求结果。只确认文件内容不够，刷新客户端也不会强迫服务重载配置。

短期处理应使用该服务支持的受控刷新方式；长期可以考虑版本检测、文件变化检测或明确的缓存失效机制。选择哪种方式取决于实际环境，不应把重启整台机器作为默认排障步骤。

### 如何确认结论

确认限制项已更新，确认运行进程已经读取新状态，再执行同一最小请求。访问层恢复后，仍需要保留后续认证与业务权限检查。修复误拦不等于放宽所有访问。

### 两个阶段的证据不能合并成一次成功修复

早期会话确认的是“访问检查先返回 404”和“静态缓存不随文件更新”；记录提出回收对应应用池，但没有留下同轮恢复成功的请求回执。不能把这个建议改写为已经实施的热重载。

后续同类故障的处置才有发布和对照请求证据：保留地址黑名单，只对精确匹配的 MCP 路径增加认证准入，要求安全传输、允许的方法、有效凭据、有效账号及已有 MCP 权限；异常或认证助手不可用时仍拒绝。通过准入后，业务授权继续执行。选择它而不是整段出口白名单，是因为共享出口不等于可信用户。

| 后续部署后的请求条件 | 实际结果 | 能说明什么 |
|---|---|---|
| 被拦地址条件、有效凭据、初始化请求 | 200 | 指定认证路径可到达 MCP |
| 同地址条件、无效凭据 | 404 | 不是无条件解除黑名单 |
| 同地址条件、有效凭据、扫描路径 | 404 | 豁免没有扩展到任意路径 |

地址条件由应用当时已有的转发头取址机制构造，没有为测试增删生产黑名单。这个机制本身仍缺可信代理边界核验，本轮未解决；测试也不能证明最初扫描者是谁。**局部恢复与剩余安全缺口必须同时保留。**

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

### 实际修复曾先被另一条重试分支截走

历史桥接故障中的 `reasoning_without_action` 是适配器主动发出的失败状态：上游有推理片段，但没有最终答案或结构化工具调用。把它展示成“重连”容易误导调查方向；补一段友好结束语也不能代替执行动作。

当时实施了一次有界兼容重试：关闭该上游支持的 thinking 参数、移除 reasoning effort，并为过小输出预算设置下限。第一次定向测试却失败，测试预期的 thinking 参数不存在。原因不是新逻辑完全没有运行，而是搜索后的 follow-up 响应先被旧分支归为“空回答”，走了另一条重试路径。

修正是在流式 chunk 层区分“有 reasoning、无正文”和“完全无内容”，再决定分支。随后对应测试通过；运行包的最小流式与非流式请求取得有效回答，日志出现一次 reasoning-only 重试事件。仍只有推理的二次响应继续失败，没有伪装成 completed。

这证明了特定适配路径的修复，不证明所有模型、工具和长任务都已兼容。真正的工具执行仍须有结构化调用及回执；输出文本里写着“将调用某工具”，不属于调用证据。

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

### 先用实际线程证据推翻“每轮都重建”

一次只读审计检查了同一 CAD 会话的 14 个历史任务：首次 start，之后 13 次 resume 均成功，没有 missing-thread 回退。另一个无工程绑定的三回合测试中，三个执行进程 PID 不同，threadId 却相同，且后两轮恢复成功。这直接说明进程寿命与会话寿命不同；它没有证明所有恢复分支都正确。

普通任务还有更具体的输入对照：第二回合数据库保存的 prompt 为 2,291 字符，rollout 中实际新增宿主消息只有 1,148 字符，与精简续聊消息一致。数据库存档、宿主新增输入、最终模型请求是三种东西；本次未取得完整模型线缆请求，不能从存档大小推算计费 token。

### 逐调用统计改变了优化优先级

长任务统计按任务时间边界筛选完成事件，再按 MCP 调用 ID 去重；以下大小是 text 内容的 UTF-8 字节，不含独立 image 块，也不是 token：

| 观察项 | 实测 | 工程含义 |
|---|---:|---|
| MCP 调用及文本总量 | 77 次；2,546,830 字节 | 不能按重复事件行累计 |
| 文档读取占文本量 | 47.68% | 先检查文档负载，而非凭感觉减少状态查询 |
| 两次截图占文本量 | 41.21% | 需检查响应结构 |
| 异常截图 / 正常截图文本量 | 1,048,604 / 941 字节 | 前者把完整 MCP 容器再次序列化进 text，图片 data 也随之进入文本 |
| 完整工具卡 | 41 份、12 种正文 | 29 次旧正文重读均跨越压缩；同阶段重复为 0 |
| 状态摘要占文本量 | 0.44% | revision 或状态变化有实际用途，不是主要负担 |

两次截图参数相同，但图片哈希不同，不能称为同一图片的前后压缩实验。证据能确认错误信封形状，尚不能把历史故障精确定位到当前源码某一行。12 次压缩中，前 5 次发生在异常截图之前；“去掉图片文本即可消除全部压缩”因此被时间顺序推翻。

### 后续候选改动与有限 A/B

后续改动识别已经成型的 MCP 内容，避免二次文本包装；按发布元数据提供本代必需工作流，保留完整参数卡、fresh 与压缩后的补读。它没有采用“服务端曾发送过，就永不再给正文”的缓存策略。

首个候选还暴露旧提示要求补读完整旧文档、隔离目录过长导致截图写盘失败。修正提示与工作目录后另做最终对照，早期试跑未混入结果表。最终 A/B 使用相同输入包、空工程和模型配置，但分别运行原便携程序与隔离源码服务，只是一对观察样本：

| 同一主体阶段指标 | A | B |
|---|---:|---:|
| MCP 文本字节 | 975,718 | 710,214 |
| 原生压缩次数 | 4 | 3 |
| 建模回合秒数 | 541.010 | 508.724 |
| 准备加建模秒数，不含人工等待 | 597.695 | 590.113 |

文本下降 27.2%，但准备加建模只下降约 1.27%。两轮 20 个 B-rep 资产哈希集合相同；细节、全部视角、截面及工程复开门禁未完成，正式比较器仍判“基线范围不完整，不可作完整任务比较”。而且两轮图片进入 text 的字节均为 0，不能把这次收益归功于现场消除了历史图片故障。源码已提交，生产便携包未在该轮更新。

可迁移的判断方法是：**先按任务边界量返回体积，再按压缩阶段解释重复，最后以同范围产物限定性能结论。** 几何是否符合设计另见[第 05 篇](05_AI_CAD的几何验证与恢复.md)。

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

This revision draws on historical execution records for access rejection, model-bridge repair, session auditing, and context optimization. Organizational identities and internal identifiers are removed, but causal chronology is preserved. Measurements describe specific samples, not platform performance guarantees. Diagnosis, candidate implementation, and deployment are distinguished.

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

In this investigation, the user removed the address from the persistent blacklist and reported continued rejection. Source inspection found an early return whenever the static set was already initialized; only the first access read the file. File modification time did not invalidate the cache. An updated file and continued rejection were therefore compatible observations.

Distinguish disk configuration, process cache, and actual request behavior. Reading the file is insufficient, and refreshing a client does not force the server to reload it.

Use the service's supported refresh mechanism for immediate recovery. Version checks, change detection, or explicit invalidation are possible longer-term designs. The choice depends on the environment; restarting the entire machine should not be a default diagnostic step.

### Confirm the result

Confirm the updated restriction, establish that the process loaded the new state, and rerun the same minimal request. Keep authentication and business authorization in place. Correcting a false block does not justify unrestricted access.

### Do not merge two stages into one successful repair

The earlier session established a 404 from the access check and a static cache unaffected by file edits. It recommended recycling the relevant application pool, but retained no successful recovery receipt from that session. That recommendation is not evidence of an implemented hot-reload mechanism.

A later recurrence has deployment and comparison-request evidence. The remedy retained the address blacklist and added authenticated admission only for exact MCP paths, requiring secure transport, permitted methods, valid credentials, an active account, and existing MCP permissions. Errors or an unavailable authentication helper still denied access. Downstream business authorization remained active. This was chosen over an exit-address allowlist because a shared exit does not identify a trusted user.

| Request condition after the later deployment | Observed result | Supported conclusion |
|---|---|---|
| Blocked-address condition, valid credential, initialization | 200 | The authenticated path reached MCP |
| Same address condition, invalid credential | 404 | The blacklist was not unconditionally lifted |
| Same address condition, valid credential, scanner path | 404 | Admission did not extend to arbitrary paths |

The address condition used the application's existing forwarding-header interpretation; tests added or removed no production blacklist entries. Trusted-proxy validation for that mechanism remained unresolved. The tests also did not identify the original scanner. **Local recovery and the remaining security gap belong in the same account.**

## 3. Case two: model output contains no actionable result

An upstream response may contain reasoning but no final answer or tool call. Network communication can succeed while the executor has no action to process.

Check whether an actionable call existed upstream, whether adaptation preserved it, whether ending events matched the content, and whether the executor continued.

Only after establishing missing action should compatibility retry be considered. One bounded design retries with parameters suited to the model after confirming that no business tool executed. Continued absence of a valid result should remain an explicit failure. Parameters and retry counts are implementation choices.

When a tool might already have changed data, establish its execution state first. Reissuing a model request and repeating a business write carry different risks.

Verify ordinary responses, tool round trips, stream endings, and failures. A successful minimal chat validates only that chat path.

### The first repair was intercepted by another retry branch

In the historical bridge failure, `reasoning_without_action` was an intentional adapter failure: upstream reasoning arrived without a final answer or structured tool call. A UI reconnect label obscured that distinction; a friendly closing sentence would not supply the missing action.

The implemented compatibility retry was bounded to one attempt. It disabled the upstream's supported thinking option, removed reasoning effort, and imposed a floor on an unusually small output budget. The first targeted test failed because the expected thinking parameter was absent. The post-search follow-up had first been classified as an empty answer and routed through an older retry branch.

The correction distinguished reasoning without answer text from completely empty content at the streaming-chunk layer. Targeted tests then passed. Minimal streaming and non-streaming requests against the running package returned usable answers, and the log recorded a reasoning-only retry. A second reasoning-only result still failed instead of becoming a false completed response.

This validates that adaptation path, not every model, tool, or long task. Tool execution still requires a structured call and receipt; text announcing an intended tool call is not execution evidence.

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

### Test the claim that every turn starts over

A read-only audit examined 14 historical tasks in one CAD conversation: an initial start followed by 13 successful resumes, with no missing-thread fallback. In a separate three-turn test without a bound project, three different executor PIDs used the same threadId, with successful resume on the last two turns. Process lifetime and conversation lifetime were demonstrably different; this did not validate every recovery branch.

An ordinary-task comparison was equally instructive: the second turn's database prompt contained 2,291 characters, but its actual new host message in the rollout contained 1,148, matching the compact continuation message. Stored prompts, new host inputs, and final model requests are distinct. Complete model wire requests were unavailable, so stored size could not establish billable tokens.

### Per-call measurements changed the priorities

The long-task analysis selected completion events within the task boundary and deduplicated MCP call IDs. Sizes below are UTF-8 bytes of text content, excluding separate image blocks; they are not tokens.

| Observation | Measurement | Engineering implication |
|---|---:|---|
| MCP calls and total text | 77; 2,546,830 bytes | Do not count repeated event rows as new calls |
| Documentation share | 47.68% | Investigate documentation before suppressing state queries |
| Two screenshots' share | 41.21% | Inspect the response envelope |
| Abnormal / normal screenshot text | 1,048,604 / 941 bytes | The former serialized an entire MCP container, including image data, into text again |
| Complete tool cards | 41 returns, 12 distinct bodies | All 29 repeated bodies crossed a compaction boundary; none repeated within a phase |
| State-summary share | 0.44% | Revision or state changes justified reads; this was not the dominant load |

The screenshot arguments matched, but image hashes differed. This was not a before/after compression experiment on one image. The malformed envelope was established, but its historical production path was not traced to an exact line of current source. Five of twelve compactions preceded the abnormal screenshot, disproving the claim that removing image text would explain every compaction.

### Candidate changes and a limited A/B comparison

The later change recognized already-formed MCP content to prevent duplicate text wrapping. Publisher metadata selected required workflows for the current context generation, while complete parameter cards, fresh reads, and post-compaction recovery remained available. It did not assume that previously sent content must still be present in the model's context.

The first candidate also exposed an old prompt demanding full legacy documents and a screenshot-write failure caused by an overly long isolated path. After correcting the prompt and working directory, a final comparison was run; exploratory results were kept separate. A and B used the same input package, empty projects, and model settings, but ran the original portable program and an isolated source service respectively. This was one observed pair:

| Metric for the same main-body stage | A | B |
|---|---:|---:|
| MCP text bytes | 975,718 | 710,214 |
| Native compactions | 4 | 3 |
| Modeling-turn seconds | 541.010 | 508.724 |
| Preparation plus modeling seconds, excluding human waits | 597.695 | 590.113 |

Text fell 27.2%, while preparation plus modeling fell only about 1.27%. The sets of hashes for 20 B-rep assets matched. Detail work, all views, sections, and project-reopen gates were incomplete; the formal comparator still rejected a full-task comparison because the baseline scope was incomplete. Both runs already had zero image bytes embedded in text, so the observed gain cannot be credited to eliminating that historical image fault in the live baseline. Source changes were committed; the production portable package was not updated in that round.

The transferable method is to **measure payloads within task boundaries, interpret repetition across compaction phases, and constrain performance claims to equivalent artifacts and scope**. Design correctness is addressed separately in [Article 05](05_AI_CAD的几何验证与恢复.md).

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
