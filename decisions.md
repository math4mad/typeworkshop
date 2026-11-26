# 裁决记录（ADR）

> 只记录「有分歧、且选择会长期影响实现」的决定。
> 每条格式：背景 → 选项 → 决定 → 理由 → 影响面。

## ADR-001 基座选择

- **背景**：四份草案各有优势，直接拼盘会制造同名概念冲突。
- **选项**：`_1` 语义贴合 / `_2` 类型工程最严谨 / 新建骨架。
- **决定**：以 `_2` 为基座，增量吸收 `_1/_3/_4` 的独有资产。
- **理由**：`_2` 已有 branded ID、整数金额、名义 `BusinessPulse`、编译期墙测试；
  这些是「改起来最贵」的地基。`_1` 的锚点/样本/体验层是「加进来最便宜」的上层资产。
- **影响**：`contracts/`、`catalog/`、`commerce/`、`world/` 的结构以 `_2` 为准。

## ADR-002 金额与数量用整数 + brand

- **背景**：四份里三份用 `number` 表示元或件。
- **决定**：`Money` = 整数分，`Quantity` = 整数千分单位，均为 branded 类型。
- **理由**：库存与毛利每天做加减，浮点误差会累积成对不上的账；
  brand 让「把 SkuId 传给 SaleId」在编译期失败。
- **影响**：所有金额/数量字段、Zod 边界（`transform` 回 brand）、报表。

## ADR-003 Onset（滞后）跨墙

- **背景**：`_4` 的 `WorldPulse` 完全没有滞后字段。
- **决定**：`Onset { delayMinutes, rampMinutes }` 进入 `contracts/time.ts`，
  `WorldPulse` 与 `Reading` 都可携带。
- **理由**：冲击不是瞬时传导的；没有 lag，模型永远比现实早反应。
- **影响**：Data World 的 Reading、WorldPulse、模拟步的生效时间。

## ADR-004 单向墙用名义类型而非注释

- **背景**：`_4` 直接把含 `_provenance` 的 `WorldPulse` 带进决策。
- **决定**：`BusinessPulse = Omit<WorldPulse,'_provenance'|'_source'> & 品牌`；
  `SimulationStep.pulse` 只能是 `BusinessPulse`；`EvidenceRef` 无 `news` 分支。
- **理由**：墙一旦只靠约定，某次「顺手 import」就会穿透。
- **影响**：`world/decision.ts`、`dataworld` 的边界适配、`check-walls.mjs`。

## ADR-005 Reading 必须是可辨识联合

- **背景**：`_4` 的 signal 分支漏了 `kind`，联合无法收窄。
- **决定**：signal / noise 两分支都带 `kind`；signal 额外带
  `channel`（cost/demand/both）与 `affectedCategories`。
- **理由**：没有 `channel`，Resolver 无法决定强度落到哪个通道。
- **影响**：`schemas/data-world.ts`、`parity.ts`、样本。

## ADR-006 Magnitude 两模式并存，禁止假预留

- **背景**：`_4` 预留了没有产出方的 `composite` 分支。
- **决定**：只保留 `explicit` 与 `directional`；需要新分支时先写缺陷报告。
- **理由**：预留分支会逼所有消费方写不可能命中的 case。
- **影响**：`dataworld/reading.ts`、`resolver.ts`、`schemas`。

## ADR-007 手写类型为源，Zod 在边界，parity 兜底

- **背景**：类型可读性与运行时校验存在张力。
- **决定**：领域类型手写；Zod 只在边界；`parity.ts` 用编译期断言防漂移。
- **理由**：`z.infer` 会把类型形状藏起来，削弱「类型即 TOC」。
- **影响**：`packages/schemas/`、`tests/type-tests.ts`。

## ADR-008 锚点集与真实样本归 dataworld

- **背景**：`_1` 的锚点/样本是独有资产，但物理位置在原 `types` 目录。
- **决定**：放入 `packages/dataworld/src/{anchors,samples}.ts`。
- **理由**：锚点描述「理解校准」，属于 Data World，不属于超市世界；
  放在 world 会导致 world 依赖 dataworld，直接破墙。
- **影响**：`dataworld` 的导出、`review.md` 的可回收资产表。

## ADR-009 契约锁 + 缺陷报告，而不是自由修改

- **背景**：类型与实现解耦后，实现方仍可能「为了让代码跑通」改契约。
- **决定**：`packages/contracts/` 生成 sha256 锁；实现方只能写
  `_composer/reports/`，不能改契约；Workshop 统一 triage 后升版本。
- **理由**：保障类型设计的稳定性，同时保留反馈通道。
- **影响**：`scripts/check-contracts-lock.mjs`、`npm run check`、`Contract.md`。

## 未决（显式留白）

| 留白 | 说明 | 谁来定 |
|---|---|---|
| lag 的具体数值与传播函数 | 当前只有 `delayMinutes/rampMinutes` 形状 | Workshop round-2 |
| 锚点刻度的版本化方式 | 目前是常量 `strength-v1`，未做迁移策略 | Workshop round-2 |
| KPI 口径变更流程 | `KpiDefinition.version` 已留位，未定审批 | Workshop round-2 |
| `rlBased` 店长策略 | 只留接口，无实现计划 | 实现期提案 |
