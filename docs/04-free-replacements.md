# 04 · 免费替代矩阵（中国可跑 0 元版 = workflow-custom.json）

| 模板原配 | 免费替代 | 改动量 | 说明 |
|---|---|---|---|
| Apify Jobs actor | **Remotive API**（`https://remotive.com/api/remote-jobs?limit=8`，免 key） | +1 展开适配节点 | 字段映射：`id→String(id)`、`company_name→company`、`candidate_required_location→city`、`url→applyUrl`。⚠️ Arbeitnow 也可免 key 但被 Cloudflare 拦；要抓国内岗位可换拉勾/BOSS 的 RSS 或自建抓取 |
| Claude | **DeepSeek**（`https://api.deepseek.com/v1/chat/completions`，`deepseek-chat`） | +1 响应适配节点 | `choices[0].message.content → content[0].text`，**Merge Scores 原样未动**；jsonBody 必须 `JSON.stringify` 整串表达式 |
| Google Sheets | **本地 JSON 文件**（seen.json 台账） | Code 节点 + fs | 空台账输出 1 空 item（等效 Always Output Data，防坑#2b）；Log 节点读改写回，同时产出 log.json 快照 |
| Telegram + Gmail | **飞书群推送**（tenant_access_token + im/v1/messages 两个 HTTP 节点） | 双通道并单通道 | 要保双通道可再挂 SMTP 节点（QQ/163 邮箱授权码免费） |

## 部署步骤（0 元版）
1. 导入 `workflow-custom.json`
2. Search Settings 填 5 个占位符：`YOUR_DEEPSEEK_API_KEY` / `YOUR_FEISHU_APP_ID` / `YOUR_FEISHU_APP_SECRET` / `YOUR_FEISHU_CHAT_ID` / profile（你的简历画像）
3. 确认 seen.json 路径存在（默认 `C:\job-alerts\seen.json`，自建目录）
4. 手动跑一遍验证 → 激活定时器

## 凭证怎么拿（全免费）
- **DeepSeek**：platform.deepseek.com 注册送额度，打分这种小文本几分钱一天
- **飞书 bot**：open.feishu.cn 建自建应用 → 拿 app_id/secret → 拉进目标群 → 群 chat_id 用 API 查
- **Remotive**：免 key 直接调