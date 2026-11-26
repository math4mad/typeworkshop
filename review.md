# 四份 Agent 实现的「类型定义完成度」评审

> 评审对象：`Draft-Sim-SuperMarket_1` ~ `_4`
> 输入前提：四个目录共享同一份 `AGENTS.MD` 与同一份 `external/*.html` 对话（四份 HTML 的 MD5 完全一致）
> 评审日期：2026-11-26
> 评审方式：逐文件阅读 + 实际运行 `tsc --noEmit`（`_4` 为 `tsc -b`）+ 仓库自带边界检查脚本

---

## 一、结论速览

| 排名 | 目录 | 综合得分 | 一句话评价 |
|---|---|---|---|
| 1 | `Draft-Sim-SuperMarket_2` | **97 / 100** | 类型体系最完整，且唯一把「单向墙」做成了编译期可强制的名义类型 |
| 2 | `Draft-Sim-SuperMarket_1` | **90 / 100** | 与对话理解最贴合，业务/世界/模拟/体验四层齐全，锚点集是独有加分项 |
| 3 | `Draft-Sim-SuperMarket_4` | **59 / 100** | 架构概念对，但丢了整个进销存实体层，且契约类型有多处硬伤 |
| 4 | `Draft-Sim-SuperMarket_3` | **50 / 100** | 只交了通用进销存 CURD 类型，几乎忽略了对话里最重要的「两个世界 + 滞后」发现 |

四份产物都能通过 TypeScript 编译（`_1`/`_2`/`_3` 的 `tsc --noEmit` 与 `_4` 的 `tsc -b` 均返回 0），所以下面讨论的不是「能不能编译」，而是**类型是否把项目真正需要的概念表达完整、表达正确**。

---

## 二、评分口径

| 维度 | 权重 | 说明 |
|---|---|---|
| 进销存业务实体覆盖 | 25 | 商品/供应商/采购/收货/销售/库存流水/批次/盘点/会员/门店/报表 |
| 两世界契约与阅读类型 | 25 | `WorldPulse`、`Reading`（signal/noise）、`Magnitude`、Resolver、双模式、**滞后 onset** |
| 模拟层 | 15 | `WorldState` / `Decision` / `SimEvent` / reducer / 事件溯源 |
| 体验层与分析层 | 10 | 3D 场景、购物行为、视觉秤、客服、KPI/购物篮/预测、代数展示 |
| Zod 运行时覆盖 | 10 | schema 覆盖范围、与类型的一致性 |
| 类型纪律机械化 | 10 | branded ID、单向墙的编译期/脚本强制、类型-schema parity、`@ts-expect-error` 测试 |
| 工程结构与可编译 | 5 | 目录分层、多包拆分、tsconfig 严格度 |

---

## 三、逐份详评

### 1) Draft-Sim-SuperMarket_2 —— 97 分（最佳）

**规模**：`agents/deepseek-v4.1-flash-free-2026-11-26/src` 共 56 个文件、约 3750 行，12 个模块（contracts / catalog / commerce / world / dataworld / agents / analytics / math / view / vision / api / schemas）。

**完成度亮点**

- **业务实体最细**：`catalog/product.ts` 刻意把价格、库存从 `Product` 中拆出；`commerce/stock.ts` 用 `StockLedgerEntry` 表达「库存 = 流水之和」，并给出 `StockSnapshot` 的 `onOrder` / `onShelf`；`commerce/purchase.ts` 把「采购单（意图）」与「收货单（事实）」拆开，天然支持部分收货；`commerce/pricing.ts` 有 `PricePoint` / `PriceBook` / `SupplierQuote`；`commerce/member.ts` 甚至有只追加的 `LoyaltyEntry`。
- **契约层最强**：`contracts/world-pulse.ts` 用 `unique symbol` 给 `BusinessPulse` 加名义品牌，让「原始 `WorldPulse` 直接进超市世界」在编译期就写不出来；`src/type-tests.ts` 用 6 条 `@ts-expect-error` 把墙、审计字段、证据不含新闻、品牌 ID 等都变成可回归的断言。
- **对话洞察落地最完整**：`Onset` 被提升为跨墙一等公民（`contracts/time.ts`），`Reading` 同时保留 `signal` / `noise`、`confidence` 与 `strength`、`magnitudeKind` 双模式，并额外补了 `channel: 'cost' | 'demand' | 'both'` 与 `affectedCategories` —— 这是四份里唯一把「强度落到哪个通道」想清楚的。
- **工程质量**：ID 全部 branded（`SkuId` 不能赋给 `SaleId`），`Money` 用整数分、`Quantity` 用整数千分单位，规避浮点误差；12 个包的依赖方向由 `scripts/check-walls.mjs` 机械校验（本次运行：131 条 import，0 处越界）。

**不足**

- Zod 只放在边界（`schemas` 5 个文件：common / world-pulse / reading / decision / api），commerce、world 的事件与状态没有运行时 schema。这是刻意的取舍，但离「可运行的进销存」还差一层校验。
- 类型层已相当完整，但 `analytics` / `view` / `agents` 更偏「类型契约 + 少量纯函数」，尚未形成可跑切片（README 也自认是类型层草案）。

---

### 2) Draft-Sim-SuperMarket_1 —— 90 分（理解最贴）

**规模**：`src/types` + `src/schemas` 共约 2210 行，`src/types` 分 `supermarket` / `experience` / `world` / `sim` 四层，另有 `checks/` 与 `scripts/`。

**完成度亮点**

- **进销存覆盖齐全**：商品与分类、供应商/客户、采购（含部分收货与退货）、销售（含支付记录、退货）、库存流水（`StockLedger` 不可变）、批次、盘点、预警、用户/门店/操作日志、报表，全部齐备。
- **对话的两条核心发现都落了地**：`world/pulse.ts` 的 `WorldPulse.onset { delay, ramp }` 与 `Reading.onset`；`world/anchors.ts` 的 `STRENGTH_ANCHORS` + `CalibrationRecord`（「偏差不是错误」的复核回路）是四份中独一无二的资产，`world/samples.ts` 还把三条真实新闻写成了回归样例。
- **纪律可执行**：`toWorldPulseView()` 在类型层把 `_` 字段投影掉；`scripts/check-boundaries.mjs` 本次运行通过（32 文件，无越界）；`checks/parity.ts` 用 `Equal<>` 编译期断言 Zod 与手写类型同形。
- **体验层最丰富**：`experience/scene.ts`（货架/货位/拣货路径/动线热力）、`behavior.ts`（购物车事件流、漏斗、弃购、关联规则）、`vision.ts`（视觉秤识别、错例回流、MLOps 指标）、`agent.ts`（客服工具集与思考轨迹）。这正好对应 `AGENTS.MD` notice 里的 bullet 列表。

**不足**

- 基元偏弱：`ID = string`、`Money = number`（元，两位小数）没有 branded 化，跨实体误用编译器抓不住；`Product.stock` 作为 `@deprecated` 字段仍保留在商品上，与「商品不持有库存」的纪律相冲突。
- Zod 只覆盖 common / product / purchase / sale / reading（5 个文件），库存、系统、体验层没有运行时 schema。
- `world/pulse.ts` 里 `export type Category = string` 与 `supermarket/product.ts` 的 `Category` interface 同名，虽在同构模块内不冲突，但作为「给学习者读的 TOC」会造成概念混淆。

---

### 3) Draft-Sim-SuperMarket_4 —— 59 分（架构对、实体缺、契约有硬伤）

**规模**：`packages/types/src` + `packages/schemas/src` 约 926 行，8 个 workspace 包；`tsc -b` 通过。

**完成度亮点**

- 目录拆分最「工程化」：`types` / `schemas` / `data-world` / `simulation` / `three-view` / `eshop` / `dashboard` / `ml` 的 monorepo 骨架清晰，符合 instruction 6。
- 概念性类型有独立文件：`data-world.ts` 与 `supermarket-world.ts` 分居两世界，`WorldState` 里写了「不含 weather / geopolitical」的注释纪律。
- Zod 与类型成对出现（`data-world.zod.ts` / `supermarket-world.zod.ts`），并有 `validateWorldState` 等入口。

**严重缺口（影响分数的主要项）**

1. **整个进销存实体层缺失**。全仓库检索不到 `Product` / `Supplier` / `PurchaseOrder` / `SaleOrder` / `StockLedger` / `StockBatch` / `InventoryCheck` / `Member` / `ShelfSlot`（0 命中）。`WorldState.inventory` 只是内联的 `{ quantity, unitCost, retailPrice, category }`，`transactions` 也只是内联匿名结构。鉴于对话文件的标题就是「超市进销存系统的 TypeScript 类型」，这属于基准要求的缺失，而不是可选扩展。
2. **`Reading` 的可辨识联合是坏的**。`data-world.ts` 里 signal 分支没有 `kind: 'signal'`，只有 noise 分支有 `kind`，导致 `z.infer` 也无法形成真正的 discriminated union；`data-world.zod.ts` 的 `ValidReadingSchema` 同样缺 `kind`，只能靠 `'kind' in reading` 的近似守卫（`packages/data-world/src/pipeline.ts:51`）区分。
3. **丢掉了对话最重要的 `onset`（滞后）**。`WorldPulse` 没有 `onset`，四个包全仓库也没有任何 `onset` 定义 —— 等于没有吸收「运价涨 → 报价涨 → 成本涨 → 售价涨 → 顾客反应」这条链条的结论。
4. **单向墙被穿透**。`Decision.pulseRef: WorldPulse` 与 `WorldState.lastPulse: WorldPulse` 直接把含 `_provenance` 的原始脉冲带进模拟层，没有 `BusinessPulse` / `toBusinessPulse` 之类的栅栏类型；`supermarket-world.ts` 还从 `./data-world` import 了 `SimTime`、`Category`，越过了「只允许 `WorldPulse` 越墙」的自我约定。
5. **违反「Union 分支」原则**。`Magnitude` 预留了 `composite` 分支（注释写「Reserved」），而 `pipeline.ts` 已经要处理它 —— 正是对话里明确反对的「假预留」。
6. **模块只有壳**。`eshop`、`dashboard`、`three-view`、`ml` 四个包的 `src/index.ts` 都只有 1 行，尚未定义任何类型。

---

### 4) Draft-Sim-SuperMarket_3 —— 50 分（范围最窄）

**规模**：`agents/lola` 约 648 行，8 个类型文件 + 3 个 Zod schema 文件；`tsc --noEmit` 通过。

**完成度亮点**

- 进销存 CURD 类型本身是干净、可用的：`Product` / `ProductStock`（多门店维度）、`PurchaseOrder`（含部分收货状态机）、`SaleOrder`（含支付记录与快照）、`StockLedger`（只追加）、`StockBatch`、`InventoryCheck`、`Supplier` / `Customer`、`User` / `Store`、`SalesSummary`。
- `agent-idea.md` 的框架调研与 Mermaid 拆分图是四份里写得较清楚的（Mastra 首选 / Vercel AI SDK 备选）。

**主要不足**

- **几乎忽略了对话的主体内容**。没有 `world` 层、没有 `WorldPulse`、没有 `Reading` / `Magnitude` / `Resolver`、没有 `onset`、没有 `Data World` 与超市世界的分离，也没有模拟层（`WorldState` / `Decision` / `SimEvent` / reducer）。这些不是「可选漂亮话」，而是 `AGENTS.MD` instruction 2 与对话里反复固化的核心。
- **没有体验层/分析层类型**：3D 场景、购物行为事件流、视觉秤、客服 Agent、购物篮/时间序列分析均缺失；`report.ts` 只有 2 个类型。
- **Zod 覆盖最薄**：仅 product / purchase / sale 三个 schema，库存流水、盘点、系统、报表全无运行时应答；且类型与 schema 是两份手写定义，没有像 `_1`/`_2` 那样做一致性断言。
- 没有 `README`、没有边界检查脚本，工程约束只停留在文档口头约定。

---

## 四、关键维度对照

| 维度 | `_1` | `_2` | `_3` | `_4` |
|---|---|---|---|---|
| 商品/分类 | ✅ | ✅（价格库存拆出） | ✅ | ❌ |
| 供应商/采购/部分收货 | ✅ | ✅（采购单 vs 收货单） | ✅ | ❌ |
| 销售/支付/退货 | ✅ | ✅（+ 促销/小票） | ✅ | ❌ |
| 库存流水/批次/盘点/预警 | ✅ | ✅（+ onShelf/onOrder） | ✅ | ❌（仅内联 inventory） |
| 会员/门店/用户/报表 | ✅ | ✅（+ 价格历史/报价） | ✅ | ❌ |
| `WorldPulse` 契约 | ✅ | ✅（名义品牌 `BusinessPulse`） | ❌ | ⚠️（有类型但漏 `onset`） |
| `Reading` signal/noise | ✅ | ✅（+ channel/affectedCategories） | ❌ | ⚠️（联合不可辨识） |
| `Magnitude` 双模式 | ✅ | ✅ | ❌ | ✅（但多留了 composite） |
| 滞后 `onset` | ✅ | ✅ | ❌ | ❌ |
| 双模式 frozen/live | ✅ | ✅ | ❌ | ✅ |
| `WorldState/Decision/Event/reducer` | ✅ | ✅ | ❌ | ⚠️（有 state/decision，事件弱） |
| 3D / 行为 / 视觉 / 客服 | ✅ | ✅ | ❌ | ❌ |
| 分析/KPI/购物篮/预测 | ⚠️（报表层） | ✅ | ❌ | ❌ |
| Zod 覆盖 | ⚠️ 5 文件 | ⚠️ 5 文件（边界策略） | ⚠️ 3 文件 | ✅ 覆盖其全部自身类型 |
| Branded ID / 整数金额 | ❌ | ✅ | ❌ | ❌ |
| 墙的机械检查 | ✅ 脚本 | ✅ 脚本 + 类型测试 | ❌ | ❌ |
| 类型-schema 一致性断言 | ✅ | ✅ | ❌ | ❌ |
| 编译通过 | ✅ | ✅ | ✅ | ✅ |

---

## 五、总体评价

1. **第一梯队（`_1`、`_2`）已经把「类型是宪法」这件事做成了可执行的东西**：不仅能编译，还有脚本/编译期断言去阻止越墙、读审计字段、混用 ID。两者的差异主要在取舍：`_2` 在类型工程的严谨度（branded 类型、整数金额、名义品牌）与模块完整度上更强；`_1` 在「与对话语义的贴合度」（锚点集、真实样本、体验层细节）上更胜，且更适合作为教学 TOC 来读。

2. **第二梯队的问题不是「写得少」，而是「写偏了」**：
   - `_4` 搭了正确的世界观（两个世界、monorepo），却把最基础的进销存实体层整个漏掉，且 `Reading` 联合、`onset`、单向墙三处出现与对话结论相悖的实现，属于「架构正确、地基缺失」。
   - `_3` 交了一份合格的通用进销存类型文件，但没有回答这份对话真正提出的问题（世界如何被外部数据驱动、冲击如何滞后传导、判断如何被校准），属于「范围偏离」。

3. **共同短板**：四份都还停留在类型层，没有一份能跑通第一个切片（扫描 → 称重 → 销售 → 扣库存 → 流水 → 看板）。若按 `AGENTS.MD` 的「五天反复思考」定位，这是可以接受的；但若进入 12 月开工，`_2` 的类型骨架最适合直接接实现，`_1` 的锚点/样本最适合接数据 Agent，`_4` 与 `_3` 需要先补齐实体层与契约层再谈实现。

---

## 六、建议（按优先级）

**给 `_2`（作为实现基线时）**
- 为 `commerce` 与 `world` 事件补边界 Zod，或在 `schemas` 中增加 parity 断言，避免两套定义长期漂移。
- 把 `analytics` / `view` 从「类型 + 注释」推进到可运行的纯函数切片，验证「同一批类型多视图」的立论。

**给 `_1`**
- 将 `Money` 改为整数分、`ID` 改为 branded，至少为 `SkuId` / `OrderId` 建立区分，否则跨实体误用仍是编译器盲区。
- 把 `Product.stock` 从主类型中移除或迁到 `ProductStock`，与「商品不持有库存」纪律对齐；同时消解 `Category` 同名二义。
- 给库存/盘点/系统类型补 Zod。

**给 `_4`**
- 补齐进销存实体层（Product / Supplier / PurchaseOrder / GoodsReceipt / SaleOrder / StockLedger / StockBatch / InventoryCheck / Member / ShelfSlot）。
- 修复 `Reading`：signal 分支补 `kind: 'signal'`；`WorldPulse` 补 `onset`；删除 `Magnitude.composite`。
- 引入 `BusinessPulse`（不含 `_` 字段）替换 `Decision.pulseRef` / `WorldState.lastPulse`，把墙补回来。

**给 `_3`**
- 若目标仍是完整项目，至少补齐 `world`（WorldPulse / Reading / Magnitude / onset）、`sim`（State / Decision / Event / reducer）与体验层骨架，否则应明确声明本产物仅覆盖「进销存子域」。
