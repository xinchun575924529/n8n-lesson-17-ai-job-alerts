# 00 · 总览：这个工作流解决什么问题

## 场景
求职时每天刷招聘网站太累，让机器替你盯：每天上午 9 点自动抓过去 24 小时的新岗位，AI 按你的简历打分，只把最值得投的 Top N 发到你手机上，而且**同一个岗位永远不会打扰你第二次**。

## 数据流（四段式）
```
[Daily Schedule] 每天 9:00
  ↓
[Search Settings] 全部配置集中一个 Set 节点（关键词/国家/简历/模型/topN/开关/凭证位）
  ↓
[Fetch Jobs] Apify actor（pk07007~ats-jobs-api）抓岗，query 带 maxTotalChargeUsd=1 烧钱封顶
  ↓
[Get Seen IDs] 读 Google Sheet「Jobs」表拿历史 id
  ↓
[Filter & Prep] Code：三道闸去重（已见/批内重复/无 id）+ 压缩 prompt + 输出 {jobs,count,prompt,model,keywords,topN}
  ↓
[Any New Jobs?] If：count > 0？
  ├─ 是 → [Score with Claude] → [Merge Scores] 展开成逐岗行（sent=前topN）→ [Build Message] 文案 / [Log to Sheet] 台账
  └─ 否 → [No New Jobs Message]「今天没有新岗位」
  ↓
[Telegram Enabled?] → [Send Telegram]   （开关独立）
[Gmail Enabled?]    → [Send Gmail]      （开关独立，Gmail 发 HTML 版）
```

## 设计亮点（值得抄的设计）
- **配置集中**：所有可调参数（关键词、国家、简历、模型、topN、通道开关、chatId、sheetId）全在 Search Settings 一个节点里，学员只改一处。
- **烧钱封顶**：Apify query 参数 `maxTotalChargeUsd` 直接来自配置（默认 $1），跑飞了也烧不了多少。
- **topN 与台账分离**：Merge Scores 给每行打 `sent=i<topN` 标记——消息只发 Top N，**台账记全部**。低分岗位也留痕，方便复盘调整画像。
- **打分全灭即抛错**：Claude 一条有效打分都没返回时，Merge Scores 主动抛错终止——这些岗位因未落账，明天会自动重试，而不是静默丢失。
- **TG 3900 字符截断**：Build Message 超长整条丢弃 + `+N more` 收尾，不会发出半行残文。

## 模板的「身份证」
- 名称：Send daily AI-scored job alerts with Apify Jobs, Claude, Google Sheets, Telegram and Gmail
- 模板 ID：20683（⚠️ 2026-10 中旬已从官方库下架；本课程 workflow.json 为下架前完整镜像）
- 节点：15 功能节点 + 6 张教学便签；凭证需求：Apify token + Anthropic key + Google Sheets + Telegram/Gmail