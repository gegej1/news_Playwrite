# 交接文档：Codex 实现任务

## 项目概述

飞书-Claude Playwright 自动化爬虫系统 MVP

**仓库：** https://github.com/gegej1/news_Playwrite/tree/001-feishu-crawler-mvp

## 你的任务

按照 `specs/001-feishu-crawler-mvp/plan.md` 实现代码，完成 12 个任务。

## 快速开始

1. **查看规格文档**
   ```bash
   cat specs/001-feishu-crawler-mvp/spec.md
   cat specs/001-feishu-crawler-mvp/plan.md
   ```

2. **查看任务列表**
   - 在 Codex 中运行 `TaskList` 查看所有任务
   - 从任务 #1 开始实现

3. **遵循规范**
   - 阅读 `.specify/memory/constitution.md`
   - 阅读 `AGENTS.md`

## 任务顺序

1. 初始化 TypeScript 项目结构
2. 创建 SQLite 数据库和 tasks 表
3. 实现飞书 Webhook 接收服务
4. 实现任务队列 CRUD 操作
5. 实现任务调度器
6. 实现 Claude Playwright 执行器
7. 实现 Notion API 写入器
8. 实现飞书消息推送
9. 实现主入口和配置管理
10. 配置 PM2 和外部服务
11. 端到端测试验证
12. 完善项目文档

## 工作流程

每完成一个任务：
1. 使用 `TaskUpdate` 标记为 in_progress
2. 实现代码
3. 测试验证
4. Git commit
5. 使用 `TaskUpdate` 标记为 completed
6. 继续下一个任务

## 技术栈

- TypeScript + Node.js 18+
- Express.js (Webhook)
- SQLite (任务队列)
- Claude Code CLI + Playwright MCP (抓取)
- Notion API (存储)
- 飞书 API (消息推送)

## 注意事项

- 本地部署，先在 macOS 上运行
- MVP 阶段串行处理任务
- 遵循最小化实现原则

## 联系

遇到架构问题或需要调整方案，联系 Claude Code（总司令）。
