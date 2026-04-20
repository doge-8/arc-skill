# Arc Daily Task Skill

自动完成 [Arc Community](https://community.arc.network) 每日积分任务的 Claude Code skill。

## 这个 skill 做什么

每天最多帮你拿 **35 积分**：
- 阅读 5 篇内容（Blog / Resource / External Content）= 25 分
- 观看 1 个视频 = 5 分
- 触发每日签到（访问任意页面自动算）= 5 分

## 快速开始

### 1. 准备调试 Chrome（一次性）

skill 用独立调试 Chrome 操作浏览器，不影响你日常 Chrome。

- 启动脚本：`/Users/yuepin/Desktop/Chrome-Debug.app`（双击启动）
- 调试端口：`http://127.0.0.1:9333`
- 独立 profile：`~/Library/Application Support/Google/Chrome-Debug`

首次启动后，**在调试 Chrome 窗口里手动登录 Arc Community 一次**（独立 profile 与日常 Chrome 隔离，登录态会持久保留）。

### 2. 触发任务

跟 Claude Code 说任意一句：
- "跑 arc 任务"
- "做 arc 每日积分"
- "arc 签到"
- "arc daily task"

Claude 会自动按 `SKILL.md` 流程执行。

## 文件说明

| 文件 | 作用 |
|------|------|
| `SKILL.md` | 给 Claude 看的执行说明，frontmatter + 7 个步骤 |
| `arc_task_record.json` | 持久化状态：上次执行日期、未读未看库存、已读已看历史 |
| `README.md` | 本文档（给人看） |

## arc_task_record.json 结构

```json
{
  "last_run": "YYYY-MM-DD",         // UTC 日期，控制 24h 间隔
  "unread_articles": [...],          // 候选未读
  "unwatch_videos": [...],           // 候选未看
  "read_history": ["title1", ...],   // 已读 title 历史
  "video_history": ["title1", ...]   // 已看 title 历史
}
```

**主键是 title 字符串**（不是 slug）—— 因为 my-contributions 页面只暴露标题，不暴露 URL slug。

## 常见问题

**Q: 为什么不影响我日常的 Chrome？**
A: skill 用 9333 端口 + 独立 profile，与日常 Chrome 完全并存。要新窗口管理两个 Chrome（任务栏会有两个图标）。

**Q: 调试 Chrome 弹"远程调试授权"？**
A: 不会。`Chrome-Debug.app` 启动时已带 `--remote-debugging-port=9333`，开机即开端口，不弹窗。

**Q: 一天能跑两次吗？**
A: 不能。`last_run == today` 就拒绝执行，因为 Arc 24h 内重复浏览不计分。

**Q: 库存不足怎么办？**
A: 能做几个做几个。skill 会扫首页 100+ 个候选，通常 Blog/Resource 读完后还有几十篇 External Content。

**Q: 视频是 Wistia 还是 YouTube？**
A: 都有。skill 用 `take_snapshot` 找名为 "Play Video" / "Play video" 的按钮（两种平台兼容），找不到时降级用 iframe 选择器兜底。

**Q: 积分多久到账？**
A: 约 5 分钟，到 [my-contributions 页面](https://community.arc.network/home/contributors/my-contributions) 确认。

## 把 skill 装到全局

让所有 Claude Code 项目都能用：

```bash
ln -s /Users/yuepin/Desktop/wan/ai-skill/arc/arc-daily-task ~/.claude/skills/arc-daily-task
```

或者把整个目录拷过去。

## 维护

- skill 行为改动：编辑 `SKILL.md`，下次新会话生效
- 状态重置：删掉 `arc_task_record.json`，下次执行会从空 history 重建
- 临时强制重跑：把 `last_run` 改成昨天日期

## 已验证环境

- macOS 14+ / Chrome 147+
- chrome-devtools MCP（用户级 `~/.claude.json` 配 `--browserUrl http://127.0.0.1:9333`）
- Claude Code Opus 4.7
