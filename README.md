# MJ 提醒系统

一个基于 Cloudflare Pages 的在线提醒系统，支持多种提醒方式和循环类型，可以通过 Telegram 和企业微信接收提醒通知。

## 功能特点

- 🔔 支持多种提醒类型：单次、每周、每月、每年循环
- 📱 支持多平台通知：Telegram、企业微信
- ⏰ 灵活的时间设置：支持快速设置和自定义时间
- 🔗 支持添加相关链接
- 🌐 基于 Cloudflare Pages 的可靠部署
- 💾 使用 Cloudflare D1 数据库存储

## 系统要求

- Cloudflare 账号
- Cloudflare Pages
- Cloudflare D1 数据库
- （可选）Telegram Bot Token 和 Chat ID
- （可选）企业微信 Webhook 地址

## 数据库设置

1. 在 Cloudflare Workers and Pages 中创建 D1 数据库：

```sql
-- 创建提醒表
CREATE TABLE reminders (
    id TEXT PRIMARY KEY,
    title TEXT NOT NULL,
    content TEXT NOT NULL,
    remind_time TEXT NOT NULL,
    cycle_type TEXT NOT NULL,
    status INTEGER DEFAULT 0,
    link TEXT,
    cron_job_id INTEGER
);
```

## 环境变量配置

在 Cloudflare Workers 的环境变量中设置以下值：

- `CRON_SECRET`: 定时任务的访问密钥
- `CRONJOB_API_KEY`: cron-job.org 的 API 密钥
- `TG_BOT_TOKEN`: Telegram Bot Token（可选）
- `TG_CHAT_ID`: Telegram Chat ID（可选）
- `WECOM_KEY`: 企业微信 Webhook 地址（可选）

## 部署步骤

1. 在 Cloudflare Pages 中创建新项目
2. 连接您的 Git 仓库（GitHub、GitLab 等）
3. 在 Pages 项目设置中：
   - 绑定 D1 数据库，绑定时变量为DB
   - 配置环境变量（见环境变量配置部分）
4. 部署完成后，Pages 会自动为您生成一个域名

## 使用说明

### 添加新提醒

1. 点击页面上的"📑 添加新提醒"按钮
2. 填写提醒信息：
   - 项目名称：提醒的标题
   - 链接地址：（可选）相关的 URL
   - 项目内容：提醒的详细内容
   - 提醒时间：可以使用快捷按钮或自定义时间
   - 循环类型：选择提醒的重复方式
3. 点击"添加提醒"保存

### 快速时间设置

支持以下快捷时间设置：

- 3 分钟后
- 15 分钟后
- 30 分钟后
- 1 小时后
- 2 小时后
- 明早 9 点
- 明天中午 12 点
- 6 天后
- 29 天后
- 364 天后

### 提醒循环类型

- 单次提醒：只提醒一次
- 每周循环：在指定的每周某一天重复提醒
- 每月循环：在每月的指定日期重复提醒
- 每年循环：在每年的指定日期重复提醒

## API 接口

### 获取提醒列表

```http
GET /api/reminders
```

### 添加新提醒

```http
POST /api/reminders
Content-Type: application/json

{
    "id": "unique-id",
    "title": "提醒标题",
    "content": "提醒内容",
    "remind_time": "2023-12-31T12:00:00Z",
    "cycle_type": "once",
    "link": "https://example.com"
}
```

### 通知接口

```http
GET /api/notify?key=YOUR_CRON_SECRET&id=REMINDER_ID
```

## Cron-job.org 额度限制

使用 cron-job.org 的免费计划有以下限制：

- 最多创建 50 个定时任务
- 最短执行间隔为 1 分钟
- 每个任务的超时时间为 30 秒
- 每月可执行 10,000 次
- 每个任务最多可以设置 100 个不同的执行时间点

如果您需要更多的任务数量或执行次数，可以考虑升级到付费计划。

## 安全说明

- 所有 API 请求都需要正确的访问密钥
- 通知接口使用 CRON_SECRET 进行验证
- 建议使用 HTTPS 确保数据传输安全
- 定时任务通过 cron-job.org 的 API 进行管理

## 故障排除

1. 如果提醒没有按时发送：

   - 检查 cron-job.org 的任务状态
   - 验证环境变量配置是否正确
   - 检查数据库中的提醒状态

2. 如果通知没有收到：
   - 确认 Telegram Bot 或企业微信的配置是否正确
   - 检查网络连接状态
   - 查看 Cloudflare Workers 的日志

## 贡献指南

欢迎提交 Issue 和 Pull Request 来改进这个项目。在提交之前，请确保：

1. 代码符合现有的代码风格
2. 添加了必要的测试
3. 更新了相关文档

## 许可证

MIT License

## 密码保护

你可以通过设置环境变量来添加一个简单的密码保护功能：

1. 在 Cloudflare Pages 的项目设置中，找到 "Environment variables" 部分
2. 添加一个新的环境变量：
   - 变量名：`PASSWORD`
   - 变量值：你想设置的密码

注意事项：
- 如果不设置 PASSWORD 环境变量，网站将不会启用密码保护
- 密码验证成功后会保存 30 天，期间无需重复输入
- 这是一个基础的保护机制，主要用于防止随意访问
- 建议使用复杂密码，因为密码是明文传输的
