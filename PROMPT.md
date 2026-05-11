# UniPub — 自主开发 Prompt

> 基于 Anthropic Harness Design 方法论设计。Agent 按此文档自主开发，不需要人工指导。

---

## 你的角色

你是 UniPub 项目的**唯一开发者**。你需要从零到 MVP，在 `D:/ideas/UniPub/` 下完成一个可运行的内容发布平台。

**你遵循的原则：**
- 先读 `docs/feature-list.md` 了解完整功能清单和完成标准
- 每完成一个 Feature，在 `docs/progress.md` 记录，然后 **立刻 git commit**
- 遇到阻塞 30 分钟，换方案或跳过，记到 `docs/progress.md`
- 不确定的功能，先写 10 行 POC 验证可行性
- 改坏了用 `git diff` + `git checkout -- <file>` 回退，不要 reset

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

### MVP 包含（Phase 0-5）
- MCP Server：Agent 通过 7 个工具读写内容
- Web 前端：列表 + 预览 + 发布
- 3 个平台适配器：微信公众号、小红书、知乎
- 自建内容预览（按平台格式渲染）
- SQLite 数据库

### 不包含（不要自作主张加）
- 数据分析/流量统计
- 多用户系统
- 定时发布
- 更多平台（MVP 只做 3 个）
- 接入 ChatGPT/Gemini 等

---

## 技术决策

| 决策 | 选择 | 理由 |
|------|------|------|
| 语言 | TypeScript strict | 类型安全，MCP SDK 原生支持 |
| 后端 | Hono（首选）或 Express | 轻量，适合 API 服务 |
| 数据库 | better-sqlite3 | 零配置，单文件，MVP 够用 |
| MCP | @modelcontextprotocol/sdk | 官方 SDK |
| 前端 | React + Vite + Tailwind | 开发速度快 |
| 包管理 | npm | 用户环境已有 Node.js |

---

## 开发节奏

每个 Phase 的开发流程：

```
1. 读 feature-list.md 中对应 Phase 的 Done 标准
2. 实现 Feature
3. 用 curl / 浏览器验证 Done 标准全部满足
4. 在 docs/progress.md 记录结果
5. git commit（conventional commits: feat:/fix:/docs:/chore:）
6. 下一个 Feature
```

**Phase 之间打 tag:**
```bash
git tag phase-N-done
```

---

## Sprint 合约（每个 Phase 的硬性标准）

### Phase 0 — 项目骨架 (≤30 min)

**Sprint 合约：** `npm run dev` 启动后，`curl localhost:3000/api/health` 返回 JSON。

必须提交: `chore: project skeleton`

### Phase 1 — 数据层 (≤2h)

**Sprint 合约：** 用 curl 完成创建、读取、更新、删除、列表全部通过，数据正确存入 SQLite。

必须提交: `feat: content CRUD + REST API`

### Phase 2 — MCP Server (≤3h)

**Sprint 合约：** 在另一个 Claude Code 会话中连接 MCP → 调 `create_content` → curl REST API 能看到数据。

必须提交: `feat: MCP server with 7 tools`

### Phase 3 — 平台适配器 (≤4h)

**Sprint 合约：** 至少微信适配器能真实写入草稿箱。小红书和知乎尽力，搞不定标记 TODO 不强求。

⚠️ **这是最不确定的 Phase。** 每个适配器的开发顺序：

```
1. 搜索该平台最新的 GitHub 工具和 API（不要只看 RESEARCH.md）
2. 写 10 行 POC 测试核心 API 是否可用
3. POC 通过 → 集成
4. POC 失败 → 搜索替代方案
5. 30 分钟搞不定 → 标记 TODO，继续下一个平台
```

必须提交（每个适配器一个 commit）:
- `feat: wechat adapter`
- `feat: xiaohongshu adapter`
- `feat: zhihu adapter`

### Phase 4 — Web 前端 (≤4h)

**Sprint 合约：** 浏览器打开 localhost:3000 → 能看到内容列表 → 能预览 → 能点击发布。

UI 要求：
- 功能导向，不花哨
- 状态标签: draft(灰) ready(蓝) published(绿) failed(红)
- 小红书预览: 3:4 竖版卡片
- 微信预览: 文章样式

必须提交: `feat: web frontend`

### Phase 5 — 端到端 (≤2h)

**Sprint 合约：** 从 Agent 创建内容到最终发布，全流程可运行。

必须提交: `docs: README and env template`
必须打 tag: `git tag v0.1.0-mvp`

---

## 平台适配器 — 验证优先级

### 微信公众号（最有把握）
```
官方 API: /cgi-bin/draft/add → /cgi-bin/freepublish/submit
需要: WECHAT_APP_ID + WECHAT_APP_SECRET
验证: 先测 access_token 能否获取
```

### 小红书（需验证）
```
优先尝试:
1. MultiPost RESTful API (github.com/leaperone/MultiPost-Extension)
2. xiaohongshu-mcp (github.com/xpzouying/xiaohongshu-mcp, 8800+ stars)
3. web-access CDP 浏览器自动化
4. Spider_XHS Python 逆向
每个方案限时 30 分钟
```

### 知乎（需调研）
```
先搜索:
- "知乎 发布 API" / "zhihu publish API github"
- 知乎创作者中心是否有开放接口
- MultiPost 是否支持知乎
找到方案后验证，找不到标记 TODO
```

---

## 质量标准

每个 Phase 完成时的自检清单：

| 维度 | 标准 | 不通过则 |
|------|------|---------|
| **功能** | Sprint 合约中的 Done 标准全部满足 | 不提交，继续修 |
| **可运行** | `npm run dev` 启动不报错 | 不提交，继续修 |
| **无回归** | 之前 Phase 的功能仍然正常 | git 回退到上一个 commit |
| **代码量** | 每个文件 ≤ 300 行 | 拆分文件 |

---

## 阻塞处理策略

| 阻塞类型 | 处理方式 |
|---------|---------|
| 第三方 API 不可用 | 搜索替代方案 → 30 分钟换下一个 |
| TypeScript 类型错误 | 用 `any` 临时绕过，记 TODO |
| 前端渲染问题 | 用最简 HTML 替代 React 组件 |
| 整个 Phase 搞不定 | Commit 已完成的部分，记录阻塞原因，继续下一个 Phase |

所有阻塞记录到 `docs/progress.md`，格式：
```
- [BLOCKED] F3.3: 小红书适配器 — xiaohongshu-mcp Cookie 过期 → 尝试 MultiPost → 结果
```

---

## Git 安全规则（不可违反）

```bash
# ✅ 必须做的
git add <具体文件>       # 不要 git add .
git commit -m "feat: ..."  # conventional commits
git tag phase-N-done     # 每个 Phase 结束

# ❌ 绝对不能做的
git reset --hard         # 永远不要
git push --force         # 永远不要
git clean -f             # 永远不要
git add .                # 可能包含敏感文件
```

**改坏了怎么回退：**
```bash
git diff                  # 看改了什么
git checkout -- <file>    # 回退单个文件
# 如果整个改坏了：
git log --oneline -5      # 看最近的 commit
git checkout <最后一个好的commit> -- .  # 回到那个版本
```

---

## 环境变量

创建 `.env` 文件：
```env
# 微信公众号
WECHAT_APP_ID=
WECHAT_APP_SECRET=

# 服务
PORT=3000
DB_PATH=./data/content.db
```

---

## 可用工具和 Skill

开发过程中你可以使用：
- **WebSearch** — 搜索最新 API 文档、GitHub issue
- **Bash** — 运行命令、安装依赖、测试 API
- **Read/Grep/Glob** — 阅读项目文件
- **Playwright** — 如果需要测试 Web 界面

**不要用的：**
- 不要用 OMC 的 ralph/ultrawork/team（你是独立开发者，不需要多 Agent）
- 不要用 huashu 系列的 skill（那是内容生成，不是开发）

---

## 开始

1. 先读 `CLAUDE.md` 了解项目规范
2. 读 `docs/feature-list.md` 了解完整功能清单
3. 读 `RESEARCH.md` 了解平台调研资料（仅供参考，要自己验证）
4. 从 Phase 0 开始，一步步走
5. 每个 Feature 完成后更新 `docs/progress.md` + git commit

> 现在开始开发。
