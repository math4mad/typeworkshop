# Workshop 操作手册

本文件定义「一轮 type workshop 怎么跑、怎么算完成、怎么回到上一轮」。

## 1. 三个可调旋钮

| 旋钮 | 本轮取值 | 换掉会发生什么 |
|---|---|---|
| 输入 | AGENTS.MD + 同一份对话 | 换对话/约束 → 得到不同的类型边界 |
| 模型 | 四个不同 Agent | 换模型 → 得到不同的抽象粒度与命名 |
| 约束 | 纯类型、不做实现 | 加入实现要求 → 类型会更贴近运行时而牺牲可读性 |

> 类型工作的价值在于「可以只换一个旋钮再走一遍」。
> 因此每轮都必须记录这三个旋钮，否则结果不可解释。

## 2. 阶段 A：Draft（并发起草）

- 每个 Agent 拿到**完全相同**的输入，独立产出。
- 交付要求：`tsc --noEmit` 通过；类型目录可读；有模块说明。
- 禁止：参考其他 Agent 的草案。
- 产物留在各自目录，不合并。

## 3. 阶段 B：Review（统一标尺）

评审维度与权重（与 `review.md` 一致）：

| 维度 | 权重 |
|---|---|
| 进销存业务实体覆盖 | 25 |
| 两世界契约与阅读类型 | 25 |
| 模拟层（state/decision/event/reducer） | 15 |
| 体验层与分析层 | 10 |
| Zod 运行时覆盖 | 10 |
| 类型纪律机械化 | 10 |
| 工程结构与可编译 | 5 |

评审必须给出**证据**（文件/行/命令输出），不接受「感觉更完整」。
结论落成 `review-<round>.md`，并给出每份的「可回收资产清单」。

## 4. 阶段 C：Fusion（融合而非拼盘）

1. 选基座：综合分最高，且类型纪律可机械验证者优先。
2. 逐概念裁决：同一概念多份写法冲突时，在 `decisions.md` 留一条 ADR。
3. 吸收独有资产：锚点集、真实样本、体验层细节等。
4. 明确丢弃：列出「不吸收及原因」，防止下一轮重复讨论。

## 5. 阶段 D：Freeze（冻结并交付）

必须同时满足：

```bash
cd _composer
npm run check        # 契约锁 + 墙检查 + tsc
npm run docs         # 生成 HTML 与 Markdown
npm run lock:contracts  # 仅在有意修改 contracts 后执行
```

- 更新 `Contract.md`：版本号、留白清单、已知坑。
- 记录本轮版本：`round-1` → `types v0.1.0`。

## 6. 阶段 E：Handoff（交给实现）

交付物只有三样：

1. `_composer/`（类型项目）
2. `Contract.md`（使用条款）
3. `_composer/reports/` 的空模板（反馈入口）

不给：对话历史、草案、评审过程、决策记录。

## 7. 阶段 F：Feedback（回收）

- Coding Agent 只在 `_composer/reports/` 写报告，**不改** `packages/contracts/`。
- 契约锁 `npm run check:contracts` 会阻止任何对 `contracts/` 的静默修改。
- Workshop 每周（或按阻塞程度）triage：
  - `blocker`：当天处理，必要时发临时补丁版本；
  - `major`：本轮处理；
  - `minor`：进入下一轮草案输入。
- 处理结果回写 `decisions.md` 与 `Contract.md`，并回信给实现方。
