# AI CAD 的几何验证与恢复：让可见结果成为可靠产物

## 中文

本文依据通用测试件的独立 STEP 检查、冷恢复回执和一次生成截面放样修复记录。实际产品身份、图纸及产品尺寸已移除；文中数值仅保留通用测试夹具和运行计数。专业原理与项目观察分开说明，不把单件成功推广为任意模型能力。

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

### 实际弯管基准：相同体积也能是错误模型

历史评测先在独立进程中生成并写出、读回一个空心四分之一弯管参考件，再加入四个故意错误的 STEP 样本。验证器检查体积、直径、端面法向、壁厚和孔道，而不是只检查文件可读。

| 通用夹具 | 观测体积（约，mm³） | 指定拒绝依据 |
|---|---:|---|
| 正确参考件 | 493.480220 | 通过 |
| 壁厚错误 | 666.198297 | 壁厚不符 |
| 路径错误 | 542.828242 | 直径/截面位置约束不符 |
| 端口堵塞 | 506.314338 | 贯通不符 |
| 扇区旋转错误 | 493.480220 | 端面法向不符，贯通检查也失败 |

表中每个负样本还可能触发其他检查；列出的是该负样本必须触发的判据。旋转错误件与参考件的体积在所列精度下相同，直接否定了“体积对就形状对”的假设。参考检查合格后，实际页面的固定步骤建模及其原始、往返 STEP 都通过相应独立读取。

“独立”指另一个进程从实际导出字节读取，不是换一个 UI 再看同一内存对象；但读取器仍属同一 OCCT 内核家族，不能称为完全独立算法验证。记录也明确没有连续最大误差证书。这是带负样本校准的有限几何证据，不是任意曲面的严格全域证明，更不是自主 Agent 已完成全部题目。

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

### 曾出现过什么样的自相矛盾

另一项复合模型的诊断中，实测件与目标参考件体积都约为 1,479.050660 mm³；记录的朝向一致，中心点在内、远点在外。布尔比较却把“缺失体积”算成整个参考件体积，而“多余体积”为零。重新处理朝向后，矛盾没有消失。

这个结果不足以证明两件几何完全相同，却足以否定“检查器给了 FAIL，所以应该立即修改产品”的推理。被推翻的是朝向修正即可解决的假设；具体布尔比较故障在该阶段仍未解决。记录将它保留为参考检查阻塞，没有放宽容差凑通过，也没有把它包装成一个已修复的内核缺陷。

从通用几何原理看，B-rep 描述曲面、边界及拓扑关系；有效性检查会检查子形状、方向和边界一致性，但并不理解设计意图。[OCCT 有效性检查说明](https://www.occt3d.com/dev/doc/refman/html/class_b_rep_check___analyzer.html)因此，一个有效实体仍可能孔径错误；反过来，一个布尔差集异常也不能单凭返回体积判定源模型无效。检查器需要已知正负样本，容差必须对应单位、尺度和测量方法。

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

### 已有恢复证据：缺资源与类型错误是两条故障链

一次 300 实体工程的冷恢复失败，定位到两个带哈希名的嵌套 Worker JavaScript 资源在实际站点缺失。处理只补充这两个程序资源，保留站点入口文件哈希不变；随后从真实冷上下文打开原工程，恢复 300 个 Solid，总体积约 57,600 mm³，且 dirty=false。这是局部资源修复与恢复验证，不是整站发布。热页面能用的机制解释是已加载资源可能掩盖依赖缺失；不能据此反推每次热成功都由缓存造成。

后续另一个工程出现不同问题：恢复路径把构造 Face 强制当作 Solid。项目中的面并不一定是缺陷，它可以是后续建模所需的合法中间对象。修复后冷重开原文件，构造面与最终实体均保留，并重新导出 STEP；另有独立读取结果支撑该样本恢复。这里若一律要求所有对象都有正体积、闭合水密，就会把合法构造面误判为失败。

因此恢复核对需要按对象角色判断：实体看有效性、实体数和体积；构造面看面类型、边界和后续依赖；工程整体还要看位置、外观和保存状态。以上两类故障属于同一升级过程中的不同恢复机制，不能把跨会话复测次数当成独立成功案例数量。

## 6. 远端计算与前端版本怎样一起验收

一个新工具可能需要界面、执行服务和计算组件同时更新。仅更新界面或工具说明，用户仍可能调用到旧能力。

核对相互依赖的契约和运行版本，使用同一代表样本走完创建、测量、导出和读取。已经一致的组件不必重复替换；失败位置确定后，修正对应层即可。

## 7. 面向长任务的最小交接

记录当前工程版本、最后一个可验证状态、关键对象、已测量约束、失败步骤和下一步。阶段成果尽量保存为可重开的工程或文件，不依赖“页面别刷新”来维持。

恢复前先读取实际状态，确认上一次操作是否生效，避免重复生成实体。自动检查点、幂等操作和跨会话恢复都是可进一步建设的能力，应按实际版本确认，不能从界面有保存按钮推断。

## 8. 一个可执行的建模验收模板

开始前列单位、输入来源、关键尺寸、允许误差和预期产物。执行后列实际参数、几何测量、保存位置、回读结果及未确认约束。

在开发层面，分别维护简单工具基准、复合建模样本和 AI 自主任务。它们覆盖不同能力。按需要选择检查，保留失败样本和明确范围，比增加一张漂亮截图更能支持后续改进。

## 9. 放样失败：修正截面对应关系，而不是改变设计凑结果

一次复杂主体建模使用同族生成式多边截面，各截面的采样角度和顶点顺序有确定对应关系。原调用仍让内核自动重排、分割截面，构建返回 `ProfilesInconsistent`。如果错误处理接着读取未完成的 Shape，原始状态还可能被后续异常遮蔽。

先把“构建状态”与“读取结果”分开，失败时保留内核状态、实际边数、对应方式和阶段耗时，再比较原参数与重试参数。修复仅在确认截面同族、采样数一致且实际边数符合预期时保留生成式顶点对应；其他轮廓继续走原兼容路径。没有减少截面、改变位置或放宽几何有效性来换成功。

通用原理上，OCCT 的截面兼容处理会计算 wire 起点与方向，并调整边数以避免扭曲。它是有用途的算法，不应全局关闭；当应用已经提供可靠对应关系时，再次推断可能与该关系冲突。[OCCT 放样兼容处理说明](https://occt3d.com/dev/doc/refman/html/class_b_rep_offset_a_p_i___thru_sections.html)本例确认的是这类受约束输入上的调用策略问题，不是所有放样失败都来自内核重排。

实际页面回执多次取得一个实体，保留 B-rep 有效性和正体积检查；原参数与位置逐项一致。两条体积测量路径因计算选项不同给出略不同结果，记录没有把它们当成同一精度的重复证明。局部厚度和点内外测试也有结果，但完整正交视图、阶段保存及整模型门禁没有闭合。

过程中一次测试还被同时升级执行服务中断，这属于环境干扰，不能记作几何失败。后续限定发布验证了简单长方体的真实建模、4 mm 测厚、截图和保存链路，并发布累计修复；没有重跑完整复杂模型。因此准确结论是“该放样路径修复并随包发布，限定链路验收通过”，不是“完整复杂模型已完成”。

该案例把几何判断的几层分开了：**内核是否构建、实体是否有效、尺寸是否满足、模型是否完整、文件是否能恢复**。每层都需要自己的证据，前一层的成功不能替后一层作答。

---

# AI CAD Geometry Validation and Recovery

## English

This article draws on independent STEP checks of generic fixtures, cold-recovery receipts, and a generated-section loft repair. Product identities, drawings, and product dimensions are omitted. Numbers are limited to generic test fixtures and runtime counts. General geometry principles are separated from project observations; one successful part does not establish arbitrary modeling capability.

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

### A real bend benchmark: equal volume can describe the wrong model

The historical evaluator first generated, exported, and read back a hollow quarter-bend reference in a separate process, then tested four deliberately wrong STEP fixtures. It checked volume, diameters, end normals, wall thickness, and passage continuity rather than file readability alone.

| Generic fixture | Approximate volume, mm³ | Required rejection criterion |
|---|---:|---|
| Correct reference | 493.480220 | Accepted |
| Wrong wall | 666.198297 | Wall thickness |
| Wrong path | 542.828242 | Diameter/section-position constraints |
| Plugged end | 506.314338 | Passage continuity |
| Rotated sector | 493.480220 | End normals; continuity also failed |

Each negative could fail additional checks; the table identifies its required rejection criterion. The rotated fixture matched the reference volume at the displayed precision, directly disproving volume as a sufficient shape test. After reference qualification, fixed-step modeling in the actual page and both original and round-trip STEP exports passed corresponding independent reading.

Independent means reading actual exported bytes in another process, rather than viewing the same in-memory object through a different UI. The reader still belonged to the OCCT kernel family, so this was not fully independent algorithmic verification. Records explicitly withheld continuous maximum-error certification. This was bounded geometry evidence calibrated with negative fixtures, not a global proof for arbitrary surfaces or full autonomous-Agent acceptance.

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

### What the contradictory result looked like

For another composite-model diagnostic, the actual and target reference volumes were both approximately 1,479.050660 mm³. Their recorded orientations matched; the center point was inside and a distant point outside. Boolean comparison nevertheless reported the entire target volume as missing and zero extra volume. Reprocessing orientation did not change the contradiction.

This did not prove complete geometric equivalence, but it invalidated the inference that a checker FAIL immediately justified changing the product. The orientation-only remedy had failed; the Boolean-comparison defect remained unresolved at that stage. Records retained a reference-check blocker instead of loosening tolerances or presenting a repaired kernel defect.

As a general principle, B-rep describes surfaces, boundaries, and topology. Validity checks examine subshapes, orientation, and boundary consistency, not design intent. [OCCT validity-check documentation](https://www.occt3d.com/dev/doc/refman/html/class_b_rep_check___analyzer.html) A valid solid can therefore have the wrong bore, while an anomalous Boolean difference does not by its returned volume alone establish invalid source geometry. Checkers need known positive and negative fixtures, and tolerances must match units, scale, and measurement method.

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

### Recovery evidence: missing resources and wrong shape types

A failed cold restore of a 300-solid project was traced to two missing, hash-named nested Worker JavaScript resources at the actual site. Only those resources were added, with the entry-file hash unchanged. Reopening the original project in a genuinely cold context then restored 300 solids, approximately 57,600 mm³ total volume, and dirty=false. This was a local resource repair and recovery check, not a whole-site deployment. Already-loaded resources can explain why a warm page hides a missing dependency; that does not prove caching caused every warm success.

A later project exposed a different defect: recovery forced a construction Face into a Solid. A face can be a legitimate intermediate modeling object. After correction, reopening the original file retained both the construction face and final solid and allowed another STEP export, with independent readback supporting that sample. Requiring every object to have positive volume and a watertight boundary would incorrectly reject the valid construction face.

Recovery checks therefore depend on object role: solids need validity, count, and volume checks; construction faces need type, boundary, and dependency checks; the project also needs position, appearance, and save-state checks. These were different recovery mechanisms within the same upgrade, not separate success cases for every retest across conversations.

## 6. Verify coupled runtime components

A tool may require matching interface, executor, and computation components. Updating only its interface or description can leave users on old behavior.

Check contracts and runtime versions, then run a representative creation, measurement, export, and readback path. Avoid replacing components already confirmed current. Repair the layer established by evidence.

## 7. Preserve a handoff for long tasks

Record project version, the last verified state, key objects, measured constraints, failing step, and next action. Prefer reopenable artifacts over relying on an unrefreshed page.

Read actual state before resuming and determine whether the previous operation took effect. Checkpoints, idempotent actions, and cross-session recovery are separate capabilities to verify in the installed version.

## 8. A practical acceptance record

Before execution, state units, input provenance, critical dimensions, tolerances, and expected artifacts. Afterward, record actual parameters, measurements, save location, readback, and unresolved constraints.

Maintain separate simple tool baselines, compound modeling samples, and autonomous AI tasks. They test different abilities. Relevant checks, preserved failing samples, and explicit scope support improvement more reliably than a screenshot alone.

## 9. Loft Failure: Repair Correspondence Without Changing the Design

A complex-body task used polygonal sections generated from the same profile family, with defined sampling angles and vertex order. The original call still allowed the kernel to reorder and split sections, and construction returned `ProfilesInconsistent`. Reading Shape after an unfinished build could further obscure the original status with a secondary exception.

The investigation separated build status from result access, retaining kernel status, actual edge counts, correspondence mode, and stage timing on failure. It also compared original and retry parameters. The repair preserved generated vertex correspondence only when profile family, sample count, and actual edge count satisfied the expected conditions; other profiles retained the original compatibility path. It did not reduce sections, move geometry, or weaken validity checks to obtain success.

As a general principle, OCCT's compatibility processing computes wire origins and orientations and adjusts edge counts to avoid twisting. It is useful and should not be disabled globally. Where the application already supplies reliable correspondence, another inference can conflict with it. [OCCT loft compatibility documentation](https://occt3d.com/dev/doc/refman/html/class_b_rep_offset_a_p_i___thru_sections.html) The observed defect concerned the calling strategy for constrained inputs, not every possible loft failure.

Actual page receipts repeatedly produced one solid while retaining B-rep validity and positive-volume checks. Original parameters and placement matched field by field. Two volume-measurement paths used different computation options and returned slightly different values; records did not present them as equivalent-precision confirmation. Local thickness and inside/outside tests produced evidence, but complete orthographic views, stage saving, and whole-model gates remained open.

One attempt was interrupted by a concurrent executor upgrade, an environmental disturbance rather than a geometry failure. A later scoped release verified real creation of a simple box, 4 mm thickness, screenshot, and saving, and shipped the accumulated fixes. It did not rerun the full complex model. The defensible conclusion is that the loft path was repaired and shipped with a passing limited execution chain, not that the entire complex model was completed.

This separates **successful construction, valid solids, correct dimensions, complete models, and recoverable files**. Each needs its own evidence; success at one layer cannot answer for the next.
