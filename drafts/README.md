# 草案索引

> 四份草案保留在 workspace 根目录，本文件只做索引。
> 不复制的原因：避免两份真相、避免把上下文体积带进 workshop。

| 目录 | 综合分 | 定位 | 可回收资产 | 不采用 |
|---|---|---|---|---|
| [`Draft-Sim-SuperMarket_1`](../../Draft-Sim-SuperMarket_1) | 90 | 与对话语义最贴合 | 锚点集、三条真实样本、校准记录、`SimEventEnvelope`、3D/行为/视觉/客服细节 | `Money=number`、裸 `ID`、`Product.stock` |
| [`Draft-Sim-SuperMarket_2`](../../Draft-Sim-SuperMarket_2) | 97 | 类型工程最严谨（**基座**） | branded ID、整数金额、`BusinessPulse`、`type-tests`、`Onset`、`channel/affectedCategories`、采购单/收货单分离 | Zod 未覆盖 commerce/world 事件 |
| [`Draft-Sim-SuperMarket_3`](../../Draft-Sim-SuperMarket_3) | 50 | 通用进销存 CURD | 多门店 `ProductStock`、可读命名 | 缺 `world`/`dataworld`/`sim` 层，整体未吸收 |
| [`Draft-Sim-SuperMarket_4`](../../Draft-Sim-SuperMarket_4) | 59 | 架构概念正确 | `ManagerStrategy`、`ConsumerDecisionEngine`、`SimulationStep`、`exportForML` 思路 | 无进销存实体、`Reading` 联合损坏、漏 `onset`、墙被穿透、`composite` 假预留 |

完整评审见 [`review.md`](../../review.md)。

## 复现

```bash
# 逐份编译（原件）
cd Draft-Sim-SuperMarket_1 && npx tsc --noEmit
cd Draft-Sim-SuperMarket_2/agents/deepseek-v4.1-flash-free-2026-11-26 && npx tsc --noEmit
cd Draft-Sim-SuperMarket_3 && npx tsc --noEmit
cd Draft-Sim-SuperMarket_4 && npx tsc -b
```
