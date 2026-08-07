# 设计文档 — 一期真实周报验证(first-real-weekly)

## 1. 目标

用最近完整一周的公开信息作为数据源,跑通「采集→去重→评分→生成→审核→发布」全流程,产出符合 C 模板的验证期周报,并输出 v0.4 迭代建议。

## 2. 数据源与时间窗

- **时间窗**:最近完整一周,建议 **2026-07-27(周一)~ 2026-08-02(周日)**。当前日期 2026-08-06,该周为最近完整自然周。
- **采集方式**:AI 经 WebSearch / WebFetch 采集真实公开信息(官网新闻/官方公众号/政府公告/行业媒体 S/A/B 级来源)。
- **范围**:一级名单 7 家企业(好未来/新东方/猿辅导/作业帮/科大讯飞/希沃/有道)+ 国内 9 模型 + 政策源(教育部/省市教育部门)。

## 3. 执行路径(D 分册流程)

```
D1 一级企业采集(monitor-education-companies 覆盖好未来/新东方/猿辅导/作业帮/科大讯飞教育/希沃/网易有道)
D2 政策采集(WebSearch 教育部/省市教育)
D3 模型更新采集(monitor-china-models)
D4 媒体线索扫描(二级名单 + B 级媒体)+ verify-source-evidence 回源核查(B 级线索回企业/政府原始页面核实 source_level)
D5 保存候选 + 分类 + 初评(写 signals-2026-W31.jsonl)
  → 去重(D5 步骤负责:读 dedup_key,同事件合并,保留最高 source_level 行)
  → 评分(score-taoli-relevance)
  → 冻结候选池(人工/编辑确认)
  → 生成初稿(write-company-analysis / write-taoli-actions / format-all-staff-newsletter)
  → QA(check-source-quality)
  → 人工审核(精简 checklist)
  → 发布(节点 4 人工操作:置 published + 留存 published/2026-W31-final.md)
```

## 4. 关键契约

- **信号库**:`weekly-report/signals/signals-2026-W31.jsonl`(真实信号,带 source_url/source_level/评分)。
- **周报产物**:`weekly-report/drafts/2026-W31/draft.md`(最终验证样本)与 `published/2026-W31-final.md`。
- **v0.4 迭代建议**:`weekly-report/drafts/2026-W31/iterations-v0.4.md`(流程卡点清单)。

## 5. 风险与降级

- 某周特定企业无重大动态:如实记录"已检查无新增",不硬凑(符合"比例非硬性凑数"原则)。
- 网络检索受限:记录"人工代采集"标注,不阻塞流程。
- 若 7 家 + 9 模型全周信息量过大:优先深挖最有拆解价值的企业(如好未来),其余作速览。

## 6. 验证口径

- 验证的不是"本周发生了什么",而是:**流程能否跑通 + 每步产物是否 schema 合规 + 最终周报是否符合 C 模板**。
- 流程卡点记录(≥3 条)→ 汇总为 v0.4 迭代建议,反馈 docs-mvp 深写方向。
