# WorkBuddy 开发对话上下文（workbuddy.md）

> 本文件用于核验本项目是否使用 WorkBuddy 开发，以确认是否符合大赛「WorkBuddy 联动奖励」条件。
> 项目名：麦当劳程序员节参赛作品《救命！我被麦麦包围了！》（mcd-surrounded 模拟经营 Skill）｜基于麦当劳中国 MCP 真实数据驱动，用 WorkBuddy 开发

---

## 一、开发环境

- **工具**：WorkBuddy（腾讯）Agent 模式
- **运行时**：托管版 Python 3.13 / Node 22（由 WorkBuddy 提供）
- **协作方式**：用户与 AI 在 WorkBuddy 对话中完成「活动调研 → 竞品试玩 → 方向决策 → 设计文档 → 实现」全流程

## 二、开发过程关键节点（对话脉络）

1. **活动调研**：用户得知「WorkBuddy × 麦当劳」合作，AI 检索并确认这是「麦当劳程序员节创意开发大赛」——基于麦当劳 MCP 开发 Skill，用 WorkBuddy 有专项积分。
2. **竞品试玩（WorkBuddy 内拉取并本地托管）**：
   - 试玩 `mcd-fries-architect`（薯条建筑师，3D 物理搭建，零 MCP 依赖）
   - 试玩 `麦麦消消乐 McMatch`（纯前端三消，emoji 渲染，演示版零 MCP）
   - 拉取榜单确认赛道拥挤度（省钱/营养/图鉴已红海）
3. **方向决策（多轮 AskUserQuestion）**：
   - 用户最初想做盲盒扭蛋机 → 发现盲盒/图鉴已有人做
   - 用户受竞品启发想做「小游戏」→ 试玩沙威玛类游戏后，确定做**麦当劳版沙威玛模拟经营**
   - 用户明确：**不要恋爱元素、暂不加公益、视觉走 emoji 极简**
4. **设计文档**：先聊透 DESIGN.md（产品定位 / 核心循环 / 订单 / 难度 / 计分 / MCP 映射 / 合规），用户确认后才落盘。
5. **实现**：WorkBuddy 生成单文件 `index.html`（emoji 游戏）+ 全套参赛文档（README / CONTEST_DECLARATION / MCP_INTEGRATION / skill 文件）。

## 三、WorkBuddy 能力的具体使用

- **Web 检索 / 页面抓取**：查活动规则、拉排行榜、扒竞品 README 与实现
- **文件读写**：创建项目目录、编写 DESIGN.md / index.html / 参赛文档
- **本地托管预览**：用 Python `http.server` 起本地服务器，用户在浏览器试玩竞品与本项目
- **结构化决策**：用选择题工具与用户逐轮确认方向，遵守「动文件前先点头」的协作约定
- **MCP 联动设计**：Skill 设计为「对话侧调麦当劳 MCP → 注入游戏」，体现 WorkBuddy × 麦当劳 MCP 的真实联动

## 四、与麦当劳 MCP 的联动点

- 通过 WorkBuddy 连接器调用 `query-meals` / `campaign-calendar` 拉真实数据
- 模拟经营（mcd-surrounded）：菜单→订单池、活动→限定高分单
- 数据注入游戏前端（`localStorage['mcd_surrounded_menu']` / `['mcd_surrounded_events']` 或 `?config=` base64），实现「真实菜单驱动玩法」
- 完整流程见 `MCP_INTEGRATION.md` 与 `skill/SKILL.md`

## 五、提交说明

- 本仓库待创建（GitHub 空 Public 仓库，创建时间将在 2025-12-25 ~ 2026-10-25 活动窗口内）
- 已提交 `workbuddy.md` 以申请 WorkBuddy 专项 3000 积分
- 报名 Issue 待提交（仓库建好、代码推送后，按官方格式发布；项目名称：救命！我被麦麦包围了！）
