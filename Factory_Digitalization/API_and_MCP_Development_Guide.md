# 通用 API 与 MCP 开发流程指南
# A Practical Guide to API and MCP Development

> **中文**：面向希望让 AI Agent 使用业务系统的开发者，适用于不同语言、框架、部署方式及已有系统改造。本文是可按项目规模裁剪的实践流程，不代表任何具体系统的开发或验收记录。
>
> **English**: For developers enabling AI agents to use business systems, across languages, frameworks, deployment models, and existing-system integrations. This is a practical workflow that can be scaled to the project. It is not an implementation or acceptance record for any particular system.

## 1. 明确目标与交付路径 / Define the Goal and Delivery Workflow

### 中文

目标是让 Agent 在授权范围内正确查询和操作业务系统，并能解释结果、追踪操作。

```text
明确场景与授权范围
→ 梳理 UI 功能与业务流程
→ 追查源码、数据关系及关联处理
→ 定义 API 契约
→ 实现或复用业务服务
→ 封装 MCP 工具
→ 完成身份与权限接入
→ 做针对性测试
→ 按授权部署、确认并交付使用说明
```

身份与权限应从接口设计阶段同步考虑和实现，不能等到业务开发结束才补上。

### English

The goal is to let agents query and operate business systems within their authorized scope, explain results, and leave a traceable record of operations.

```text
Define scenarios and authorization boundaries
→ Map UI capabilities and business workflows
→ Trace source code, data relationships, and related processing
→ Define API contracts
→ Implement or reuse business services
→ Expose MCP tools
→ Complete identity and permission integration
→ Run targeted tests
→ Deploy as authorized, verify, and deliver usage instructions
```

Design and implement identity and permissions alongside the API contract, rather than adding them after business development is complete.

## 2. 从 UI 追到业务逻辑 / Trace the UI to the Business Logic

### 中文

从实际任务开始，例如“查某客户未完成订单”“解释本月销售额”“修改一张单据”。围绕任务检查相关页面，无需一开始遍历整个系统。

| 检查对象 | 需要确认的问题 |
|---|---|
| 查询与详情 | 能查什么？条件如何组合？默认时间范围是什么？ |
| 字段与名称 | 哪些是业务编号，哪些是数据库 ID？数量、金额单位是什么？ |
| 新增与修改 | 必填项、默认值、自动计算、不可修改字段是什么？ |
| 删除与状态 | 是删除、作废还是状态流转？哪些条件下允许？ |
| 业务联动 | 保存后会同步哪些资料、库存、状态、通知或日志？ |
| 权限 | 谁能查看或修改？是否受组织、租户或资料归属限制？ |

UI 是业务发现入口。应继续沿页面事件、请求处理、业务服务、数据访问或外部系统追查，并结合已有源码、接口和测试确认，不必全部通过点击完成。

特别关注“保存之后发生什么”：修改订单可能同时涉及校验、金额重算、库存预占、关联单同步和审计。只更新订单表，并不等于实现完整业务能力。

| 用户动作 | 入口 | 业务处理 | 关联影响 | 权限 | 对应接口 |
|---|---|---|---|---|---|
| 修改订单 | 编辑页保存 | 校验并更新订单 | 重算金额、同步关联状态、留底 | 可编辑目标订单 | `order.update` |

没有 UI 的新功能，通过需求访谈、业务规则和样例建立同类映射。

### English

Start with real tasks, such as finding a customer's outstanding orders, explaining monthly sales, or updating a business document. Inspect the relevant pages without surveying the entire system first.

| Area | Questions to resolve |
|---|---|
| Search and details | What can be queried? How do filters combine? What is the default date range? |
| Fields and names | Which values are business references versus database IDs? What are the quantity and currency units? |
| Create and update | What is required, defaulted, calculated automatically, or immutable? |
| Deletion and status | Is the action deletion, voiding, or a status transition? When is it allowed? |
| Related processing | What records, inventory, statuses, notifications, or logs change after saving? |
| Permissions | Who may view or edit? Do organization, tenant, or ownership restrictions apply? |

The UI is a starting point. Follow page events, request handlers, business services, data access, and external integrations. Use existing source code, APIs, and tests where available; not every step requires clicking through the interface.

Pay particular attention to what happens after saving. An order update may validate fields, recalculate amounts, reserve inventory, synchronize related documents, and record an audit event. Updating the order table alone does not reproduce the complete business operation.

| User action | Entry point | Business processing | Related effects | Permission | API operation |
|---|---|---|---|---|---|
| Update order | Save on edit page | Validate and update order | Recalculate amounts, synchronize statuses, record audit | May edit the target order | `order.update` |

For new capabilities without a UI, build the same mapping from stakeholder discussions, business rules, and examples.

## 3. API 与 MCP 的分工 / Separate API and MCP Responsibilities

### 中文

API 提供业务能力；MCP 让 Agent 发现并调用这些能力。两者应尽量复用业务服务及权限规则。

```text
网页 / 移动端 / 其他程序 → API 适配层 ─┐
                                    ├→ 业务服务 → 数据库 / 外部服务
AI Agent → MCP 工具适配层 ────────────┘
             各入口采用统一的身份与权限规则
```

MCP 可以调用已有 API，也可以在同一进程内调用业务服务。不强制增加 HTTP 调用或另建微服务，应选择适合现有架构、易于维护的方案。

复杂业务规则放在共用业务服务中；分页适配、参数转换和响应包装可以保留在各入口。

### English

APIs expose business capabilities. MCP makes those capabilities discoverable and callable by agents. Both should reuse business services and permission rules wherever practical.

```text
Web / Mobile / Other applications → API adapter ─┐
                                                ├→ Business services → Database / External services
AI agent → MCP tool adapter ─────────────────────┘
            Consistent identity and permission rules across entry points
```

MCP may call an existing API or invoke business services within the same process. An additional HTTP request or a separate microservice is not mandatory. Choose an implementation that fits the existing architecture and remains maintainable.

Keep complex business rules in shared services. Pagination adaptation, parameter conversion, and response formatting may remain in the adapters.

## 4. 先定义契约 / Define Contracts Before Coding

### 中文

每个操作至少说明：

- **用途**：解决什么问题、适用什么场景。
- **输入**：类型、必填条件、默认值、枚举、日期时区及单位。
- **查询规则**：筛选组合、排序、分页、空结果及总数口径。
- **输出**：字段含义及必要的关联 ID、编号、名称。
- **权限**：操作权限、资料范围、敏感字段可见性。
- **副作用**：改变什么，是否涉及事务或外部调用。
- **错误**：区分无资料、无权限、参数错误、状态冲突和能力不可用。
- **重试**：是否可安全重试，写入如何避免重复执行。

可以按列表、详情、统计、新增、修改和状态操作分类，但不必为每张表机械生成全部增删改查。以“审批”“取消”“入库”等业务动作为单位，通常更容易保持业务规则完整。

HTTP API 可以使用 OpenAPI 描述契约，参考文末官方规范。

### English

Document at least the following for each operation:

- **Purpose**: the problem it solves and when to use it.
- **Inputs**: types, required conditions, defaults, enums, date/time zones, and units.
- **Query behavior**: filter combinations, sorting, pagination, empty results, and total-count definitions.
- **Outputs**: field meanings and necessary related IDs, references, and names.
- **Permissions**: allowed actions, accessible records, and sensitive-field visibility.
- **Side effects**: changes made, transaction boundaries, and external calls.
- **Errors**: distinguish no data, access denied, invalid input, status conflicts, and unavailable capabilities.
- **Retries**: whether retries are safe and how duplicate writes are prevented.

Operations may be grouped into lists, details, summaries, creation, updates, and status changes. Do not automatically generate every CRUD operation for every table. Business actions such as approve, cancel, or receive stock often preserve rules more effectively than unrestricted field updates.

HTTP APIs can use OpenAPI to describe their contracts. See the official reference at the end.

## 5. 设计 Agent 能正确使用的工具 / Design Tools Agents Can Use Correctly

### 中文

工具说明应回答“何时用、怎么用、会产生什么结果”。

- 能力较少时，直接提供 `search_orders`、`get_order`、`update_order` 等具体工具。
- 能力较多时，可按业务域拆分，或采用少量入口、操作目录和按需帮助。

通用入口加动态帮助是应用层设计，不是 MCP 的统一要求。具体工具应提供清晰 schema；通用入口也必须具备可靠的操作发现和参数校验机制。

说明需要包含业务语义，例如客户名称是否包含匹配、日期依据创建时间还是交期、是否包含取消订单。

结果保持结构化，包含必要的编号、名称、单位及状态，并区分空结果、失败、无权限和模块不可用。写入工具应明确副作用；需要确认时，确认必须对应本次具体动作和参数。界面提示、模型承诺或工具描述不能代替服务端权限检查。

### English

Tool descriptions should explain when to use a tool, how to call it, and what it will do.

- For a small capability set, expose specific tools such as `search_orders`, `get_order`, and `update_order`.
- For larger systems, organize tools by business domain, or use a small set of entry points with an operation catalog and on-demand help.

A generic entry point with dynamic help is an application design choice, not a universal MCP requirement. Specific tools need clear schemas. Generic tools still need reliable operation discovery and parameter validation.

Explain business semantics: whether customer names use substring matching, whether dates mean creation dates or delivery dates, and whether canceled orders are included.

Return structured results with the necessary references, names, units, and statuses. Distinguish empty results, failures, denied access, and unavailable modules. Write tools must describe their side effects. When confirmation is required, bind it to the specific action and parameters. UI prompts, model promises, and tool descriptions do not replace server-side authorization.

## 6. 每人独立身份与凭据 / Give Each User an Independent Identity and Credential

### 中文

统一鉴权意味着各入口采用一致的身份解析和权限规则，不意味着所有人共用管理员凭据。用户通常可以共用同一个服务地址，由各自凭据确定身份。

```text
用户甲 → 共用 MCP 地址 + 甲的凭据 → 甲的身份与权限
用户乙 → 共用 MCP 地址 + 乙的凭据 → 乙的身份与权限
```

在受控环境、客户端支持时，可采用自定义的 `UserID + 个人 Key` 方案：

```text
MCP 地址：https://service.example/mcp?userid=<用户ID>
请求头：Authorization: Bearer <个人Key>
```

这只是占位示例。服务端必须验证 Key 与 UserID 的绑定；修改 UserID 不得获得他人权限。也可直接由 Key 识别用户，省去地址里的 UserID。Key 应放在认证头中，避免进入查询字符串、日志、浏览历史和分享链接。

自定义方案不等同于 MCP 标准授权流程，也不保证所有客户端支持。面向通用远程 HTTP MCP 客户端时，应按选定协议版本及客户端能力实现授权；标准 OAuth 接入需要验证令牌确实面向当前服务签发。

凭据应支持生成、撤销和轮换，按需要设置有效期、权限范围与调用限制。若只需验证 Key，可保存不可逆校验值，并仅在生成时展示原始 Key。

不同客户端可分别发放凭据，便于单独撤销；自动化程序使用独立服务身份。文档、代码示例、审计记录和对话中不得保存真实 Key、令牌或 Cookie。

### English

Consistent authentication means using the same identity-resolution and permission rules across entry points. It does not mean sharing an administrator credential. Users can usually share one service URL while their individual credentials determine their identities.

```text
User A → Shared MCP URL + A's credential → A's identity and permissions
User B → Shared MCP URL + B's credential → B's identity and permissions
```

In a controlled environment with compatible clients, a custom `UserID + personal key` scheme may be used:

```text
MCP URL: https://service.example/mcp?userid=<user-id>
Header: Authorization: Bearer <personal-key>
```

These are placeholders only. The server must verify that the key belongs to the supplied UserID. Changing the UserID must not grant another user's permissions. Alternatively, identify users directly from their keys and omit the UserID from the URL. Put keys in authentication headers to avoid exposing them through query strings, logs, browser history, or shared links.

A custom scheme is not equivalent to the standard MCP authorization flow, and client support varies. For general remote HTTP MCP clients, implement authorization for the selected protocol version and client capabilities. Standard OAuth integration must verify that tokens are intended for the current service.

Support credential issuance, revocation, and rotation. Set expiration, scopes, and request limits as needed. If only key verification is required, store a one-way verifier and display the original key only when issued.

Issue separate credentials per client when useful for independent revocation. Give automation its own service identity. Never store real keys, tokens, or cookies in documentation, code examples, audit records, or conversations.

## 7. 每次调用都检查业务权限 / Enforce Business Permissions on Every Call

### 中文

认证回答“你是谁”；授权回答“你能对哪些资料做什么”。

```text
验证凭据 → 取得服务端可信身份 → 检查账号状态及凭据范围
→ 检查操作权限 → 检查组织、租户与资料归属
→ 检查敏感字段及业务状态限制 → 执行业务 → 记录结果
```

工具列表可按权限显示，但隐藏工具不能代替执行时授权。能连接 MCP 不代表能调用所有工具。

操作者身份来自可信认证上下文，不得被业务参数覆盖。“负责人 ID”“目标用户 ID”可以作为业务参数，但必须与实际操作者区分，并验证用途。

API 与 MCP 共用身份规则，不代表能把收到的令牌直接透传给任意下游服务。跨服务调用应使用下游认可的凭据或正式委托机制；参见文末 MCP 授权参考。

### English

Authentication answers who you are. Authorization determines what you may do to which records.

```text
Validate credential → Establish trusted server-side identity → Check account status and credential scope
→ Check operation permission → Check organization, tenant, and record ownership
→ Enforce sensitive-field and business-state restrictions → Execute → Record the result
```

Tool visibility may depend on permissions, but hiding tools does not replace authorization during execution. Connecting to MCP does not grant access to every tool.

The acting identity must come from trusted authentication context and must not be overridden by business parameters. An owner ID or target user ID may be a valid business parameter, but it must remain distinct from the caller's identity and be validated for its intended use.

Shared identity rules do not permit forwarding incoming tokens to arbitrary downstream services. Use credentials accepted by the downstream service or a formal delegation mechanism. See the MCP authorization references below.

## 8. 按规模分工并复核 / Divide Work According to Scale and Review Results

### 中文

大任务可按职责并行；小任务由一个 Agent 完成即可。

| 角色 | 主要任务 | 交付物 |
|---|---|---|
| 业务梳理 | 检查 UI、用例、数据关系与联动 | 功能与逻辑映射 |
| 接口实现 | 定义契约、复用业务逻辑、编程 | API 与 MCP 实现 |
| 验证 | 对照页面、真实样例与权限 | 验证结果与差异 |
| 主负责人 | 汇总结论、解决冲突、确认范围 | 可交付版本与未完成项 |

分工时明确目录、模块、读写授权和验收要求，减少同时修改同一文件。业务研究默认只读；生产写入、删除、对外发送和部署按已有明确授权执行，委派不扩大权限。

### English

Large tasks may be divided by responsibility. A single agent is sufficient for small tasks.

| Role | Responsibility | Deliverable |
|---|---|---|
| Business analysis | Inspect UI, use cases, data relationships, and related processing | Capability and logic mapping |
| Implementation | Define contracts, reuse business logic, and write code | API and MCP implementation |
| Verification | Compare pages, representative records, and permissions | Results and discrepancies |
| Lead | Consolidate findings, resolve conflicts, and confirm scope | Deliverable version and outstanding items |

Specify directories, modules, read/write authorization, and acceptance criteria. Avoid simultaneous edits to the same files. Business investigation is read-only by default. Production writes, deletions, external messages, and deployment follow existing explicit authorization; delegation does not expand it.

## 9. 三层针对性验证 / Run Targeted Verification at Three Levels

### 中文

1. **契约层**：检查参数、必填项、响应结构、错误语义、工具发现与说明。
2. **业务层**：选择直接相关的真实或受控样例，对照页面、业务规则和数据状态；关注筛选、分页、统计口径及写入联动。
3. **权限层**：检查有效和错误凭据、用户绑定、权限不足及跨资料范围访问；涉及多租户时检查隔离。

写入测试优先使用隔离环境：记录初始状态 → 执行 → 读回主记录及关联状态 → 检查审计 → 按计划清理。存在网络重试时，按风险验证幂等性或去重；跨系统操作需要说明部分成功的处理方式。

验证深度按改动和风险决定。普通改动通过一轮针对性验证即可交付，不默认扩展为全站审计。

| 证据 | 可以支持的结论 |
|---|---|
| 构建通过 | 当前实现可构建 |
| 工具可列出、帮助可读取 | 工具发现和说明入口可用 |
| 接口返回成功 | 本次调用返回成功，具体业务结果仍需核对 |
| 业务对照通过 | 已检查场景符合业务规则 |
| 部署确认通过 | 目标环境中的指定能力可用 |

不能把“接口数量齐全”或“返回成功”写成全部业务已验收。

### English

1. **Contract level**: verify parameters, required fields, response structures, error meanings, tool discovery, and documentation.
2. **Business level**: use directly relevant real or controlled examples and compare them against the UI, business rules, and data state. Focus on filtering, pagination, aggregation definitions, and related write effects.
3. **Permission level**: check valid and invalid credentials, user binding, insufficient permissions, and access outside the allowed record scope. Test tenant isolation where applicable.

Prefer isolated environments for write tests: record the initial state → execute → read back primary and related records → inspect audit records → clean up as planned. Where network retries occur, verify idempotency or deduplication according to risk. Explain how partial success is handled across systems.

Match verification depth to the change and its risk. An ordinary change can be delivered after one targeted verification pass; a full-system audit is not the default.

| Evidence | Supported conclusion |
|---|---|
| Build passes | The implementation can be built |
| Tools are listed and help is readable | Discovery and help entry points work |
| API reports success | This call reported success; the business result still needs checking |
| Business comparison passes | The checked scenarios follow the business rules |
| Deployment verification passes | The specified capability works in the target environment |

A complete API list or a successful response does not establish acceptance of all business behavior.

## 10. 部署与使用说明 / Deployment and Usage Instructions

### 中文

部署前保留必要备份，确认目标环境，保护现有配置及无关修改。按授权部署已验证的候选版本，并完成一次与改动直接相关的线上确认。

交付说明应包括：完成内容、连接方式、个人凭据取得和撤销方法、可用工具、验证层级、部署状态及剩余限制。

文档同时服务人和 Agent：人能理解业务与权限，Agent 能获取准确参数及调用方式。接口、工具 schema 和说明应同步更新。

### English

Before deployment, retain necessary backups, confirm the target environment, and protect existing configuration and unrelated changes. Deploy the verified candidate as authorized and perform one live check directly related to the change.

The handover should state what was completed, how to connect, how to obtain and revoke personal credentials, which tools are available, the verification level, deployment status, and remaining limitations.

Documentation serves both people and agents. People need clear business and permission explanations; agents need accurate parameters and calling instructions. Update APIs, tool schemas, and documentation together.

## 11. 开发 Agent 任务模板 / Development Agent Task Template

### 中文模板

```text
目标：
为【业务模块】提供 API 与 MCP 能力，使用户能完成【具体场景】。

范围与限制：
- 允许读取：【页面、源码、文档、测试数据】。
- 允许修改：【模块与目录】。
- 数据写入、删除、外部发送及部署授权：【明确范围】。
- 保护已有配置及无关修改；凭据不得写入日志或文档。

执行要求：
1. 先说明下一步动作，再执行。
2. 梳理相关 UI、字段、筛选、校验、权限及保存后联动。
3. 追查实际实现，提交功能与逻辑映射。
4. 定义参数、结果、错误、副作用与重试语义。
5. 复用业务服务实现 API 与 MCP，避免重复业务规则。
6. 使用服务端可信身份，检查操作权限及目标资料范围。
7. 做与改动直接相关的契约、业务及权限验证。
8. 按授权部署、确认并更新使用说明。

验收：
- 指定场景能通过接口完成，结果符合原业务规则。
- 个人凭据绑定正确，不能通过参数冒用他人身份。
- 写入的关联状态和审计符合约定。
- 文档与实际参数、返回及权限一致。
- 区分已实现、已测试、已部署和未验证项。

最终输出：
变更摘要、使用示例、验证范围、部署状态、限制和文件位置。
```

### English Template

```text
Goal:
Provide API and MCP capabilities for [business module] so users can complete [specific scenarios].

Scope and limits:
- May read: [pages, source code, documents, test data].
- May modify: [modules and directories].
- Authorized writes, deletions, external messages, and deployment: [explicit scope].
- Protect existing configuration and unrelated changes. Never put credentials in logs or documents.

Execution requirements:
1. State the next action before executing it.
2. Map relevant UI, fields, filters, validation, permissions, and post-save effects.
3. Trace the actual implementation and deliver a capability and logic mapping.
4. Define parameters, results, errors, side effects, and retry behavior.
5. Reuse business services for API and MCP instead of duplicating business rules.
6. Use trusted server-side identity; check operation permissions and target record scope.
7. Run contract, business, and permission checks directly related to the change.
8. Deploy as authorized, verify, and update usage instructions.

Acceptance:
- The specified scenarios work through the interfaces and follow existing business rules.
- Personal credentials are correctly bound; parameters cannot impersonate another user.
- Related write effects and audit records match the contract.
- Documentation matches actual parameters, responses, and permissions.
- Implemented, tested, deployed, and unverified items are clearly distinguished.

Final output:
Change summary, usage examples, verification scope, deployment status, limitations, and file locations.
```

## 12. 常见遗漏 / Common Omissions

| 中文 | English |
|---|---|
| 只按数据库字段做接口，忽略完整业务流程。 | Building interfaces from database fields without tracing the complete business workflow. |
| 只检查 UI 表面，未追查后台校验及关联动作。 | Inspecting only the UI without tracing backend validation and related processing. |
| API 与 MCP 各写一套规则，逐渐出现行为差异。 | Duplicating business rules in API and MCP implementations until behavior diverges. |
| 把 UserID 当作认证，未验证凭据绑定。 | Treating a UserID as authentication without validating credential binding. |
| 所有人共用管理员 Key，无法隔离权限或追踪操作者。 | Sharing an administrator key, preventing permission isolation and caller attribution. |
| 只限制工具显示，未限制实际调用和资料范围。 | Restricting tool visibility without enforcing permissions and record scope during execution. |
| 测试只看成功响应，未读回业务结果和关联状态。 | Checking success responses without reading back business results and related state. |
| 文档列出能力，却未说明限制和实际验证范围。 | Listing capabilities without stating limitations and actual verification coverage. |

**交付标准 / Delivery standard**：Agent 能理解能力、以正确身份调用，并取得符合业务规则的结果。 / The agent understands the capability, calls it under the correct identity, and obtains results consistent with business rules.

## 参考资料 / References

以下链接沿用原文，协议实现时应核对所选版本及客户端支持情况。
The following references are retained from the source. Check the selected protocol version and client support when implementing an integration.

- [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [MCP Authorization — 2026-07-28 reference](https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/docs/specification/2026-07-28/basic/authorization/index.mdx)
- [MCP Authorization — 2025-11-25 reference](https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/docs/specification/2025-11-25/basic/authorization.mdx)
