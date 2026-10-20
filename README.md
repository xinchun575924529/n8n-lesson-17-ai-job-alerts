# n8n 实战课 16：AI 岗位雷达（AI-Scored Job Alerts）

> 官方模板：[Send daily AI-scored job alerts with Apify Jobs, Claude, Google Sheets, Telegram and Gmail](https://n8n.io/workflows/20683/)（模板 ID 20683，⚠️已于 2026-10 中旬从官方库下架，本课程 `workflow.json` 为下架前完整镜像，15 功能节点 + 6 张教学便签）

每天定时抓取过去 24 小时新发布的岗位，对照台账去重，AI 按你的简历逐岗打 0-10 分，Top N 推送到你的聊天工具，全部打分回写台账——一个永远不会重复打扰你的求职雷达。

## 你会学到

1. **「采集→去重→AI 打分→推送」四段式管道**：定时器 + 职位 API + 台账 + LLM + 双通道通知的标准范式
2. **台账驱动去重**：Sheet/JSON 即记忆体，已处理岗位永不重复推送，打分失败的岗位靠「不落账」自动获得下轮重试
3. **prompt 工程实战**：把岗位列表压成 `id | title | company | location` 紧凑格式，一次调用换全量打分
4. **topN 与台账分离设计**：发几条（topN）和记几条（全部打分）互不干扰
5. **三大实锤坑**：本模板是我们解剖过「看着最能跑、实际开箱即死」的模板之一（见 docs/02-pitfalls.md）

## 课程文件

| 文件 | 内容 |
|---|---|
| `workflow.json` | 官方模板原样镜像（下架前存档，含原始缺陷） |
| `workflow-custom.json` | 中国可跑免费替代加固版（Remotive + DeepSeek + 本地 JSON 台账 + 飞书推送，已脱敏） |
| `docs/00~05` | 总览 / 结构 / 坑集 / 验证报告 / 免费替代 / 生产加固 |
| `exercises/exercise.md` | 3 道动手练习（含答案） |
| `script/short-video.md` | 60 秒获客短视频口播稿 |

## 三层验证结论（本课全部实测通过）
- **A 单元级**：4 个 Code 节点忠实移植，**35/35 断言全过**（含坏正则实锤）
- **B mock 全链**：8 场景（正常/全已见/0 抓取/AI 乱说/部分打分/topN 截断/双通道关闭/原样必死），**45/45 断言全过**
- **C 真跑**：Remotive 真抓 8 岗 + DeepSeek 真打分 8/8 + 飞书群真投递 + 二轮真去重，**两轮 E2E PASS**

## 一分钟看懂它干啥
```text
每天 9:00
  → Apify/Remotive 抓过去 24h 新岗位（烧钱封顶 $1）
  → 对照台账：已见/批内重复/无 id 三道闸去重
  → AI 按你的简历逐岗打 0-10 分 + 一句话理由
  → Top N 推送 Telegram/飞书，全部打分回写台账
```