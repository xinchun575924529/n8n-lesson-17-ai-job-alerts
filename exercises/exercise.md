# 练习（3 道，含答案）

## 练习 1：定位致命正则
模板 Merge Scores 节点里 `const arr = JSON.parse(txt.match(/[[sS]*]/)[0]);` 为什么会让整个工作流在数据完全健康时也必死？写出修复后的代码行。

<details><summary>答案</summary>

`/[[sS]*]/` 不是「匹配 `[...]` 包裹的任意字符」，而是字符集 `[[sS]` + 量词 `*` + 字面 `]`——即「由 `[`、`s`、`S` 组成的若干字符后跟一个 `]`」。Claude 的正常 JSON 输出含大量字母数字，匹配失败返回 null，`null[0]` 抛 TypeError，被外层 catch 吞掉 → scores 空 → rows 空 → 主动抛 `Claude returned no readable scores` → 工作流 error 终止。

修复：
```js
const arr = JSON.parse(txt.match(/\[[\s\S]*\]/)[0]);
```
</details>

## 练习 2：0 item 断链排查
学员反馈：「工作流每天都跑成功（绿色对勾），但我一周没收到任何消息，台账也没有新行。」已知 Sheet 里**一行数据都没有**。根因是什么？怎么修？

<details><summary>答案</summary>

根因：Sheet 为空 → Get Seen IDs 输出 0 个 item → n8n 规则「0 item 则下游节点不执行」→ Filter & Prep 及之后全部不执行 → 执行结果显示 success（没有节点报错），但什么都没发生。

修复：Get Seen IDs 节点勾选 **Always Output Data**（空表输出 1 个空 item）；Filter & Prep 的 `String(i.json.id||'')` + `filter(Boolean)` 会自动把这个空行滤掉，语义无损。同理要警惕 Fetch Jobs 返回 0 条的情况（链路断在更早位置，连 No New Jobs 通知都发不出）。
</details>

## 练习 3：topN 与台账的关系
设 topN=2，某天有 4 个新岗位打分 10/9/8/7。问：① 推送消息里有几岗？② 台账新增几行？③ 如果 Claude 只给 4 岗中的第 1 岗打了分，明天重跑时哪些岗位会重新打分？为什么？

<details><summary>答案</summary>

① 2 岗（10 分和 9 分，Merge Scores 打 `sent=i<topN` 标记，Build Message 只取 sent=true）。
② 4 行——台账记**全部打分行**（含 sent=false 的 8 分、7 分岗）。
③ 4 岗中只剩 3 岗会重新打分：第 1 岗已落台账（在 seen 集合里被 Filter & Prep 滤掉）；没被打分的 3 岗没落账，明天会被当作「新岗位」重新进入打分流程。这就是「漏打不丢数据、只延迟一天」的容错设计。
</details>