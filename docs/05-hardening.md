# 05 · 生产加固清单（把模板从「能跑」做到「敢用」）

## 必修（不修就是坏的）
- [ ] **Merge Scores 正则**：`/[[sS]*]/` → `/\[[\s\S]*\]/`（坑#1，不修开箱即死）
- [ ] **Get Seen IDs 开 Always Output Data**：否则空台账首跑静默断链（坑#2b）

## 建议（体验与可观测）
- [ ] **0 抓取告警**：Fetch Jobs 后加 If「结果数为 0 → 发一条调试通知」，把坑#2a 的静默变可见
- [ ] **打分覆盖率监控**：Merge Scores 后比较 `merged.length` vs `prep.count`，低于 100% 时在消息尾标注「N 岗延迟到明天打分」（把坑#3 从静默变显式）
- [ ] **DeepSeek max_tokens 对齐**：模板按 `500 + count*40` 算 Claude 额度，DeepSeek 同样适用；岗位多时注意 8K 输出上限
- [ ] **失败重试**：Score 节点开 n8n 节点级 retryOnFail（2 次，间隔 5s），网络抖动不炸整条链
- [ ] **台账快照**：每周把 seen.json / Sheet 导出备份一次，防误清
- [ ] **成本控制**：DeepSeek 打分每次约 2-4K token；Apify 付费用户保留 `maxTotalChargeUsd` 保险丝
- [ ] **profile 脱敏**：简历画像会进 prompt 发给 LLM 厂商，别放身份证号/现公司等敏感信息

## 可选扩展
- [ ] 多关键词轮询：splitInBatches 循环多组 keywords，合并后再去重
- [ ] 分数趋势：台账加周报节点，看自己的市场行情变化
- [ ] 双向交互：飞书卡片按钮「不感兴趣」→ 写回台账 status 列，训练画像