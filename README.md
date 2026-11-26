# Type Workshop

> 类型设计是**可反复打磨的独立阶段**，不是实现阶段的副产品。
> 这里保存「几天里四个 Agent 的尝试 + 评审 + 融合决策 + 反馈通道」，
> 让类型工作与之后的 Coding Agent 解耦，但不失去联系。

## 1. 一句话定位

- **Workshop 负责**：起草 → 评审 → 融合 → 冻结类型与契约。
- **Coding Agent 负责**：只读类型项目，做实现；遇到类型缺陷写报告，不改契约。
- **两者之间的唯一接口**：类型项目文件夹 + [Contract.md](https://github.com/math4mad/sim-supermarket-types/blob/master/Contract.md)。

## 2. 工作流

```
Draft workshop                 Type workshop                   产出
(多 Agent 并发起草)   ──►   (评审 + 融合 + typedoc)   ──►   类型项目文件夹
     ▲                                │                    （_composer/）
     └──── 换输入 / 换模型 / 换约束 ────┘                     + Contract.md
                                                              + 缺陷报告通道
```

- 可循环：换一组输入或约束，重跑 draft → review → fusion，得到**另一套类型方案**。
- 每轮产物都要能独立编译、能生成文档、能被一个没读过对话的 Agent 直接使用。
- 本轮产物：[review.md](review.md)（评审报告）+ [sim-supermarket-types](https://github.com/math4mad/sim-supermarket-types)（融合后的类型项目）。

## 3. 目录职责

| 路径 | 作用 | 是否给 Coding Agent |
|---|---|---|
| `inputs/` | 共享输入：AGENTS.MD、对话、截图 | ❌ 不给 |
| `drafts/` | 四份草案的索引与一句话评价 | ❌ 不给 |
| `decisions.md` | 融合期的裁决记录（为什么这样定） | ⚠️ 只提炼进 Contract |
| `feedback/` | 缺陷报告协议、模板、周转规则 | ✅ 协议给，过程不给 |
| [Contract.md](https://github.com/math4mad/sim-supermarket-types/blob/master/Contract.md) | 面向实现者的使用条款 | ✅ 必需 |
| [sim-supermarket-types](https://github.com/math4mad/sim-supermarket-types) | **类型项目文件夹**（唯一交付物） | ✅ 只给这个 |
| [review.md](review.md) | 四份实现的完成度评审（round-1 快照） | ❌ 不给 |

> 草案与对话保留在原地，这里只做索引，不复制。
> 原因：复制会带来两份真相与巨大的上下文/体积；索引足够复现。

## 4. 边界（解耦但不隔离）

Coding Agent 的可见范围**只有**：

1. `../_composer/` 全部源码与 `docs/api` 文档；
2. `Contract.md`；
3. `_composer/reports/`（它自己写缺陷报告的地方）。

它看不到：本文件、`WORKFLOW.md`、`decisions.md`、`inputs/`、`drafts/`、`review.md`。

反过来，Workshop 必须消费 Coding Agent 的反馈：

```
实现中发现类型缺陷
  → Coding Agent 写 reports/<date>-<slug>.md（不改 contracts）
  → Workshop 定期 triage
  → 结论写入 decisions.md，修改类型，升版本，更新 contracts.lock.json
  → 回信：受影响包、迁移方式、是否阻塞实现
```

## 5. 一轮 Workshop 的操作清单

1. 准备输入：确认 `inputs/` 指向的 AGENTS.MD 与对话版本。
2. 起草：多 Agent 并发，各自产出可编译的类型方案。
3. 评审：按 `WORKFLOW.md` 的维度打分，产出 `review-<round>.md`。
4. 融合：以胜出方案为基座，按 `decisions.md` 的裁决吸收其余资产。
5. 冻结：`npm run check` 全绿 + `npm run docs` 生成文档 + 更新 `contracts.lock.json`。
6. 交付：只把 `_composer/` + `Contract.md` 交给 Coding Agent。
7. 回收：处理 `_composer/reports/`，进入下一轮。

## 6. 当前状态

- 轮次：`round-1`
- 输入：四份 Agent 草案（见 `drafts/README.md`）
- 评审：[review.md](review.md)（`_1` 90 / `_2` 97 / `_3` 50 / `_4` 59）
- 基座：`Draft-Sim-SuperMarket_2`
- 类型项目：`_composer`（10 包、52 文件、TypeDoc 已生成）
- 交付契约：`Contract.md` v0.1.0

## 7. 发布地址

| 内容 | 地址 |
|---|---|
| 类型项目 | https://github.com/math4mad/sim-supermarket-types |
| 类型文档（GitHub Pages） | https://math4mad.github.io/sim-supermarket-types/ |
| Type Workshop | https://github.com/math4mad/typeworkshop |
| 提交账本 | https://github.com/math4mad/commit-ledger |
