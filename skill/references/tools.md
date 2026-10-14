# 麦当劳 MCP 工具清单速查（references/tools.md）

> 来源：麦当劳中国 MCP 平台（https://mcp.mcd.cn）。协议 Streamable HTTP，认证 `Authorization: Bearer ${MCD_MCP_TOKEN}`。
> 以下为本项目（mcd-surrounded）**实际使用**的工具，参数以官方实时返回为准。

## 游戏核心工具（真实调用）

### `query-meals`
- **作用**：查询当前门店可售餐品列表（分类 + 价格 + 标签）。
- **用途**：构建游戏的「菜单区」与「顾客订单池」——每个顾客订单都来自这份真实数据。
- **关键返回字段**：餐品 id / 名称 / 分类（汉堡、小食、甜品、饮料、早餐…）/ 价格 / 标签。
- **返回示例（字段名以官方实时返回为准，以下为示意）**：
  ```json
  [
    { "id": "m1", "name": "巨无霸", "category": "汉堡", "price": 24.0, "tags": ["经典"] },
    { "id": "m2", "name": "薯条(中)", "category": "小食", "price": 12.0, "tags": [] }
  ]
  ```
- **映射到游戏**：`{ k: id, e: 分类→emoji, n: name }`（emoji 映射见 SKILL.md 第 1 步）。
- **调用方**：WorkBuddy 侧 Skill（对话触发），结果注入游戏前端。

### `campaign-calendar`
- **作用**：营销活动日历查询（含限时联名 / 限定 / 节日活动文案）。
- **用途**：生成「★限定」特殊高分订单；活动名作为订单标签。
- **注意**：仅取「当前有效」活动；无有效活动时跳过，不影响普通玩法。
- **返回示例（示意）**：
  ```json
  [
    { "name": "麦麦周末狂欢", "status": "active", "type": "限时" }
  ]
  ```
- **判定**：仅取 `status === "active"`（或语义等价的「当前有效」）活动名；空数组则跳过活动单。

### `query-meal-detail`（可选）
- **作用**：查询餐品详情（套餐组成、默认选择）。
- **用途**：订单卡片展示单品详情。
- **入参**：餐品 id 或名称。
- **本项目状态**：可选增强，非必需。

## 本项目**不调用**的写操作工具（明确边界）

以下工具一律不调用，确保游戏零交易、零风险：

`create-order`、`auto-bind-coupons`、`available-coupons`、`query-my-coupons`、`query-my-account`、
`mall-points-products`、`mall-product-detail`、`mall-create-order`、`query-nearby-stores`、
`delivery-query-addresses`、`delivery-create-address`、`query-order`、`calculate-price`（下单/领券/积分/地址类）。

> 本游戏只做「读取 + 展示 + 娱乐」，绝不触达交易、领券、积分变动或任何账户状态修改。

## 调用架构示意

```
用户：「开始麦麦包围」
  → Skill 调 query-meals + campaign-calendar（WorkBuddy mcd-mcp 连接器，Bearer 认证）
  → 结果写入 localStorage['mcd_surrounded_menu'] / ['mcd_surrounded_events']
  → 启动 index.html，游戏读取真实数据初始化
```

Token 由 WorkBuddy 连接器管理，**不进入游戏前端、不写入项目文件**。
