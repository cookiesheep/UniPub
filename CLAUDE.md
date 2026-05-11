# CLAUDE.md — UniPub

## 一句话

AI Agent 驱动的多平台内容发布平台。Agent 生成内容 → Web 预览 → 一键发布到微信公众号、小红书、知乎。

## 技术栈

| 层 | 技术 |
|---|------|
| Runtime | Node.js 18+, TypeScript strict |
| 后端 | Hono 或 Express |
| 数据库 | better-sqlite3 |
| MCP | @modelcontextprotocol/sdk |
| 前端 | React + Vite + Tailwind CSS |

## 常用命令

```bash
npm run dev      # 启动开发服务器
npm run build    # 编译 TypeScript
npm run mcp      # 单独启动 MCP Server（测试用）
```

## 目录结构

```
src/
├── core/        # 数据库 + 类型 + CRUD
├── adapters/    # 平台适配器（wechat / xiaohongshu / zhihu）
├── mcp/         # MCP Server（Agent 接口）
├── api/         # REST API（Web 前端接口）
└── web/         # React 前端
```

## Git 纪律（铁律）

- **每个功能完成后立即 commit**，commit message 用 conventional commits: `feat:`, `fix:`, `docs:`, `refactor:`
- **绝不执行** `git reset --hard`, `git push --force`, `git clean -f`
- **发现改坏了** → `git diff` 定位 → `git checkout -- <file>` 回退单个文件 → 不要整个 reset
- **每个 Phase 结束后打 tag**: `git tag phase-1-done`

## 代码规范

- 每个文件 ≤ 300 行
- 不写测试（MVP 阶段）
- 不写注释（除非 WHY 不显而易见）
- 不过度抽象

## 平台适配器现状

| 平台 | API 类型 | 草稿支持 | 状态 |
|------|---------|---------|------|
| 微信公众号 | 官方 REST API | ✅ 完整草稿箱 | 待实现 |
| 小红书 | 非官方（需验证） | ❌ 无官方草稿 | 待验证 |
| 知乎 | 待调研 | 待确认 | 待调研 |

适配器接口统一: `validate() → {ok, error}` + `publish(content) → PublishResult`

## MCP 工具

Agent 通过以下工具与平台交互:
`create_content`, `read_content`, `list_content`, `update_content`, `delete_content`, `publish_content`, `get_platforms`

## 关键文件

- `RESEARCH.md` — 调研资料（修改适配器前先读）
- `docs/feature-list.md` — 功能清单和完成标准
- `docs/progress.md` — 开发进度记录
