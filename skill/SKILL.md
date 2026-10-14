---
name: mcd-surrounded
version: 1.0.0
description: 麦当劳版「沙威玛」模拟经营小游戏《救命！我被麦麦包围了！》。当用户想玩麦当劳主题小游戏、说「开始麦麦包围」「玩麦麦包围」「麦麦包围」时启用；先调用麦当劳 MCP 拉取真实菜单与活动，注入游戏后再启动。
---

# 救命！我被麦麦包围了！（mcd-surrounded）

一个麦当劳版沙威玛-like 模拟经营小游戏：用户是被订单淹没的店员，限时组装套餐、连击冲分。
本 Skill 的独特价值：**用麦当劳 MCP 的真实菜单/活动数据驱动游戏**，而非装饰。

## 何时启用

- 用户说「开始麦麦包围」「玩麦麦包围」「麦麦包围」「来局麦麦包围」
- 用户想玩麦当劳主题的小游戏、或想看麦当劳 MCP 的真实调用示例

## 工作流程

### 第 1 步：拉取真实数据（真实调用麦当劳 MCP）

调用以下只读工具（认证头由 WorkBuddy 的 mcd-mcp 连接器提供，无需手动处理 Token）：

1. **`query-meals`** —— 获取当前门店可售餐品列表。
   - 将返回的每个餐品映射为 `{k, e, n}`：
     - `k` = 餐品 id 或名称（去重键）
     - `e` = 一个代表 emoji（按分类映射：汉堡🍔 / 小食🍟 / 鸡类🍗 / 饮料🥤 / 沙拉🥗 / 甜品🍦 / 其他🌭🥧🧀🥪）
     - `n` = 餐品名称
   - 这就是游戏的「菜单区 + 订单池」。

2. **`campaign-calendar`** —— 获取当前有效营销活动（联名/限时/节日）。
   - 提取活动名称列表，作为「★限定」特殊订单的来源。
   - 若无有效活动，跳过（游戏照常，只是没有活动单）。

> 可选：`query-meal-detail` 拉单品详情用于订单卡片展示，非必需。

### 第 2 步：注入游戏（两种方式，二选一）

**方式 A — localStorage（推荐：菜单 + 活动都能注）**
在对话侧把数据写入前端可读位置（不要写入任何项目文件、不要写 Token）：

```js
// 菜单：数组，元素为 {k, e, n}
localStorage.setItem('mcd_surrounded_menu', JSON.stringify(menu));
// 活动：活动名字符串数组，无活动则为 []
localStorage.setItem('mcd_surrounded_events', JSON.stringify(events));
```

**方式 B — URL 参数 `?config=`（仅注菜单）**
`?config=` 只能传菜单，**活动关仍须用方式 A 的 localStorage**。编码公式必须与游戏解码端配对（游戏端：`JSON.parse(decodeURIComponent(escape(atob(p))))`）：

```js
// ✅ 正确配对（务必用 unescape 包裹）
const cfg = btoa(unescape(encodeURIComponent(JSON.stringify(menu))));
// 启动：index.html?config=<cfg>
```

> ⚠️ 常见错误：写成 `btoa(encodeURIComponent(...))` 会让游戏解码出乱码/报错，菜单注入失败。必须 `btoa(unescape(encodeURIComponent(...)))`。

**降级处理**：若用户未配置 MCP，或 `query-meals` 调用失败，不要阻塞——直接启动游戏，游戏自动回退内置演示菜单池，仍可正常玩；并告诉用户「当前是演示数据，配置麦当劳 MCP 后会注入真实菜单」。

**验证注入**：注入后请用户刷新页面确认 —— 订单/菜单出现真实餐品名（非演示名）即成功；若仍是演示数据，多半是 `?config=` 编码错误或 localStorage 未写入，重新走方式 A。

### 第 3 步：启动游戏

- 若环境能直接打开本地文件：引导用户用浏览器打开 `index.html`。
- 若需本地预览：在 `mcd-surrounded/` 目录运行 `python -m http.server 8200`，让用户访问 `http://127.0.0.1:8200/index.html`。
- 告诉用户玩法：点菜单装盘 → 交付比对队首顾客 → 连击冲分 → 撑过 75 秒看评级。

## 调用示例（对用户说的话）

> 「我先去麦当劳 MCP 拉一下真实在售菜单和当前活动，然后给你开一局～」
> （调 `query-meals` + `campaign-calendar`，注入后启动）
> 「开好了！现在你被麦麦包围了，订单会从左边排过来，按气泡把单品装盘点交付就行 🍔」

## 合规边界（不可逾越）

- **仅读取**：只调用 `query-meals` / `campaign-calendar` / `query-meal-detail` 等只读工具。
- **绝不**调用任何写操作：`create-order`、`auto-bind-coupons`、`query-my-account`、`mall-*`、`calculate-price`（下单类）等一律不调。
- **Token 安全**：MCP 认证由 WorkBuddy 连接器管理；本 Skill 不把 Token 写入任何项目文件，配置仅用 `${MCD_MCP_TOKEN}` 占位符（见 `../mcp-config.example.json` 或仓库根说明）。
- 游戏输出仅供参考，不构成医疗/营养建议；非麦当劳官方产品。

## 文件结构

```
mcd-surrounded/
├── index.html              # 游戏本体（单文件）
├── README.md
├── DESIGN.md
├── CONTEST_DECLARATION.md
├── MCP_INTEGRATION.md
├── workbuddy.md
└── skill/
    ├── SKILL.md            # 本文件
    └── references/tools.md # 工具参数速查
```
