# 实施计划 — 一期真实周报验证(first-real-weekly)

## 0. 目标

用 2026-07-27 ~ 08-02 公开信息,跑通全流程,产出验证期周报 + v0.4 迭代建议。

## 1. 前置依赖

- [ ] docs-mvp 已产出:A 信号 JSON schema、C 模板、D 流程规范、E 锚点
- [ ] core-skills-impl 已产出:9 个核心 Skill 可调用
- (若某一 Skill 缺失或不适配,记录为 v0.4 待办,用临时 prompt 补齐,不阻塞本期)

## 2. 实施清单(按 D 流程)

### 阶段 0:准备
- [ ] 创建 `weekly-report/signals/`、`drafts/2026-W31/`、`published/` 目录
- [ ] 确认时间窗 2026-W31(07-27 ~ 08-02)

### 阶段 1:采集(D1-D4)
- [ ] D1 一级企业:monitor-education-companies(覆盖好未来/新东方/猿辅导/作业帮/科大讯飞教育/希沃/网易有道)
- [ ] D2 政策:WebSearch 教育部/工信部/省市教育部门
- [ ] D3 模型:monitor-china-models(9 家能力更新)
- [ ] D4 媒体线索:二级名单触发信号 + B 级媒体(芥末堆/多知等)+ verify-source-evidence 回源核查(B 级线索回原始页面核实 source_level)

### 阶段 2:去重 + 评分(D5)
- [ ] 候选信号写入 signals-2026-W31.jsonl(带 source_url/source_level)
- [ ] classify-education-signal 分类
- [ ] **去重(由 D5 步骤负责)**:读取 dedup_key 做事件级去重(同事件合并,保留最高 source_level 行);周五冻结前对全池重跑一次
- [ ] score-taoli-relevance 初评(6 维档位 + 总分)

### 阶段 3:冻结 + 生成
- [ ] 冻结候选池(编辑确认,人)
- [ ] write-company-analysis(业务层条目)
- [ ] write-taoli-actions(桃李建议)
- [ ] format-all-staff-newsletter(聚合 draft.md)

### 阶段 4:QA + 审核 + 发布
- [ ] check-source-quality(来源质量报告)
- [ ] 人工审核(精简 checklist:来源/评分/结构/语言/建议)
- [ ] **发布(由每周流程节点 4 人工操作负责,AI 不自动发布;本期为验证样本,以文件交付,不推送真实全员渠道)**:信号置 published + 留存 published/2026-W31-final.md

### 阶段 5:v0.4 迭代建议
- [ ] 记录流程卡点(≥3 条)→ iterations-v0.4.md(按采集/去重/评分/生成/审核分类)

## 3. 校验与评审门

### 3.1 产物校验
- [ ] signals-2026-W31.jsonl:真实信号、来源完整、评分字段完整
- [ ] draft.md / published:符合 C 模板(期要点/分层正文/桃李建议/条数合理)
- [ ] iterations-v0.4.md:卡点 ≥3 条,每条含现象/原因/建议

### 3.2 交叉验收(父任务持有)
- [ ] 验证样本由 D 流程 + 已实现 Skill 实际跑出(非手工拼凑)

### 3.3 评审门
- 门 1(自审):产物校验全过。
- 门 2(用户审批):验证样本 + 迭代建议,用户确认后归档。

## 4. 风险与回退

- 信息量不足:如实记录"该企业本周无重大动态",不硬凑。
- Skill 不可用:临时 prompt 补齐,标记"临时采集/临时生成",不阻塞本期。
- 时间窗争议:若 2026-W31 信息过于稀疏,退而用 2026-W30,并在产物中注明。

## 5. 完成判定

- [ ] 验证期周报产出(published/2026-W31-final.md)
- [ ] signals JSONL 真实且合规
- [ ] iterations-v0.4.md 产出
- [ ] 用户审批通过
