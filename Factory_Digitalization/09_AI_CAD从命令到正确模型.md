# AI CAD：从命令执行到正确模型

## 中文

## 最终产物才是目标

建模助手可以生成步骤，工具也可以返回成功，但用户需要尺寸、结构和保存结果符合要求的模型。这一原则适用于不同建模软件、几何内核和自动化方式。

## 通用案例：带内孔的弯管

任务要求沿路径生成带内孔的管件。执行前明确截面、单位、路径、壁厚与误差要求，再由助手调用可用建模工具。

执行后按任务重点检查：孔道是否贯通，壁厚是否满足要求，实体数量是否正确，关键尺寸或体积是否合理。保存并导出，再重新读取，确认产物完整。视觉预览有帮助，但不能单独证明内部结构。

## AI 怎样参与

AI 帮助解释需求、选择操作顺序、阅读工具参数、生成调用、根据错误修正，并整理测量结果。人明确设计意图、关键约束和允许误差。

参考外部算法或资料时，还需要核对来源和使用条件。参考代码有助于理解方法，但不能据此跳过兼容性和分发检查。

## 工具测试与自主任务评测分开记录

固定步骤成功，可以证明该操作路径在对应样本上有效。让 AI 从需求出发独立完成，还要考察规划、选参、纠错和结束判断。

检查器也需要验证。用已知正确和已知错误模型确认检查器能够区分，再让它评价其他产物。不能因为自动检查输出失败，就直接认定几何一定错误。

## 容易踩的坑

命令被接受不等于形状正确。截图看起来完整不等于实体或内孔正确。导出文件存在不等于回读成功。界面与计算组件不配套，还可能使本地成功无法转化为实际可用。

长任务只留下打开的页面，也会增加续接风险。阶段成果应保存为可重开的工程或文件，并记录标识和测量结果；是否支持自动恢复，应以实际实现为准。

## 可复用的验收路径

需求约束 → 调用步骤 → 几何计算 → 直接测量 → 保存导出 → 重开回读。

按任务选择必要检查，不必每次枚举全部几何理论。失败时区分规划、参数、计算、传输、保存和检查器问题。局部样本成功应保留范围，不能推广为任意图纸均可自动完成。

本文使用通用管件案例，不包含实际产品编号、图纸或测量值。

---

# AI CAD: From Executed Commands to Correct Models

## English

## The deliverable is the model

A modeling assistant can generate steps and tools can report success, while the user still needs correct dimensions, structure, and saved output. This applies across modeling applications, geometry engines, and automation approaches.

## Generic example: a hollow curved tube

Define the section, units, path, wall thickness, and tolerances before invoking modeling tools.

Then check the properties that matter: a continuous internal passage, required wall thickness, expected solid count, and reasonable dimensions or volume. Save and export the model, then read it again to check completeness. A preview helps, but does not by itself establish internal structure.

## How AI participates

AI can interpret requirements, choose an operation sequence, read tool parameters, generate calls, correct errors, and explain measurements. People establish design intent, critical constraints, and acceptable tolerances.

External algorithms and references require source and usage checks. Reading reference code does not remove compatibility or distribution obligations.

## Separate tool checks from autonomous task evaluation

Successful fixed steps establish that a particular operation path works for the sample. Autonomous completion additionally evaluates planning, parameter selection, recovery, and the decision to stop.

An evaluator also needs validation. Use known-good and known-bad models before relying on it. A failed automated check does not alone prove that the geometry is wrong.

## Common pitfalls

Accepted commands do not establish correct shapes. Complete-looking previews do not prove solid validity or internal passages. An exported file does not prove successful readback. Mismatched interfaces and computation components can prevent local success from becoming usable delivery.

Leaving a long task only in an open page also makes continuation fragile. Save intermediate results in reopenable files or projects and record their identifiers and measurements. Confirm recovery support from the actual implementation.

## Reusable acceptance path

Requirements, operation steps, geometry calculation, direct measurement, save and export, and reopen or readback.

Select checks relevant to the task rather than every theoretical geometry property. Distinguish planning, parameters, calculation, transport, saving, and evaluator failures. Preserve the scope of sample success rather than extending it to arbitrary drawings.

This generic tube example contains no real product identifiers, drawings, or measurement values.
