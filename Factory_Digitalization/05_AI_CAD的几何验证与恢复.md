# AI CAD 的几何验证与恢复：让可见结果成为可靠产物

## 中文

本文使用通用管件和模型工程场景，保留建模、评测与恢复中的问题机制，不包含实际产品、图纸、软件实现或内部测量数据。方法适用于不同建模工具与自动化接口。

## 1. 一次“成功”可能只发生在其中一层

建模助手选择了操作，不代表参数正确；工具接受参数，不代表实体符合设计；页面出现模型，不代表文件保存完整。一次任务至少经过：

需求约束 → 操作规划 → 工具执行 → 几何结果 → 页面显示 → 测量 → 保存导出 → 重开回读。

这些层之间应有可以核对的标识和结果。任务状态用于描述执行过程，几何和文件用于判断最终产物。

## 2. 案例一：空心管看起来正确，怎样确认孔真的存在

### 先明确任务

假设任务是沿一条路径建立带内孔的管件。开始前说明单位、外轮廓、内轮廓、路径、关键壁厚、连接要求和允许误差。

如果图纸没有给出某个约束，应标为待确认或明确采用的假设，不应让模型把视觉猜测伪装成确定尺寸。

### 用简单基准验证工具路径

可以先用直管验证带孔截面是否正确传入。对理想直圆管，在满足几何条件时，理论体积是 π(R²−r²)L。这个基准有助于发现截面被错误填实、半径与直径混用、单位不一致等问题。

这只是直管的简化检查。它不能不加条件地用于任意曲线、交叠或复杂连接，也不能替代实体有效性和边界检查。

### 再进入弯曲与复杂结构

对曲线路径重点检查内孔是否贯通、壁厚是否异常、是否产生意外实体或自交。视觉预览可以发现明显错误，但不能证明内部结构。

当外观接近时，比较实体数量、关键尺寸、体积及截面，可以把“看着像”变成更明确的判断。体积一致也不是唯一标准，不同形状可能体积相同，因此需要结合任务约束。

### AI 怎样参与

AI 可以读取工具说明、选择操作顺序、构造参数、解释错误，并将测量值与要求对照。人确定设计意图、制造约束和允许误差。

执行者应记录实际使用的参数及对象标识。修正后重新测量，不能只因为第二次命令没有报错就宣布几何正确。

## 3. 案例二：固定步骤通过，为什么自主建模评测仍失败

固定步骤已经规定操作与参数，主要验证工具和计算路径。自主任务则要求模型从需求出发决定步骤、识别错误、选择补救并判断结束。

因此，两类结果应分别报告。固定步骤样本通过，不能推广成模型能独立解决整个题库；自主任务传输中断，也不能直接断言几何算法失败。

较有用的评测记录包含任务要求、模型实际调用、命令回执、最终对象、测量结果、导出回读以及中断位置。把失败定位到规划、调用、计算、传输或保存层，比一个笼统 failed 更能指导修复。

如果一次错误发生在工具选择之前，先修任务链；如果相同参数在人工固定步骤下也失败，再重点调查工具实现或几何限制。

## 4. 案例三：检查器连标准样本都判错

自动验收可以提高重复检查效率，但验收程序也是软件。一个检查器若不能正确识别已知样本，就暂时不能作为产品裁判。

检查时可以按顺序处理：

1. 确认输入模型和文件版本是否一致；
2. 确认单位、容差、坐标与对象选择；
3. 用最简单的已知正确和已知错误样本验证判断条件；
4. 比较独立测量，确认是模型异常还是检查逻辑异常；
5. 修正检查器后重跑之前受影响的样本。

必要时使用独立读取或测量路径，减少生产代码和检查器共享同一个错误假设的机会。不能为了让指标变绿，直接放宽容差或跳过失败条件。

若检查器尚未修好，应明确其结论不可用，同时继续不依赖它的其他工作。

## 5. 案例四：当前页面能用，冷重开却失败

### 为什么同一会话的成功会掩盖问题

当前页面可能已经加载了旧资源、临时数据或后台组件。工程保存后立即在同一页面操作，不足以证明全新会话可以恢复。

一个典型问题是安装包缺少后台计算组件。热会话依赖已加载内容还能继续，冷重开才暴露缺失。这里的修复重点是打包与恢复路径，而不一定是几何算法。

### 怎样建立最小复现

先保存一个小工程，记录对象数量与关键测量。关闭原上下文或使用独立新会话，再从持久化文件打开，执行必要测量和一次相关操作。

检查工程文件是否包含所需内容，外部资源是否可访问，运行包是否带齐依赖，恢复过程是否报错。保存回执应能关联到文件或版本，而不只是显示一个提示。

### 区分几种保存结果

| 保存内容 | 通常可以支持什么 | 需要说明的限制 |
|---|---|---|
| 最终几何 | 查看、测量、部分后续加工 | 可能没有完整参数历史 |
| 参数与操作历史 | 重算、编辑参数 | 依赖工具版本及重放能力 |
| 显示数据 | 快速预览 | 不一定足以恢复可编辑实体 |
| 引用与外部资源 | 连接原始资料 | 依赖资源可用性与访问范围 |

实际格式可以组合这些内容。关键是准确描述它保存了什么、恢复依赖什么。采用回退保存时，也要说明损失了哪些编辑能力，不能把能打开等同于完整恢复。

## 6. 远端计算与前端版本怎样一起验收

一个新工具可能需要界面、执行服务和计算组件同时更新。仅更新界面或工具说明，用户仍可能调用到旧能力。

核对相互依赖的契约和运行版本，使用同一代表样本走完创建、测量、导出和读取。已经一致的组件不必重复替换；失败位置确定后，修正对应层即可。

## 7. 面向长任务的最小交接

记录当前工程版本、最后一个可验证状态、关键对象、已测量约束、失败步骤和下一步。阶段成果尽量保存为可重开的工程或文件，不依赖“页面别刷新”来维持。

恢复前先读取实际状态，确认上一次操作是否生效，避免重复生成实体。自动检查点、幂等操作和跨会话恢复都是可进一步建设的能力，应按实际版本确认，不能从界面有保存按钮推断。

## 8. 一个可执行的建模验收模板

开始前列单位、输入来源、关键尺寸、允许误差和预期产物。执行后列实际参数、几何测量、保存位置、回读结果及未确认约束。

在开发层面，分别维护简单工具基准、复合建模样本和 AI 自主任务。它们覆盖不同能力。按需要选择检查，保留失败样本和明确范围，比增加一张漂亮截图更能支持后续改进。

---

# AI CAD Geometry Validation and Recovery

## English

These generic tube and project scenarios preserve modeling, evaluation, and recovery mechanisms without real products, drawings, implementation details, or internal measurements. They apply across modeling tools and automation interfaces.

## 1. Success can occur at only one layer

A selected operation may use incorrect parameters. Accepted parameters may create the wrong solid. A displayed shape may not be fully saved.

A task passes through requirements, planning, execution, geometry, display, measurement, saving and export, and reopening. Maintain identifiable results between these stages. Task status describes execution; geometry and files establish the deliverable.

## 2. Case one: checking a hollow tube

### Define the task

State units, outer and inner profiles, path, wall requirements, connections, and tolerances. Missing constraints should remain questions or explicit assumptions rather than invented dimensions.

### Validate a simple baseline

A straight tube can confirm that the hollow section reaches the tool correctly. Under ideal straight circular-tube assumptions, volume is π(R²−r²)L. This can reveal a filled section, radius/diameter confusion, or unit errors.

It is not a universal formula for arbitrary curves, overlaps, or connections, and it does not replace solid validity checks.

### Extend to curved structures

Check passage continuity, abnormal thickness, unintended solids, and self-intersections. Previews expose visible defects but do not establish internal structure.

Compare solid count, dimensions, volume, and relevant sections. Equal volume alone is insufficient because different shapes can share it. Check properties against the actual design constraints.

### AI's role

AI can read tool documentation, choose operations, construct parameters, interpret errors, and compare measurements with requirements. People define design intent, manufacturing constraints, and tolerances.

Record the parameters actually executed and resulting object identities. Measure again after correction rather than treating absence of an error as proof of correctness.

## 3. Case two: scripted success but failed autonomous evaluation

Fixed steps prescribe operations and parameters, primarily checking tools and computation. Autonomous modeling also checks planning, error recognition, recovery, and the decision to finish.

Report them separately. Successful scripted samples do not prove autonomous completion of an entire task set. An interrupted autonomous request does not by itself establish a geometry defect.

Useful evidence includes requirements, actual model calls, receipts, final objects, measurements, export readback, and the interruption point. Locate failure in planning, invocation, calculation, transport, or persistence.

If failure occurs before tool selection, repair the execution chain. If the same parameters also fail under fixed steps, investigate tool implementation or geometric limitations.

## 4. Case three: an evaluator rejects a known sample

Evaluation software can be wrong. Until it recognizes known samples correctly, it is not a reliable product judge.

Check the model and file version, units, tolerances, coordinates, and selected objects. Test simple known-good and known-bad inputs, compare an independent measurement, then rerun affected samples after repairing the evaluator.

Independent reading or measurement can reduce shared assumptions between production code and its checker. Do not loosen tolerances or remove failure conditions merely to obtain passing results.

State when evaluation is unavailable and continue unrelated work that does not depend on it.

## 5. Case four: a live page works but a cold reopen fails

### Why live success can hide a defect

An open page may retain resources, temporary data, or computation components. Continuing immediately after saving does not prove recovery in a fresh session.

A missing background component in an installation package is one possible cause. A warm session continues using loaded resources; a cold reopen reveals the omission. The defect may lie in packaging rather than geometry.

### Build a minimal reproduction

Save a small project and record object count and critical measurements. Close the old context or use an independent session, open the persisted file, measure again, and perform a related operation.

Check file contents, external resources, packaged dependencies, and recovery errors. A save receipt should identify a file or version, not merely display a success message.

### Describe what was saved

| Content | Typical use | Limitation to disclose |
|---|---|---|
| Final geometry | Viewing, measurement, some further operations | Full parameter history may be absent |
| Parameters and history | Recalculation and editing | Depends on compatible tools and replay |
| Display data | Fast previews | May not reconstruct editable solids |
| References and external resources | Linking source material | Depends on resource availability and access |

Formats may combine these approaches. Explain saved content and recovery dependencies accurately. A fallback save should disclose lost editing capabilities; opening a file is not necessarily complete restoration.

## 6. Verify coupled runtime components

A tool may require matching interface, executor, and computation components. Updating only its interface or description can leave users on old behavior.

Check contracts and runtime versions, then run a representative creation, measurement, export, and readback path. Avoid replacing components already confirmed current. Repair the layer established by evidence.

## 7. Preserve a handoff for long tasks

Record project version, the last verified state, key objects, measured constraints, failing step, and next action. Prefer reopenable artifacts over relying on an unrefreshed page.

Read actual state before resuming and determine whether the previous operation took effect. Checkpoints, idempotent actions, and cross-session recovery are separate capabilities to verify in the installed version.

## 8. A practical acceptance record

Before execution, state units, input provenance, critical dimensions, tolerances, and expected artifacts. Afterward, record actual parameters, measurements, save location, readback, and unresolved constraints.

Maintain separate simple tool baselines, compound modeling samples, and autonomous AI tasks. They test different abilities. Relevant checks, preserved failing samples, and explicit scope support improvement more reliably than a screenshot alone.
