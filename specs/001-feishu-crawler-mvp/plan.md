# Implementation Plan: 飞书-Claude Playwright 自动化爬虫系统 MVP

**Branch**: `001-feishu-crawler-mvp` | **Date**: 2026-04-01 | **Spec**: [spec.md](./spec.md)

## Summary

构建一个飞书 Bot 驱动的微信公众号文章自动化抓取系统。用户通过飞书发送 `/爬取 [URL]` 命令，系统使用 Claude Code CLI + Playwright MCP 模拟浏览器操作抓取文章，提取结构化数据后写入 Notion News 数据库，并通过飞书卡片消息返回结果。核心技术路径：飞书 Webhook → SQLite 任务队列 → 调度器 → Claude Playwright → Notion → 飞书消息推送。

## Technical Context

**Language/Version**: TypeScript (Node.js 18+)
**Primary Dependencies**: Express.js, better-sqlite3, @larksuiteoapi/node-sdk, child_process (Claude CLI)
**Storage**: SQLite (任务队列), Notion API (文章持久化)
**Testing**: 手动测试（MVP 阶段）
**Target Platform**: macOS/Linux 服务器，PM2 守护进程
**Project Type**: Backend service (单体应用)
**Performance Goals**: 单任务 P95 ≤ 90s，任务成功率 ≥ 80%
**Constraints**: 单任务超时 120s，队列上限 100 个任务，串行执行（MVP）
**Scale/Scope**: 支持 10-20 人团队使用，日处理 50-100 篇文章

## Project Structure

### Source Code

```
src/
├── index.ts                 # 入口
├── config.ts                # 配置
├── webhook/
│   ├── server.ts            # Express 服务
│   ├── handler.ts           # 命令处理
│   └── validator.ts         # 签名校验
├── queue/
│   ├── db.ts                # SQLite 连接
│   ├── taskStore.ts         # 任务 CRUD
│   └── scheduler.ts         # 调度器
├── playwright/
│   ├── executor.ts          # Claude CLI 调用
│   ├── parser.ts            # 结果解析
│   └── prompt.ts            # Prompt 模板
├── feishu/
│   ├── client.ts            # 飞书 API
│   ├── card.ts              # 卡片模板
│   └── auth.ts              # 鉴权
├── notion/
│   └── writer.ts            # Notion 写入
└── utils/
    ├── logger.ts            # 日志
    └── errors.ts            # 错误码

prompts/
└── fetch-wechat.md          # Playwright Prompt

migrations/
└── 001_init.sql             # 建表脚本
```

## Implementation Phases

### Phase 1: 基础设施搭建
- 初始化 TypeScript 项目（package.json, tsconfig.json）
- 配置环境变量（.env.example）
- SQLite 数据库初始化（tasks 表）
- 飞书 Bot 创建和 Webhook 配置

### Phase 2: 核心功能实现
- 飞书 Webhook 接收和命令解析
- 任务队列 CRUD 操作
- 调度器轮询逻辑
- Claude CLI + Playwright 执行器
- Notion API 写入器
- 飞书卡片消息推送

### Phase 3: 测试和部署
- 端到端测试（手动）
- PM2 配置
- 部署到本地服务器
- 文档完善（README.md）

## Task Management

本项目使用 **TaskCreate/TaskUpdate 工具**管理任务，不生成 tasks.md 文件。
