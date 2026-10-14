# MCP 接入说明（MCP_INTEGRATION.md）

> 项目：救命！我被麦麦包围了！（mcd-surrounded）
> 平台：麦当劳中国 MCP（https://mcp.mcd.cn，Streamable HTTP）
> 认证：`Authorization: Bearer ${MCD_MCP_TOKEN}`（仅占位符，不写入任何文件）

---

## 一、实际使用的 MCP 工具

本项目**真实调用**以下只读工具，作为游戏的「数据引擎」：

| 工具 | 调用场景 | 返回数据如何使用 |
|---|---|---|
| `query-meals` | 用户说「开始麦麦包围」时 | 返回真实在售单品（名称 + 分类 + 价格 + 标签）→ 构成游戏菜单区与顾客订单池 |
| `campaign-calendar` | 同一次初始化 | 返回当前有效营销活动（联名/限时/节日）→ 生成「★限定」高分特殊订单 |
| `query-meal-detail` | 可选，订单详情展示 | 单个单品的套餐组成/默认选择 → 订单卡片详情 |

> 为什么是「引擎」而非「装饰」：游戏的**每一个顾客订单都来自 `query-meals` 实时返回的真实菜单**，活动单来自 `campaign-calendar` 实时活动。没有 MCP，游戏只能退回内置演示池；有 MCP，才是「此刻真能买到的麦麦」。

## 二、调用流程

```
用户：「开始麦麦包围」
   ↓
AI（本 Skill）：
  1. 调 query-meals  → 真实菜单 JSON
  2. 调 campaign-calendar → 当前活动列表
  3. 将菜单写入 localStorage['mcd_surrounded_menu']（活动名写入 mcd_surrounded_events）
  4. 启动 index.html（或注入 ?config=base64）
   ↓
游戏读取真实数据初始化 → 顾客订单全部为真实单品
```

**架构要点**：纯前端网页游戏无法直接持有 Token 调 MCP，因此采用「对话侧拉取 → 注入 `localStorage` → 游戏读取」模式。
- 真实数据由 WorkBuddy 侧 Skill 调用，Token 不进入游戏前端；
- 游戏本体零依赖、可离线运行；
- 无注入时自动回退内置演示池，保证「开箱即玩」。

## 三、业务价值

1. **真实使用 MCP 能力**：菜单、活动数据实时驱动玩法，符合大赛「基于麦当劳 MCP 能力」的参赛导向（`MCP_INTEGRATION.md` 与 `mcp-config.example.json` 为官方必交文件，印证 MCP 为大赛核心能力）。
2. **差异化赛道**：当前榜单以「省钱/营养计算器」「图鉴/盲盒」「动作生存」为主，**「模拟经营 + 多线操作」是空白**，且真实 MCP 当数据底座的做法稀缺。
3. **可玩性 + 传播性**：沙威玛-like 魔性玩法天然适合录屏分享，利于 Star 增长。
4. **零风险合规**：仅读取展示，不点餐、不领券、不查/改账户，Token 不落前端文件。

## 四、明确不调用的工具（边界）

本项目**不调用**任何写操作工具，包括：
`create-order`、`auto-bind-coupons`、`available-coupons`、`query-my-coupons`、`query-my-account`、
`mall-points-products`、`mall-product-detail`、`mall-create-order`、`query-nearby-stores`、
`delivery-query-addresses`、`delivery-create-address`、`query-order`、`calculate-price` 等。

游戏只做「读取 + 展示 + 娱乐」，绝不触达交易、领券、积分变动。
