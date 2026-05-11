# UniPub — 功能清单与完成标准

> 每个 Feature 有明确的「完成标准」。Agent 不能自行定义「完成了」——必须满足下方的 Done 条件。

## Phase 0: 项目骨架

### F0.1 项目初始化
- [ ] `package.json` 包含所有依赖
- [ ] `tsconfig.json` 配置 strict 模式
- [ ] `npm run dev` 启动不报错
- [ ] `curl localhost:3000/api/health` 返回 `{"status":"ok"}`
- [ ] git commit: `chore: project skeleton`

## Phase 1: 数据层

### F1.1 SQLite 数据库
- [ ] `src/core/db.ts` 初始化数据库 + 建表
- [ ] `content` 表包含: id, platform, title, body, images, status, result, platform_id, metadata, created_at, updated_at
- [ ] 启动时自动创建 `data/content.db`

### F1.2 CRUD 操作
- [ ] `src/core/content.ts` 实现 create/read/update/delete/list
- [ ] 所有操作有 TypeScript 类型

### F1.3 REST API
- [ ] `GET /api/content` — 列出内容
- [ ] `GET /api/content/:id` — 获取单条
- [ ] `POST /api/content` — 创建内容
- [ ] `PUT /api/content/:id` — 更新内容
- [ ] `DELETE /api/content/:id` — 删除内容
- [ ] 用 curl 测试全部通过
- [ ] git commit: `feat: content CRUD + REST API`

**Phase 1 Done = 数据库能存取内容 + REST API 全部可用**

## Phase 2: MCP Server

### F2.1 MCP Server 入口
- [ ] `src/mcp/server.ts` 使用 @modelcontextprotocol/sdk 创建 stdio MCP Server
- [ ] 注册 7 个工具: create_content, read_content, list_content, update_content, delete_content, publish_content, get_platforms
- [ ] 能独立启动不报错

### F2.2 MCP 工具实现
- [ ] 每个工具调 `src/core/content.ts` 共享逻辑
- [ ] 在另一个 Claude Code 会话中连接 MCP，能看到所有工具
- [ ] 调用 `create_content` 后，curl REST API 能看到数据
- [ ] git commit: `feat: MCP server with 7 tools`

**Phase 2 Done = Agent 通过 MCP 创建的内容，Web API 能读到**

## Phase 3: 平台适配器

### F3.1 适配器接口
- [ ] `src/adapters/types.ts` 定义 `PlatformAdapter` 接口
- [ ] 包含 `validate()` 和 `publish(content)` 方法

### F3.2 微信公众号适配器
- [ ] `src/adapters/wechat.ts` 实现
- [ ] `validate()` 测试 access_token 可获取
- [ ] `publish()` 调 draft/add → freepublish/submit
- [ ] POC 测试：能成功写入草稿箱
- [ ] git commit: `feat: wechat adapter`

### F3.3 小红书适配器
- [ ] `src/adapters/xiaohongshu.ts` 实现
- [ ] 先验证工具（MultiPost / xiaohongshu-mcp / 其他），30 分钟不行就换
- [ ] `publish()` 能成功发布（或标记 TODO 如果全失败）
- [ ] git commit: `feat: xiaohongshu adapter`

### F3.4 知乎适配器
- [ ] `src/adapters/zhihu.ts` 实现
- [ ] 调研知乎 API/发布方式（搜索 GitHub + 知乎创作者中心）
- [ ] 找到可行方案后实现，找不到则标记 TODO
- [ ] git commit: `feat: zhihu adapter`

### F3.5 发布端点
- [ ] `POST /api/content/:id/publish` 调用对应适配器
- [ ] 返回发布结果（成功/失败/平台ID）
- [ ] git commit: `feat: publish endpoint`

**Phase 3 Done = 至少微信适配器能真实发布，其他至少有接口**

## Phase 4: Web 前端

### F4.1 React 项目初始化
- [ ] Vite + React + Tailwind CSS 配置完成
- [ ] 能在浏览器打开看到页面

### F4.2 内容列表页
- [ ] 显示所有内容卡片（标题 + 平台 + 状态 + 时间）
- [ ] 状态标签颜色: draft(灰) ready(蓝) published(绿) failed(红)

### F4.3 内容预览
- [ ] 小红书: 3:4 竖版卡片预览
- [ ] 微信: 文章样式预览
- [ ] 知乎: 问答/文章样式预览

### F4.4 发布按钮
- [ ] 点击后调 `/api/content/:id/publish`
- [ ] 显示发布结果（成功/失败）
- [ ] git commit: `feat: web frontend`

**Phase 4 Done = 浏览器能看到内容列表、预览、点击发布**

## Phase 5: 端到端集成

### F5.1 完整流程验证
- [ ] Claude Code 通过 MCP 创建一条内容
- [ ] Web 界面自动显示新内容
- [ ] 点击预览，内容格式正确
- [ ] 点击发布，至少一个平台成功

### F5.2 README + 收尾
- [ ] README.md: 启动方法、MCP 配置、环境变量说明
- [ ] `.env.example` 模板文件
- [ ] git commit: `docs: README and env template`
- [ ] git tag: `v0.1.0-mvp`

**Phase 5 Done = 从 Agent 创建到发布，全流程可运行**
