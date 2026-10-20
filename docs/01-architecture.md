# 01 · 逐节点拆解

## 1. Daily Schedule（scheduleTrigger）
`rule.interval: [{triggerAtHour: 9}]` —— 每天上午 9 点触发。

## 2. Search Settings（set）—— 全流配置中枢
19 个字段分四组：
- **搜索**：keywords、excludeKeywords、categories、subcategories、experienceLevels、countries（默认 `["India"]`）、city、workModes、includeWorldwideRemote
- **抓取**：maxItems=20、maxSpendUsd=1（Apify 烧钱封顶）
- **AI**：profile（简历画像，占位符提示粘贴简历摘要）、claudeModel（`claude-haiku-5-5`）、topN=5
- **投递**：sendTelegram/telegramChatId、sendGmail/gmailTo、sheetId

> 教学点：把所有魔法数字收进一个 Set 节点，是模板最值得抄的习惯。

## 3. Fetch Jobs（httpRequest → Apify）
- POST `https://api.apify.com/v2/acts/pk07007~ats-jobs-api/run-sync-get-dataset-items`
- body 用 `JSON.stringify({...})` 从 Search Settings 装配，含 `postedAfter: $now.minus({days:1})`（只要过去 24h）
- query 带 `maxTotalChargeUsd` ← maxSpendUsd —— **烧钱保险丝**
- 认证：genericCredentialType / httpHeaderAuth（Apify token）

## 4. Get Seen IDs（googleSheets 读）
读 `sheetId` 指向文档的「Jobs」表，输出全部历史行（取 `id` 列）。

## 5. Filter & Prep（code）—— 去重 + prompt 压缩
一次全量模式（runOnceForAllItems）：
```js
const seen = new Set(seenRows.map(i => String(i.json.id || '')).filter(Boolean));
const fetched = fetchedItems.map(i => i.json).filter(j => j && j.id);
// 三道闸：已见跳过 / 批内重复跳过（ids 集合）/ 无 id 已被上行 filter 丢弃
// 字段裁剪：{id,title,company,location(city||country||'Remote'),applyUrl}
// prompt：'id | title | company | location' 逐行 + 简历画像 + 打分指令
// 输出：单 item { jobs[], count, prompt, model, keywords, topN }
```
> 输出是**一个汇总 item**而不是逐岗 item——下游 If 判 `count>0`，Score 节点一次调用全打完。

## 6. Any New Jobs?（if）
`{{ $json.count }}` 数字 > 0。false → No New Jobs Message。

## 7. Score with Claude（httpRequest）
POST `https://api.anthropic.com/v1/messages`，`max_tokens = min(16000, 500 + count*40)`（按岗位数动态扩），body 直接吃 Filter & Prep 的 prompt/model。

## 8. Merge Scores（code）—— ⚠️ 致命坑所在地
解析 Claude 返回的 JSON 数组 → 按 id 合并回岗位 → 过滤未打分项 → 全灭则抛错 → 降序排序 → 打 `sent=i<topN` 标记，展开成逐岗 item 输出（补 status/loggedAt/keywords/total）。

## 9. Build Message（code）
只取 `sent=true` 行：纯文本版（Telegram，3900 字符截断 + `+N more`）+ HTML 版（Gmail，& < > 转义）+ subject。

## 10. Log to Sheet（googleSheets append）
**全部打分行**落账（含 sent=false 的），列：id/sent/score/title/reason/status/company/applyUrl/location/loggedAt。

## 11. No New Jobs Message（code）
`No new jobs today for "<keywords>".` —— count=0 分支的安慰通知。

## 12/13. Telegram/Gmail Enabled?（if）→ Send
布尔开关独立控制两通道；Telegram 发 text，Gmail 发 html+subject。