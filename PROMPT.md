# Content Platform — MIMO 开发 Prompt

> 项目目录: `D:/ideas/content-platform/`
> 调研资料: `RESEARCH.md`（同目录下，仅供参考）
> 目标: 用 MIMO token 长时间自主开发，完成可运行的 MVP

---

## 一、你是谁

你是一个资深全栈开发者，正在独立开发一个**内容发布管理平台**。你需要从零开始，在 `D:/ideas/content-platform/` 目录下完成整个项目的开发。

你的工作方式：
- 自主决策技术细节，遇到非关键选择不要停下来问
- 每完成一个 Phase，运行项目确认能正常工作再继续
- 遇到工具/API 不工作时，**立即搜索替代方案，不死磕**
- 代码简洁，不过度设计，每个文件控制在 300 行以内

---

## 二、项目概述

### 要解决什么问题

用户（内容创作者）需要管理多个社交平台（微信公众号、小红书等）的内容发布。AI Agent 负责生成内容素材，用户在 Web 界面预览确认后一键发布。

### 核心创新点

**Agent ↔ Web 双向交互**——这是市面上没有的：

```
Agent (Claude Code)
    ↓ MCP 工具调用
    ↓ create_content("小红书", "标题", "正文", [图片URLs])
    ↓
MCP Server → 写入 SQLite 数据库
    ↓
Web 前端 ← REST API 读取 ← SQLite 数据库
    ↓
用户在浏览器看到 Agent 创建的内容
    ↓ 预览确认
    ↓ 点击「发布」按钮
    ↓
REST API → Platform Adapter → 微信/小红书 API → 发布完成
```

### 产品流程

1. Claude Code 里，用户让 Agent 生成一篇小红书帖子
2. Agent 调用 MCP 工具 `create_content` 保存到平台
3. 用户打开 Web 界面，看到新内容的预览
4. 用户确认内容没问题，点击「发布」
5. 平台调用小红书/微信 API 完成发布

---

## 三、MVP 范围（铁律，不得扩展）

### ✅ 包含

- MCP Server：Agent 通过 MCP 工具创建/读取/更新内容
- Web 前端：内容列表 + 预览 + 发布按钮
- 微信公众号适配器：通过官方 API 创建草稿 + 发布
- 小红书适配器：通过 MultiPost 或其他工具发布
- SQLite 数据库：存储所有内容
- 内容预览：在 Web 端按平台格式渲染预览

### ❌ 不包含（后续版本再考虑）

- 数据分析/流量统计
- 多用户系统
- 移动端 App
- 接入 ChatGPT/Gemini 等其他 AI
- 接入知乎/抖音/B站等更多平台
- 定时发布
- 内容版本管理

---

## 四、技术栈

| 层 | 技术 | 理由 |
|---|------|------|
| Runtime | Node.js 18+ | 用户环境已有 |
| 语言 | TypeScript strict | 类型安全 |
| 后端框架 | Hono 或 Express | 轻量 REST API |
| 数据库 | better-sqlite3 | 零配置，单文件 |
| MCP SDK | @modelcontextprotocol/sdk | 官方 SDK |
| 前端 | React + Vite | 快速开发 |
| 样式 | Tailwind CSS | 效率高 |

---

## 五、项目结构

```
D:/ideas/content-platform/
├── PROMPT.md              ← 你正在读的这个文件
├── RESEARCH.md            ← 调研资料（仅供参考）
├── package.json
├── tsconfig.json
├── src/
│   ├── core/              ← 共享核心
│   │   ├── db.ts          ← SQLite 数据库初始化
│   │   ├── types.ts       ← 类型定义
│   │   └── content.ts     ← 内容 CRUD 操作
│   ├── adapters/          ← 平台适配器
│   │   ├── types.ts       ← 适配器接口定义
│   │   ├── wechat.ts      ← 微信公众号适配器
│   │   └── multipost.ts   ← MultiPost 适配器
│   ├── mcp/               ← MCP Server
│   │   └── server.ts      ← MCP 工具定义和入口
│   ├── api/               ← REST API
│   │   └── server.ts      ← HTTP 路由
│   └── web/               ← 前端
│       ├── index.html
│       ├── App.tsx
│       └── components/
├── scripts/
│   └── dev.ts             ← 开发启动脚本
└── data/                  ← SQLite 数据文件（gitignore）
    └── content.db
```

---

## 六、数据模型

```typescript
interface Content {
  id: string               // UUID
  platform: string         // 'wechat' | 'xiaohongshu'
  title: string            // 标题
  body: string             // 正文 (Markdown 或 HTML)
  images: string[]         // 图片 URL 列表
  status: 'draft' | 'ready' | 'published' | 'failed'
  result: string | null    // 发布结果（成功/失败信息）
  platform_id: string | null  // 平台返回的 ID
  metadata: Record<string, any>  // 平台特有数据
  created_at: string       // ISO datetime
  updated_at: string       // ISO datetime
}
```

数据库表定义：

```sql
CREATE TABLE content (
  id TEXT PRIMARY KEY,
  platform TEXT NOT NULL,
  title TEXT NOT NULL,
  body TEXT NOT NULL DEFAULT '',
  images TEXT NOT NULL DEFAULT '[]',    -- JSON array
  status TEXT NOT NULL DEFAULT 'draft',
  result TEXT,
  platform_id TEXT,
  metadata TEXT NOT NULL DEFAULT '{}',  -- JSON object
  created_at TEXT NOT NULL DEFAULT (datetime('now')),
  updated_at TEXT NOT NULL DEFAULT (datetime('now'))
);

CREATE INDEX idx_content_platform ON content(platform);
CREATE INDEX idx_content_status ON content(status);
```

---

## 七、MCP Server 工具定义

MCP Server 是本项目的核心。Claude Code 通过这些工具与平台交互。

### 工具列表

| 工具名 | 参数 | 返回 | 说明 |
|--------|------|------|------|
| `create_content` | `{platform, title, body, images?}` | `Content` | 创建一条新内容 |
| `read_content` | `{id}` | `Content` | 读取指定内容 |
| `list_content` | `{platform?, status?}` | `Content[]` | 列出内容（可筛选） |
| `update_content` | `{id, title?, body?, images?}` | `Content` | 更新内容 |
| `delete_content` | `{id}` | `{ok: true}` | 删除内容 |
| `publish_content` | `{id}` | `{ok, result}` | 发布到目标平台 |
| `get_platforms` | `{}` | `Platform[]` | 获取可用平台列表 |

### MCP 配置（用户侧）

用户需要在 Claude Code 的 `.mcp.json` 中添加：

```json
{
  "mcpServers": {
    "content-platform": {
      "command": "node",
      "args": ["D:/ideas/content-platform/src/mcp/server.js"]
    }
  }
}
```

---

## 八、平台适配器接口

```typescript
interface PlatformAdapter {
  name: string

  // 验证适配器是否可用（API Key、Cookie 等）
  validate(): Promise<{ ok: boolean; error?: string }>

  // 发布内容到平台
  publish(content: Content): Promise<PublishResult>
}

interface PublishResult {
  success: boolean
  platform_id?: string   // 平台返回的 ID
  url?: string           // 发布后的 URL
  error?: string         // 失败原因
}
```

### 微信公众号适配器

使用官方 API：

```
1. 获取 access_token (client_credential 模式)
2. 上传封面图 → thumb_media_id
3. 调 draft/add → media_id（写入草稿箱）
4. 调 freepublish/submit → 发布
```

需要的配置：
- `WECHAT_APP_ID`
- `WECHAT_APP_SECRET`

### 小红书适配器

**⚠️ 关键提醒：调研阶段发现以下工具，但你必须自行验证哪个能用。**

优先级（依次尝试，不行就换）：

1. **MultiPost RESTful API** — 如果确实有可用的 REST API
2. **xiaohongshu-mcp** — 直接发布（无草稿），8800+ stars
3. **web-access skill (CDP)** — 通过浏览器自动化操作创作者中心
4. **Spider_XHS** — Python 逆向库，完整签名算法

**验证方法**：
- 先搜索该工具的最新 issue 和讨论，确认是否还在维护
- 写一个最小 POC（10 行代码）测试能否成功调用
- 如果 30 分钟内搞不定，立即换下一个方案

---

## 九、分阶段开发计划

### Phase 0: 项目初始化 ✦ 预计 30 分钟

**目标**: 项目可以 `npm run dev` 启动，看到 Hello World

**任务**:
1. `npm init -y`，安装依赖（typescript, tsx, hono/express, better-sqlite3, @modelcontextprotocol/sdk）
2. 创建目录结构（见第五节）
3. 写 `tsconfig.json`
4. 写一个最简单的 `src/api/server.ts` 返回 `{ status: "ok" }`
5. 配置 `package.json` scripts: `dev`, `build`

**验收**: `npm run dev` 启动后，`curl localhost:3000/api/health` 返回 `{"status":"ok"}`

---

### Phase 1: 核心数据层 ✦ 预计 2 小时

**目标**: SQLite 数据库 + 内容 CRUD 全部可用

**任务**:
1. `src/core/types.ts` — 定义 Content 类型
2. `src/core/db.ts` — SQLite 初始化 + 建表
3. `src/core/content.ts` — 完整 CRUD（create, read, list, update, delete）
4. `src/api/server.ts` — REST API 端点：
   - `GET    /api/content` — 列出所有内容
   - `GET    /api/content/:id` — 获取单个内容
   - `POST   /api/content` — 创建内容
   - `PUT    /api/content/:id` — 更新内容
   - `DELETE /api/content/:id` — 删除内容
   - `POST   /api/content/:id/publish` — 发布内容（先留空实现）

**验收**: 用 curl 测试所有 CRUD 端点，数据正确存入和读取 SQLite

---

### Phase 2: MCP Server ✦ 预计 3 小时

**目标**: Claude Code 可以通过 MCP 工具与平台交互

**任务**:
1. `src/mcp/server.ts` — 使用 @modelcontextprotocol/sdk 创建 MCP Server
2. 实现所有 7 个 MCP 工具（见第七节）
3. MCP Server 和 REST API 共享 `src/core/` 代码
4. 测试：在另一个终端用 Claude Code 连接 MCP Server，调用 `list_content` 确认返回数据

**关键**: MCP Server 是 stdio 模式（Claude Code 直接启动），不需要 HTTP。

**验收**:
1. MCP Server 能独立启动不报错
2. 在 Claude Code 里配置 MCP 后，能看到所有工具
3. 调用 `create_content` 后，用 curl 查 REST API 能看到数据

---

### Phase 3: 平台适配器 ✦ 预计 3-4 小时

**目标**: 能通过 Web 界面发布内容到微信公众号（和/或小红书）

**⚠️ 这是最需要验证的 Phase。不要假设调研报告里的工具能用。**

**任务**:
1. `src/adapters/types.ts` — 定义适配器接口
2. `src/adapters/wechat.ts` — 微信公众号适配器
   - 先写 POC 验证 access_token 能获取
   - 再实现 draft/add
   - 最后实现 freepublish/submit
3. `src/adapters/multipost.ts` — 小红书适配器
   - **先验证 MultiPost RESTful API 是否真的可用**
   - 如果不行，搜索其他方案（xiaohongshu-mcp 等）
   - 写 POC 验证，确认能发帖再集成
4. `src/api/server.ts` — 补全 `POST /api/content/:id/publish` 端点

**验证策略**（每个适配器）:
```
1. 搜索该工具 GitHub 最新 issue，看是否有人报告不可用
2. 写 10 行最小 POC 测试核心 API
3. POC 通过 → 集成到项目
4. POC 失败 → 搜索替代方案，30 分钟内找不到就跳过该平台
```

**验收**: 在 Web 界面点击「发布」按钮后，内容确实出现在微信公众号草稿箱或小红书

---

### Phase 4: Web 前端 ✦ 预计 4 小时

**目标**: 完整的 Web 界面，能看到内容、预览、发布

**任务**:
1. React + Vite 前端项目初始化
2. 内容列表页（显示所有内容卡片）
3. 内容详情/预览页
   - 小红书预览：3:4 竖版卡片（1080×1440 比例）
   - 微信公众号预览：文章样式
4. 发布按钮（调 `POST /api/content/:id/publish`）
5. 创建内容表单（手动创建，补充 MCP 创建的内容）

**UI 设计原则**:
- 简洁、功能导向，不需要花哨
- Tailwind CSS 快速搭建
- 响应式：桌面端为主
- 状态标签用颜色区分：draft(灰) → ready(蓝) → published(绿) → failed(红)

**验收**: 浏览器打开 `localhost:3000`，能看到内容列表、点击预览、点击发布

---

### Phase 5: 端到端集成 ✦ 预计 1-2 小时

**目标**: 完整流程跑通

**任务**:
1. 在 Claude Code 里通过 MCP 创建一条小红书内容
2. 打开 Web 界面，看到新内容
3. 预览内容，确认格式正确
4. 点击发布，确认内容出现在小红书
5. 修复发现的 bug
6. 写 README.md（启动方法、MCP 配置方法、环境变量）

**验收**: 从 Agent 创建内容到最终发布，全流程无手动介入（除点击发布按钮外）

---

## 十、⚠️ 开发铁律（必须遵守）

### 1. 验证优先，不信调研

调研报告（RESEARCH.md）里的信息可能已过时。开发时：
- **每个第三方工具/API，先写 POC 验证再集成**
- 搜索该工具的 GitHub 最新 issue 和 commit
- 验证时间控制：单个工具 30 分钟内搞不定就换方案
- 善用 `WebSearch` 搜索最新信息

### 2. 不死磕

遇到阻塞超过 30 分钟：
- 立即换一种实现方式
- 如果是平台 API 问题，降级为手动发布（先跑通流程）
- 如果是前端问题，用最简 HTML 替代 React
- 记录阻塞原因和替代方案到 `BLOCKERS.md`

### 3. 每步验证

每完成一个 Phase：
1. 运行 `npm run dev` 确认不报错
2. curl 测试 API 端点
3. 检查数据库数据是否正确
4. 确认后才开始下一个 Phase

### 4. 代码简洁

- 每个文件 ≤ 300 行
- 不写测试（MVP 阶段不需要）
- 不写注释（除非逻辑非常不直观）
- 不过度抽象（3 个相似函数 > 1 个过早抽象）

### 5. 降级策略

如果某个 Phase 实在完成不了：
- Phase 3（适配器）: 只实现微信公众号，小红书标记为 TODO
- Phase 4（前端）: 用纯 HTML + fetch 替代 React
- Phase 2（MCP）: 如果 MCP SDK 有问题，用自定义 HTTP API 替代

---

## 十一、环境变量

项目需要的配置（创建 `.env` 文件）：

```env
# 微信公众号
WECHAT_APP_ID=your_app_id
WECHAT_APP_SECRET=your_app_secret

# 小红书（如果用 xiaohongshu-mcp）
XHS_COOKIE=your_cookie

# 服务端口
PORT=3000

# 数据库路径
DB_PATH=./data/content.db
```

---

## 十二、预期产出

完成后的目录应该包含：

```
content-platform/
├── README.md              ← 启动方法、配置说明
├── PROMPT.md              ← 本文件
├── RESEARCH.md            ← 调研资料
├── BLOCKERS.md            ← 遇到的阻塞和解决方案（如果有）
├── package.json
├── tsconfig.json
├── .env.example           ← 环境变量模板
├── src/
│   ├── core/              ← 数据库 + CRUD
│   ├── adapters/          ← 平台适配器
│   ├── mcp/               ← MCP Server
│   ├── api/               ← REST API
│   └── web/               ← 前端
└── data/                  ← SQLite 文件（运行时生成）
```

**最终验收标准**: `npm run dev` 一条命令启动后：
1. `localhost:3000` 能看到 Web 界面
2. Claude Code 连上 MCP 后能 `create_content`
3. Web 界面能看到 Agent 创建的内容
4. 点击发布能将内容发到至少一个平台

---

> 开始开发吧。从 Phase 0 开始，一步步来。
