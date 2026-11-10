# 数字化与 AI 实践：五篇深度课题

## 中文

这五篇是平级文章，围绕完整的开发与调试问题组织。每篇包含问题背景、调查过程、方案取舍、AI 与人的分工、验证方法和经验边界，完整中文在前、完整英文在后。

案例由工程复核中的问题抽象而来，业务对象与表达已泛化，不包含企业身份、内部技术栈、真实数据或可回溯的内部标识。流程和建议适用于不同技术环境，不把建议方案写成已经实现的能力。

| 课题 | 主要内容 |
|---|---|
| [业务系统接入 AI 与可靠写入：API、MCP 和业务闭环](01_业务系统接入AI与可靠写入.md) | 从原业务动作追到接口、权限与审计，包含写入漏日志和批量导入恢复。 |
| [知识工程与人机分工：让业务经验成为可执行的方法](02_知识工程与人机分工.md) | 从知识规则冲突与风险误判，讨论数据口径、确定性处理和模型判断。 |
| [Agent 调试与上下文治理：从表面错误追到真正发生的事](03_Agent调试与上下文治理.md) | 围绕访问拦截、配置缓存、无动作响应和上下文压缩建立调试方法。 |
| [AI 协作开发与交付闭环：从任务拆分到用户真正用上](04_AI协作开发与交付闭环.md) | 分析依赖拆分、错误验收器、发布账本和跨组件版本落差。 |
| [AI CAD 的几何验证与恢复：让可见结果成为可靠产物](05_AI_CAD的几何验证与恢复.md) | 沿几何测量、工具评测、文件保存和冷重开检查最终产物。 |

---

# Digitalization and AI Practice: Five In-Depth Topics

## English

These five peer articles follow complete development and debugging problems. Each covers context, investigation, design choices, human and AI roles, verification, and limitations, with the complete Chinese text before the complete English text.

Cases generalize engineering-review problems without organizational identity, internal technical stacks, real operating data, or traceable internal identifiers. Methods apply across technical environments, and proposed designs are not presented as implemented capabilities.

| Topic | Coverage |
|---|---|
| [Connecting Business Systems to AI with Reliable Writes](01_业务系统接入AI与可靠写入.md) | Trace business actions through interfaces, permissions, and audit, including missing write logs and import recovery. |
| [Knowledge Engineering and Human–AI Work Allocation](02_知识工程与人机分工.md) | Resolve conflicting knowledge and risk judgments through sound data definitions and appropriate model/software roles. |
| [Agent Debugging and Context Management: Establish What Actually Happened](03_Agent调试与上下文治理.md) | Investigate access rejection, cached configuration, non-actionable responses, and context compaction. |
| [AI Collaboration and Delivery: From Task Division to Usable Results](04_AI协作开发与交付闭环.md) | Examine dependencies, defective evaluators, release ledgers, and mismatched runtime components. |
| [AI CAD Geometry Validation and Recovery](05_AI_CAD的几何验证与恢复.md) | Validate geometry, tool behavior, saved artifacts, and recovery in a fresh session. |
