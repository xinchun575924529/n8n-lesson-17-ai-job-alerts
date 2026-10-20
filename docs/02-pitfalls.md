# 02 · 坑集（全部实锤，按严重度排序）

## 坑#1【致命】Merge Scores 正则写错一个反斜杠，模板开箱即死
- **现场**：原文 `txt.match(/[[sS]*]/)` —— 作者想写 `/\[[\s\S]*\]/`（匹配 JSON 数组），但两个反斜杠都丢了。`[[sS]*]` 的语义变成「由 `[` `s` `S` `]` 组成的字符序列」，Claude 的正常输出（含大量字母数字）**永远匹配不到**。
- **后果链**：match 返回 null → `null[0]` 抛 TypeError → 被 catch 吞掉 → scores 为空 → rows 为空 → 主动抛 `Claude returned no readable scores...` → **工作流 error 终止，什么都不发**。
- **实锤方式**：L2-A B01 用完全健康的 Claude 响应喂原样代码，必抛错；L2-B O1 场景全链 mock 数据完全健康，工作流照样必死。
- **修复**：`/[[sS]*]/` → `/\[[\s\S]*\]/`（一个反斜杠之差，生死之别）。
- **教训**：交付前必须用「数据完全健康」的路径跑一遍——本模板作者大概率没跑过端到端。

## 坑#2【中】0 item 断链，两处，首跑必中其一
n8n 的铁律：**节点输出 0 个 item，后续节点根本不执行**（不是传空，是直接断）。
- **a) 抓 0 条**：某天没有新岗位发布 → Fetch Jobs 输出 0 item → 链路在 Fetch Jobs 静默终结，连「No new jobs today」的安慰通知都发不出去（L2-B S3 实锤：`lastNodeExecuted=Fetch Jobs`，status=success，零投递零报错——你以为它在正常工作）。
- **b) 台账为空（=任何人第一次运行！）**：Sheet 建好了但一行都没有 → Get Seen IDs 输出 0 item → Filter & Prep 不执行 → 新岗位明明抓到了也发不出去。
- **修复**：Sheets 读节点开 **Always Output Data**（空表也输出 1 个空 item，Filter & Prep 的 `String(i.json.id||'')` 会把它过滤掉，语义无损）；HTTP 抓取节点后加「空则补位」逻辑或接受静默（至少要在文档里讲清楚）。
- **教训**：凡是「读表/查接口」类节点后面挂着关键链路，都要想一遍「返回 0 条时会怎样」。

## 坑#3【低，设计如此但要懂】漏打分岗位静默丢弃
- Claude 只回了 3 岗中 1 岗的分数 → 另外 2 岗不进消息、**也不落台账**（L2-B S5 实锤 merged 3→1）。
- 这是作者的容错设计：不落账 = 明天重跑会重试，数据不丢，只是延迟一天。但学员必须知道「漏打 = 静默延迟」，排查时别误判为丢数据。

## 坑#4【低】No New Jobs Message 的 HTML 转义不全
只转义了 `&` 和 `<`，没转义 `>`（L2-A D04 实锤）。keywords 含 `>` 时 Gmail 版 HTML 有瑕疵。低危，知道即可。

## 中国落地时新踩的坑（L2-C 实录）
5. **免费职位 API 选型要查反爬**：首选 Arbeitnow 被 Cloudflare 人机校验拦死（403 challenge 页），换 Remotive（加 UA 头直连可用）。
6. **HTTP 节点 jsonBody 不能裸嵌多行文本**：`"content":$json.prompt` 里 prompt 含换行 → `The value in the "JSON Body" field is not valid JSON`。必须整串表达式 `={{ JSON.stringify({...}) }}`。
7. **Code 节点（task-runner 沙箱）里没有 `$helpers`**：`$helpers is not defined`——需要发 HTTP 请求就老老实实用 HTTP Request 节点。
8. **DeepSeek 兼容性**：模板原 prompt（"Reply with ONLY a JSON array..."）直接可用，8 岗全打分无漏项，无需 prompt 改造。