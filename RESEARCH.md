# 调研资料（仅供参考）

> **重要：本文件是调研阶段的发现，不代表最终实施方案。**
> 工具 API 可能已变更、文档可能过时、功能可能有隐藏限制。
> **开发时必须自行验证每个工具的实际能力，不要盲目依赖本文档。**

## 一、微信公众号官方 API

### 草稿箱 API（完整支持）

| 接口 | 路径 | 能力 |
|------|------|------|
| 新增草稿 | `POST /cgi-bin/draft/add` | 图文素材写入草稿箱 |
| 获取草稿 | `POST /cgi-bin/draft/get` | 根据 media_id 获取 |
| 获取草稿列表 | `POST /cgi-bin/draft/batchget` | 批量获取 |
| 删除草稿 | `POST /cgi-bin/draft/delete` | 删除指定草稿 |
| 发布草稿 | `POST /cgi-bin/freepublish/submit` | 将草稿提交发布 |
| 查询发布状态 | `POST /cgi-bin/freepublish/batchget` | 获取发布结果 |

Base URL: `https://api.weixin.qq.com`

### 发布流程

```
1. 上传图片 → POST /cgi-bin/material/add_material → 获取 thumb_media_id
2. 创建草稿 → POST /cgi-bin/draft/add → 获取 media_id
3. 发布草稿 → POST /cgi-bin/freepublish/submit
```

### 关键约束

- 需要已认证的非个人主体公众号
- **2025年7月起**，未认证账号无法调用发布接口
- content 中图片必须通过官方接口上传，外部 URL 会被过滤
- 单次最多 8 篇图文

### 官方文档

- 草稿箱指南: https://developers.weixin.qq.com/doc/subscription/guide/product/draft.html
- 新增草稿: https://developers.weixin.qq.com/doc/offiaccount/Draft_Box/Add_draft.html

## 二、小红书

### 官方 API：不存在

小红书没有面向内容创作者的官方发布 API。商家开放平台只管商品，不管内容。

### 非官方方案

| 工具 | Stars | 能力 | 草稿支持 | 风险 |
|------|-------|------|---------|------|
| xiaohongshu-mcp | 8800+ | 发布、评论、搜索 | ❌ 无（有 issue 请求） | Cookie 过期 |
| Spider_XHS | 高 | 完整逆向签名+发布 | 有草稿管理界面 | 违反 TOS |
| xhs-toolkit | MCP | 内容创作+发布+数据分析 | 未确认 | Cookie 依赖 |
| MultiPost | 2.2K | 多平台一键发布 | 未确认 | 浏览器依赖 |

### 认证方式

全部依赖 Cookie（扫码登录/手机验证码/Cookie 导入），Cookie 会过期。

## 三、多平台同步工具

### MultiPost-Extension（重点推荐验证）

- GitHub: https://github.com/leaperone/MultiPost-Extension
- Stars: 2.2K
- 类型: Chrome/Edge 浏览器扩展
- 支持: 知乎、微博、小红书、抖音、B站、Twitter/X 等 10+ 平台
- **提供 RESTful API**（可在脚本/服务端调用）
- **提供扩展 API**（可在 Web 应用中调用）
- 无需登录注册、无需 API Key
- 最新版本: v1.3.8 (2026-03-03)，活跃维护
- **需要验证**: RESTful API 的实际可用性、是否真的可以脱离浏览器环境调用

### WeChatSync（文章同步助手）

- GitHub: https://github.com/wechatsync/Wechatsync
- Stars: 3.7K
- 类型: Chrome 浏览器扩展
- 支持: 29+ 平台（知乎、头条、掘金、小红书、CSDN、WordPress 等）
- 推送到各平台**草稿箱**（不直接发布）
- 依赖浏览器已登录状态

### 其他

| 工具 | 特点 |
|------|------|
| TurboPush | 桌面应用 (Tauri + React)，跨平台 |
| pubcast | 内置浏览器，一对多批量分发 |
| 融媒宝 (SaaS) | 50+ 平台，付费使用 |

## 四、neuDrive 交互模式（参考借鉴）

### 核心架构

```
Agent ←→ MCP Server ←→ Database ←→ Web UI
```

松耦合模式（非实时 WebSocket）：
- Agent 通过 MCP 工具按需读写
- Web UI 独立浏览管理
- 共享数据层是唯一通信渠道

### 关键设计模式

1. **统一虚拟文件树**: 一切数据映射为路径，Agent 只需 `list` + `read` 原语
2. **来源追踪**: 每次写入带 `source` 和 `source_platform` 元数据
3. **项目日志**: `log_action()` 记录所有 Agent 操作
4. **多级导入路径**: 小文本 inline → 小文件 base64 → 大文件预签名 URL

### 可借鉴的 MCP 工具设计

neuDrive 暴露 ~25 个 MCP 工具，覆盖完整 CRUD：
- `write_file` / `read_file` / `list_directory`
- `save_memory` / `search_memory`
- `create_project` / `log_action`
- `update_profile` / `read_profile`

## 五、已有的本地工具

| 工具 | 位置 | 能力 |
|------|------|------|
| huashu-xhs-image | `.agents/skills/huashu-xhs-image/` | 生成小红书配图 + 上传 ImgBB |
| huashu-article-to-x | `.agents/skills/huashu-article-to-x/` | 长文精简为小红书/推特格式 |
| web-access | `~/.claude/skills/web-access/` | CDP 浏览器自动化，可操作小红书创作者中心 |
