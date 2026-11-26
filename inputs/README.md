# 输入清单

> Workshop 的唯一输入源。每轮开始前核对版本，避免「换了输入却不知道」。

| 输入 | 路径 | 作用 | 备注 |
|---|---|---|---|
| 任务约束 | [`AGENTS.MD`](../../Draft-Sim-SuperMarket_1/AGENTS.MD) | 定义边界、纪律、交付要求 | 四份草案共享同一份 |
| 对话原文 | [`Draft-Sim-SuperMarket_1/external/`](../../Draft-Sim-SuperMarket_1/external) | 需求与关键发现（滞后、双世界、锚点） | 四份 HTML 的 MD5 一致 |
| 流程草图 | `../inputs/screenshot_1945.jpg` | Draft → Type → 交付 → 实施 的流程 | 详见 `README.md` §2 |
| 评审示例 | [`review.md`](../review.md) | 统一评审标尺的首次应用 | round-1 |

## 一致性校验

```bash
md5 Draft-Sim-SuperMarket_1/external/*.html \
    Draft-Sim-SuperMarket_2/external/*.html \
    Draft-Sim-SuperMarket_3/external/*.html \
    Draft-Sim-SuperMarket_4/external/*.html
```

四份必须一致，否则本轮不可比。
