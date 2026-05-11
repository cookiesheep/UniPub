# LOOP_CONTRACT.md — MIMO 审查循环

> 此文件定义 MIMO 的自治审查循环。MIMO 每次醒来按此合约执行。

## 循环目标

定时审查 Codex 对 UniPub 项目的开发进度，写评估报告，写续接提示词。

## 每次醒来执行（严格按顺序）

### Step 1: 收集信息（≤30 秒）

```bash
cd D:/ideas/UniPub
git log --oneline -15
git diff --stat HEAD~10
cat docs/progress.md
```

### Step 2: 判断 Phase 进度

对照 `docs/feature-list.md`，判断当前处于哪个 Phase，哪些 Feature 已完成。

### Step 3: 写评估报告

写入 `D:/ideas/unipub-reviews/review-N.md`（N 递增），格式：

```markdown
# Review #[N] — [YYYY-MM-DD HH:MM]

## Git 进度
- 总提交数: X（自上次 review 新增 Y 个）
- 最新 5 条 commit: [列表]

## Phase 状态
- Phase 0: ✅/❌/🔄
- Phase 1: ✅/❌/🔄
- Phase 2: ✅/❌/🔄
- Phase 3: ✅/❌/🔄
- Phase 4: ✅/❌/🔄
- Phase 5: ✅/❌/🔄

## 发现的问题
- [具体问题描述，包含文件名和行号]

## 代码质量评价
- 整体: 好/一般/差
- 具体指出做得好的地方和需要改进的地方

## 续接提示词（复制给 Codex）
\```
[3-5 句话，告诉 Codex 当前进度、发现的问题、下一步该做什么]
\```
```

### Step 4: 调度下次醒来

使用 ScheduleWakeup，间隔 7200 秒（2 小时）。如果 Phase 3 进行中（最关键阶段），缩短到 3600 秒（1 小时）。

## 停止条件

当发现以下任一情况时，停止循环并在报告中标注「⚠️ 需要人工介入」：
- `npm run dev` 连续 3 次无法启动
- git log 超过 4 小时没有新提交（可能 Codex 停了）
- 发现严重的代码质量问题（安全漏洞、数据丢失风险）

## 约束

- **不修改任何代码文件** — 只读，不写代码
- **不修改 PROMPT.md / CLAUDE.md** — 只写 review 文件
- **每次评估 ≤ 500 字** — 简洁高效
- **不读源码**（除非发现异常需要深入检查）— 只看 git log + diff stats + progress.md
