# Feature Specification: 飞书-Claude Playwright 自动化爬虫系统 MVP

**Feature Branch**: `001-feishu-crawler-mvp`
**Created**: 2026-04-01
**Status**: Draft
**Input**: User description: "飞书-Claude Playwright 自动化爬虫系统 MVP"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 飞书命令触发抓取 (Priority: P1)

运营人员在飞书群中发送 `/爬取 [微信公众号文章URL]`，系统自动抓取文章并返回结构化结果。

**Why this priority**: 这是系统的核心价值链路，从用户输入到结果输出的完整闭环。没有这个功能，系统无法交付任何价值。

**Independent Test**: 在飞书测试群发送 `/爬取 https://mp.weixin.qq.com/s/xxxxx`，30秒内收到包含标题、作者、摘要的卡片消息，即为成功。

**Acceptance Scenarios**:

1. **Given** 用户在飞书群中，**When** 发送 `/爬取 https://mp.weixin.qq.com/s/kma0-gsuavX58nYan66PAA`，**Then** 收到"✅ 任务已创建，任务ID: task_xxx"的确认消息
2. **Given** 任务已创建，**When** 等待60秒内，**Then** 收到包含文章标题、作者、公众号、摘要的飞书卡片消息
3. **Given** 抓取成功，**When** 点击卡片中的"查看原文"链接，**Then** 跳转到微信公众号原文页面

---

### User Story 2 - 任务状态查询 (Priority: P2)

用户可以通过 `/状态 [任务ID]` 查询任务的当前状态（pending/processing/completed/failed）。

**Why this priority**: 当抓取耗时较长时，用户需要了解任务进度，避免重复提交。

**Independent Test**: 提交任务后立即发送 `/状态 task_xxx`，返回当前状态信息。

**Acceptance Scenarios**:

1. **Given** 任务正在处理中，**When** 发送 `/状态 task_abc123`，**Then** 返回"⏳ 任务处理中，已用时 15 秒"
2. **Given** 任务已完成，**When** 发送 `/状态 task_abc123`，**Then** 返回完整的文章信息卡片
3. **Given** 任务失败，**When** 发送 `/状态 task_abc123`，**Then** 返回错误原因和重试建议

---

### User Story 3 - 失败任务重试 (Priority: P3)

当任务因验证码或网络问题失败时，用户可以通过 `/重试 [任务ID]` 手动重新执行。

**Why this priority**: 提升系统可用性，避免用户因临时故障而放弃使用。

**Independent Test**: 模拟一个失败任务，发送 `/重试 task_xxx`，任务重新进入队列。

**Acceptance Scenarios**:

1. **Given** 任务状态为 failed，**When** 发送 `/重试 task_abc123`，**Then** 返回"✅ 任务已重新加入队列"
2. **Given** 任务状态为 completed，**When** 发送 `/重试 task_abc123`，**Then** 返回"❌ 该任务已完成，无需重试"

### Edge Cases

- 当用户提交的 URL 不是微信公众号链接时，立即返回错误提示
- 当文章已被删除或不存在时，返回明确的错误信息
- 当同一 URL 在 24 小时内重复提交时，直接返回缓存结果
- 当任务队列已满（>100 个待处理任务）时，拒绝新任务并提示稍后重试
- 当 Claude CLI 不可用时，任务保持 pending 状态，等待服务恢复
- 当单个任务执行超过 120 秒时，自动标记为超时并重试

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: 系统必须接收飞书 Webhook 事件并解析 `/爬取 [URL]` 命令
- **FR-002**: 系统必须验证 URL 格式，仅接受 `https://mp.weixin.qq.com/s/` 开头的链接
- **FR-003**: 系统必须将任务写入 SQLite 队列，状态为 pending
- **FR-004**: 调度器必须每 15 秒轮询一次队列，获取最早的 pending 任务
- **FR-005**: 系统必须通过 Claude Code CLI 调用 Playwright MCP 抓取文章
- **FR-006**: 系统必须提取文章的标题、作者、公众号、发布时间、正文内容
- **FR-007**: 系统必须将抓取结果写入 Notion News 数据库
- **FR-008**: 系统必须通过飞书 API 发送结果卡片消息到原会话
- **FR-009**: 单个任务执行时间不得超过 120 秒，超时自动标记失败
- **FR-010**: 失败任务必须自动重试 2 次，间隔 30 秒递增

### Key Entities

- **Task**: 抓取任务，包含 id, user_id, chat_id, url, status, retry_count, result, error, timestamps
- **Article**: 文章数据，包含 title, author, account, publishTime, content, summary, url（存储在 Notion）

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 用户从发送命令到收到结果，P95 耗时 ≤ 90 秒
- **SC-002**: 任务成功率 ≥ 80%（排除文章已删除等不可抗因素）
- **SC-003**: 系统能够处理队列中 100 个待处理任务而不崩溃
- **SC-004**: 抓取的文章数据完整性 100%（标题、作者、正文必填字段不为空）
- **SC-005**: Notion 数据库写入成功率 ≥ 95%
