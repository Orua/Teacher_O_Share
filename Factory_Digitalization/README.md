# 数字化与 AI 实践：五篇深度课题

## 中文

这五篇是平级文章，围绕完整的开发与调试问题组织。每篇包含问题背景、调查过程、方案取舍、AI 与人的分工、验证方法和经验边界，完整中文在前、完整英文在后。

本轮修订回查了历史会话中的中间调查、工具回执、代码差异、测试记录与后续发布报告。案例移除企业、人员、业务单号、内部地址及原始日志；保留通用故障机制、测试夹具测量和明确定义的运行指标。文章中的方法建议不等于历史系统已实现的能力，样本通过也不等于完整业务验收。

证据按事件而非会话计数：同一故障的诊断、修复和复测连成一个案例；后来发布的结果不倒写进早期调查。内部来源索引留在本地，不随公开文章提交。公开规范链接只用于解释通用原理，不用于替代项目验证证据。

| 课题 | 主要内容 |
|---|---|
| [业务系统接入 AI 与可靠写入：API、MCP 和业务闭环](01_业务系统接入AI与可靠写入.md) | UI 关联校验、认证负测、旧日志语义遗漏，以及对象重号后的来源行续传。 |
| [知识工程与人机分工：让业务经验成为可执行的方法](02_知识工程与人机分工.md) | 历史样本与当前参数的冲突，旧质量证据压过新进度的误判及小样修正。 |
| [Agent 调试与上下文治理：从表面错误追到真正发生的事](03_Agent调试与上下文治理.md) | 访问层与缓存、推理空动作分支、线程复用证据、逐调用负载和有限 A/B。 |
| [AI 协作开发与交付闭环：从任务拆分到用户真正用上](04_AI协作开发与交付闭环.md) | 六任务的依赖、重复补丁与门禁冲突，从不完整候选走向限定交付。 |
| [AI CAD 的几何验证与恢复：让可见结果成为可靠产物](05_AI_CAD的几何验证与恢复.md) | 弯管正负夹具、布尔验收器矛盾、截面对应关系与两类冷恢复故障。 |

---

# Digitalization and AI Practice: Five In-Depth Topics

## English

These five peer articles follow complete development and debugging problems. Each covers context, investigation, design choices, human and AI roles, verification, and limitations, with the complete Chinese text before the complete English text.

This revision revisits intermediate investigations, tool receipts, code changes, test records, and later release reports. Organizational and personal identities, business references, internal addresses, and raw logs are omitted. Generic failure mechanisms, fixture measurements, and explicitly defined runtime metrics are retained. Suggested methods are not claims of historical implementation, and passing samples do not establish complete business acceptance.

Evidence is grouped by event rather than conversation: diagnosis, repair, and retesting of one fault form one case. Later releases are not retroactively presented as earlier results. Internal source indexes remain local and are excluded from publication. Public specifications explain general principles; they do not substitute for project verification.

| Topic | Coverage |
|---|---|
| [Connecting Business Systems to AI with Reliable Writes](01_业务系统接入AI与可靠写入.md) | UI relationships, negative authentication tests, legacy audit semantics, and source-row resumption after duplicate identities. |
| [Knowledge Engineering and Human–AI Work Allocation](02_知识工程与人机分工.md) | Historical samples versus current parameters, and sample corrections when old quality evidence overrides newer progress. |
| [Agent Debugging and Context Management: Establish What Actually Happened](03_Agent调试与上下文治理.md) | Access checks and caches, reasoning-only branches, thread reuse, per-call payloads, and limited A/B evidence. |
| [AI Collaboration and Delivery: From Task Division to Usable Results](04_AI协作开发与交付闭环.md) | Six-task dependencies, duplicate patches, conflicting gates, and progression from incomplete candidates to scoped delivery. |
| [AI CAD Geometry Validation and Recovery](05_AI_CAD的几何验证与恢复.md) | Positive and negative bend fixtures, contradictory Boolean evaluation, section correspondence, and two cold-recovery defects. |
