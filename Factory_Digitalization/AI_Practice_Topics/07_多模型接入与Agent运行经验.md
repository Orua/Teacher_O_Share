# 多模型接入与 Agent 运行的排障方法

## 中文

## 沿调用链查证据

一次助手任务可能经过任务入口、执行服务、模型适配、上游模型和业务工具。相同的表面错误可能来自不同层，排障需要确认实际调用路径。

## 通用案例：模型有输出，但没有完成动作

某模型返回了推理内容，却没有最终答案或工具调用。适配层若直接报告成功，执行服务便无法继续任务。

可以先用最小请求复现，检查响应和结束事件，确认是模型没有给动作、适配没有正确解析，还是调用方没有继续处理。若确需兼容重试，应限制次数、记录原因，仍无有效结果时明确失败。

重试参数依赖模型与客户端能力，不应照搬一套设置。涉及写入时，先确认上一轮是否已经执行，再决定重试。

## AI 怎样参与

AI 可以对照日志、检查转换代码、构造最小样本、比较流式与非流式行为，并提出兼容修改。人提供失败的时间、任务条件与预期结果，确认哪些环境设置允许调整。

排查顺序可以是：调用者是谁 → 实际模型和路由是什么 → 上游返回什么 → 适配返回什么 → 工具是否执行 → 用户最终收到什么。

## 容易踩的坑

一句正常回答不能证明工具往返成功。网络连通不能证明鉴权通过。适配代码已修改，也不能证明当前运行实例来自新产物。

另一个坑是改变太多变量：模型、网络、代理、预算和超时一起调整，会使偶然成功难以解释。应固定条件，逐项比较。

日志缺少进程或请求标识时，结论要停在现有证据能够支持的层级，不能把时间相近当作因果。

## 可复用的检查范围

按任务需要覆盖普通回答、工具调用与结果返回、流式结束、明确失败，以及取消或长任务恢复。保留时间、请求标识、实际路由、结束状态和错误层级，避免记录凭证。

评价模型时使用相同任务、工具与完成标准，比较正确结果、延迟、使用成本和人工修正。本文不对具体模型或服务排名，也不依赖特定模型适配产品。

---

# Troubleshooting Multiple Models and Agent Execution

## English

## Follow the evidence through the call chain

An assistant task can pass through an entry point, execution service, model adapter, upstream model, and business tool. Similar symptoms can originate at different layers. Establish the actual path before diagnosing it.

## Generic example: output without an actionable result

A model returns reasoning content but no final answer or tool call. If the adapter reports success, the executor cannot progress.

Reproduce the issue with a minimal request and inspect the response and ending events. Determine whether the model omitted an action, the adapter parsed it incorrectly, or the caller failed to continue. If a compatibility retry is justified, bound it, record the reason, and return a clear failure when it still yields no valid result.

Parameters depend on model and client capabilities. For operations that can change data, establish whether the previous attempt executed before retrying.

## How AI participates

AI can compare logs, inspect conversion logic, construct minimal samples, compare streaming and non-streaming behavior, and propose compatibility changes. People provide failure timing, task conditions, expected results, and limits on environment changes.

A useful sequence is: identify the caller, actual model and route, upstream response, adapted response, tool execution, and final user-visible result.

## Common pitfalls

One ordinary answer does not validate a tool round trip. Network reachability does not establish successful authentication. Changed adapter source does not prove that the current runtime uses the new artifact.

Changing the model, network, routing, budget, and timeout together makes an apparent improvement difficult to explain. Hold conditions steady and compare one change at a time.

When logs lack process or request identifiers, stop at the level supported by evidence. Similar timestamps alone do not establish causation.

## Reusable checks

Cover normal answers, tool calls and returned results, stream completion, explicit failure, and cancellation or long-task recovery when needed. Record timing, request identifiers, actual routing, termination, and error layers without credentials.

Compare models using the same tasks, tools, and acceptance criteria. Evaluate correct outcomes, latency, usage cost, and manual correction. This article does not rank particular models or depend on a specific adapter product.
