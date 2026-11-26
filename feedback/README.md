# 反馈通道

> 这是「解耦但不隔离」的关键：实现阶段可以质疑类型，但不能私自改类型。

## 1. 谁写、写在哪

- **写的人**：Coding Agent（或任何实现者）。
- **写在哪**：`_composer/reports/`。
- **禁止**：直接修改 `_composer/packages/contracts/`。
  `npm run check:contracts` 会用 sha256 锁检测任何静默修改。

## 2. 什么算「类型缺陷」

| 类型 | 例子 | 严重度 |
|---|---|---|
| 缺少字段 | 实现订单折扣时发现 `SaleOrder` 没有 `couponCode` | major |
| 类型过窄 | 门店需要午休，`opensAtMinute/closesAtMinute` 表达不了 | major |
| 联合分支错误 | 某个 `kind` 永远不可能出现，或缺少必要分支 | blocker |
| 量纲冲突 | 需要把 `Grams` 当 `Quantity` 用 | blocker |
| 留白被误用 | 实现时被迫自行决定 `lag` 数值 | major |
| 文档与类型不符 | `Contract.md` 说的和实际类型不一致 | minor |

## 3. 报告命名与内容

```
_composer/reports/YYYYMMDD-<slug>.md
例：20261203-sale-coupon-field.md
```

必须包含：现象、最小复现、临时处理、建议、影响面、严重度。
模板见 [`TEMPLATE.md`](./TEMPLATE.md)。

## 4. Workshop 的处理节奏

| 严重度 | 响应 | 处理方式 |
|---|---|---|
| `blocker` | 当天 | 发临时补丁版本（`v0.1.1`），解除阻塞 |
| `major` | 本轮 | 修改类型 + 更新锁 + 更新 Contract |
| `minor` | 下一轮 | 进入 draft 输入，不阻塞实现 |

每条报告最终要有三种结局之一：`accepted` / `rejected` / `deferred`，
并回写到 `decisions.md`。**不允许报告石沉大海**。

## 5. 版本与锁

- 类型项目版本在 `_composer/package.json`。
- 有意修改 `contracts/` 后，必须执行：

```bash
cd _composer
npm run lock:contracts
```

- `npm run check` 包含契约锁校验；锁不一致会失败并提示：
  「请写缺陷报告，或在 workshop 中显式更新锁」。

## 6. 回信模板（Workshop → 实现方）

```markdown
## 反馈处理结果

- 报告：reports/20261203-sale-coupon-field.md
- 结论：accepted
- 类型版本：v0.1.1
- 变更：sales/SaleOrder 增加 couponCode?: string；新增 schemas 校验
- 迁移：不需要改调用；旧数据该字段为 undefined
- 校验：npm run check 全绿
```
