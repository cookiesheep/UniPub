# UniPub — Codex 开发 Prompt

> 基于 Anthropic Harness Design 方法论。Codex 按此文档自主开发，MIMO 定时审查。

---

## 你的角色

你是 UniPub 项目的**主开发者**（Codex）。你需要在 `D:/ideas/UniPub/` 下完成一个可运行的 MVP。

**开发节奏：**
1. 读 `CLAUDE.md` 了解项目规范
2. 读 `docs/feature-list.md` 了解完整功能清单和 Done 标准
3. 读 `RESEARCH.md` 了解平台调研（仅供参考，自己验证）
4. 按 Phase 0→5 顺序开发
5. 每个 Feature 完成后 git commit + 更新 `docs/progress.md`
6. **每个 Phase 完成后，读 `D:/ideas/unipub-reviews/` 下最新的 review 文件**，如果有人指出了问题，优先修复再继续

---

## 项目是什么

UniPub 是一个**AI Agent 驱动的多平台内容发布平台**。

核心流程：
```
用户在 Claude Code 里说「帮我写一篇小红书帖子」
    ↓
Agent 调用 MCP 工具 create_content() 保存内容
    ↓
用户打开浏览器 localhost:3000，看到内容预览
    ↓
确认没问题，点击「发布」
    ↓
内容发布到小红书/微信/知乎
```

核心创新：**Agent ↔ Web 双向交互**（市面上没有）。

---

## 功能边界

### MVP 包含
- MCP Server：Agent 通过 7 个工具读写内容
- Web 前端：列表 + 预览 + 发布
- 3 个平台适配器：微信公众号、小红书、知乎
- 自建内容预览（按平台格式渲染）
- SQLite 数据库

### 不包含
- 数据分析/流量统计
- 多用户系统 / 定时发布
- 更多平台（MVP 只做 3 个）
- 接入 ChatGPT/Gemini

---

## 技术决策

| 决策 | 选择 | 理由 |
|------|------|------|
| 语言 | TypeScript strict | 类型安全，MCP SDK 原生支持 |
| 后端 | Hono + @hono/node-server | 轻量，适合 API 服务 |
| 数据库 | better-sqlite3（优先）/ sql.js（Windows 编译失败时降级） | 零配置，单文件 |
| MCP | @modelcontextprotocol/sdk + zod v4 | 官方 SDK |
| 前端 | React + Vite + Tailwind | 开发速度快 |

**已知注意项（来自前一轮调研）：**
- `import * as z from 'zod/v4'`（MCP SDK 要求 zod v4 入口）
- MCP Server 和 HTTP Server 必须分离入口（StdioServerTransport 独占 stdout）
- SIGINT 处理集中到 index.ts，避免 double-close
- better-sqlite3 在 Windows 上可能需要 prebuilt binary，编译失败时降级 sql.js

---

## 开发节奏

每个 Phase 的开发流程：

```
1. 读 feature-list.md 中对应 Phase 的 Done 标准
2. 实现 Feature
3. 验证 Done 标准全部满足
4. 在 docs/progress.md 记录结果
5. git commit（conventional commits: feat:/fix:/docs:/chore:）
6. 读 unipub-reviews/ 最新 review（如果有），修复指出的问题
7. 下一个 Feature
```

Phase 之间打 tag: `git tag phase-N-done`

---

## Sprint 合约

### Phase 0 — 项目骨架 (≤30 min)

**Done:** `npm run dev` 启动后，`curl localhost:3000/api/health` 返回 JSON。

commit: `chore: project skeleton`

### Phase 1 — 数据层 (≤2h)

**Done:** curl 完成创建/读取/更新/删除/列表全部通过，数据正确存入 SQLite。

commit: `feat: content CRUD + REST API`

### Phase 2 — MCP Server (≤3h)

**Done:** 另一个 Claude Code 连接 MCP → 调 `create_content` → curl REST API 能看到数据。

commit: `feat: MCP server with 7 tools`

注意：MCP Server 用独立入口（`--mcp` flag），不与 HTTP Server 混用。

### Phase 3 — 平台适配器 (≤4h)

**Done:** 至少微信适配器能写入草稿箱。小红书和知乎尽力，搞不定标记 TODO。

每个适配器的验证流程：
```
1. 搜索该平台最新的 GitHub 工具（不要只看 RESEARCH.md）
2. 写 10 行 POC 测试核心 API
3. POC 通过 → 集成
4. 30 分钟搞不定 → 标记 TODO，继续下一个
```

commit（每个适配器一个）: `feat: wechat adapter` / `feat: xiaohongshu adapter` / `feat: zhihu adapter`

### Phase 4 — Web 前端 (≤4h)

**Done:** 浏览器 localhost:3000 → 内容列表 → 预览 → 点击发布。

UI：功能导向，状态标签 draft(灰) ready(蓝) published(绿) failed(红)。小红书 3:4 卡片预览。

commit: `feat: web frontend`

### Phase 5 — 端到端 (≤2h)

**Done:** Agent 创建内容 → Web 显示 → 预览 → 发布，全流程可运行。

commit: `docs: README and env template`
tag: `git tag v0.1.0-mvp`

---

## 平台适配器 — 验证优先级

### 微信公众号（最有把握）
官方 API: `/cgi-bin/draft/add` → `/cgi-bin/freepublish/submit`
需要: `WECHAT_APP_ID` + `WECHAT_APP_SECRET`

### 小红书（需验证，每个方案限时 30 分钟）
1. MultiPost RESTful API (github.com/leaperone/MultiPost-Extension)
2. xiaohongshu-mcp (github.com/xpzouying/xiaohongshu-mcp, 8800+ stars)
3. web-access CDP 浏览器自动化
4. Spider_XHS Python 逆向

### 知乎（需调研）
搜索「知乎 发布 API」/ 「zhihu publish API github」，找到方案后验证，找不到标记 TODO。

---

## Git 安全规则

```bash
# ✅ 必须
git add <具体文件>
git commit -m "feat: ..."
git tag phase-N-done

# ❌ 绝对不能
git reset --hard
git push --force
git clean -f
git add .
```

**改坏了回退：**
```bash
git diff                    # 看改了什么
git checkout -- <file>      # 回退单个文件
git log --oneline -5        # 找到好的 commit
git checkout <commit> -- .  # 回到那个版本
```

---

## 质量自检

| 维度 | 标准 | 不通过则 |
|------|------|---------|
| 功能 | Sprint 合约 Done 标准全部满足 | 继续修 |
| 可运行 | `npm run dev` 不报错 | 继续修 |
| 无回归 | 之前 Phase 功能正常 | git 回退 |
| 代码量 | 每文件 ≤ 300 行 | 拆分文件 |

---

## 阻塞处理

| 类型 | 处理 |
|------|------|
| API 不可用 | 搜替代 → 30 分钟换下一个 |
| TS 类型错误 | `any` 临时绕过，记 TODO |
| 前端渲染问题 | 最简 HTML 替代 React |
| 整个 Phase 搞不定 | Commit 已完成部分，继续下一个 |

记录到 `docs/progress.md`。

---

## 环境变量

```env
WECHAT_APP_ID=
WECHAT_APP_SECRET=
PORT=3000
DB_PATH=./data/content.db
```

---

## 开始

1. 读 `CLAUDE.md`
2. 读 `docs/feature-list.md`
3. 读 `RESEARCH.md`（仅供参考）
4. 从 Phase 0 开始
5. 每个 Feature: 实现 → 验证 → progress.md → git commit
6. 每个 Phase 结束: 读 `D:/ideas/unipub-reviews/` 最新 review → 修复 → 继续

> 现在开始开发。
