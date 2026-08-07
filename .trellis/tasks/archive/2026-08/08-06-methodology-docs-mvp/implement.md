# 实施计划 — 方法论文档 MVP(docs-mvp)

## 0. 目标

产出 `weekly-report/` MVP 文档体系(README + 主文档 v0.3 + 4 精简分册 + skills/registry.md + archive/v0.2 归档)。

## 1. 文件清单

```
weekly-report/
├── README.md
├── 桃李未来-周报方法论主文档-v0.3.md
├── 分册/A-信号数据手册.md
├── 分册/C-周报输出规范.md
├── 分册/D-执行手册.md
├── 分册/E-评分锚点手册.md
├── skills/registry.md
└── archive/v0.2-原稿.md
```

## 2. 实施清单

### 阶段 0:准备工作区
- [ ] 创建 `weekly-report/`、`分册/`、`skills/`、`archive/` 目录
- [ ] 复制 v0.2 到 `archive/v0.2-原稿.md`(不覆盖原文件)

### 阶段 1:主文档 v0.3
- [ ] 章节:零 文档导航与分册索引 / 一 监测范围(50/25/15/10 + 六问)/ 二 一级名单(A1-A7)/ 三 二级名单 + 7 触发 / 四 国内模型 9 家 + 三类能力 / 五 政策源 / 六 信息源 S-A-B-C 分级 / 七 评分权重表(6 维 100 分 + 分级 + 强制进入)/ 八 工作流拆解模板(九步)/ 九 抓取节奏(v0.2《第一版建议抓取节奏》全文)/ 十 最小观察范围 / 十一 版本管理
- [ ] 校验:对照 v0.2 逐节无丢失

### 阶段 2:分册 A(信号数据)
- [ ] 信号 JSON schema(含 source_url 数组、评分组、精简状态)+ 差异对照表
- [ ] 精简状态流转:candidate → selected/rejected;冻结 frozen;发布 published
- [ ] JSONL 库:signals-YYYY-WW.jsonl、首行 schema 元信息、.bak 备份、追加规则
- [ ] 去重内嵌:dedup_key 构造 + 合并规则(简版)

### 阶段 3:分册 C(输出规范)
- [ ] newsletter 模板:期元信息 / 期要点(≤60 字)/ 业务层+技术层正文 / 桃李建议(2-4 条)/ 附注
- [ ] 简版语言规范:术语替换表(首批 5-8 条)+ 首句结论规则
- [ ] 一条构造示例条目(标注"构造示例,非本期真实数据")

### 阶段 4:分册 D(执行手册)
- [ ] 人机分工界面(采集/去重/初评 AI;冻结/审核/发布 人)
- [ ] 每周时间轴(北京时区:周五 14:00 冻结 / 周五 19-21 初稿 / 周一 09:30 审核 / 周一 12:00 发布)
- [ ] 每日 5 步(D1-D5)+ 简版 QA(首期用已注册检查 Skill check-source-quality,可读性检查并入「简版语言规范」非独立 Skill)
- [ ] 精简审核 checklist + AI 触发方式
- [ ] **去重归属**:明确去重由「每日流程 D5」步骤负责(AI 读 dedup_key 做事件级去重,不单独立 Skill)
- [ ] **发布归属**:明确发布由「每周流程 节点 4 终审发布」人工操作负责(AI 不自动发布)

### 阶段 5:分册 E(评分锚点)
- [ ] 6 维权重 + 每维 3 档锚点(高/中/低,各含判据 + 正反例)
- [ ] 过滤规则(融资/口号/品牌宣传)+ 强制进入
- [ ] 1 个打分案例(高价值 / 过滤对照)

### 阶段 6:skills/registry.md + README
- [ ] registry.md:9 个核心 Skill 精简登记(编号/名称/分类/版本/负责人)——monitor-education-companies(可配置企业监控,默认一级名单 7 家)、monitor-china-models、classify-education-signal、score-taoli-relevance、verify-source-evidence、write-company-analysis、write-taoli-actions、format-all-staff-newsletter、check-source-quality
- [ ] README:体系导航 + 阅读顺序 + 快速上手

### 阶段 7:收口校验
- [ ] 跨分册链接为章节标题/分册名(无 v0.2 节号)
- [ ] 各分册文档头独立版本号 + 修订记录 + "待 v0.4 深写"标注
- [ ] 对照 archive/v0.2 逐节核验无丢失(AC1)

## 3. 校验与评审门

- 门 1(自审):按实施清单逐项核对,对照 v0.2 逐节无丢失。
- 门 2(子代理审查):trellis-check 审查体系衔接一致性,修复问题。
- 门 3(用户审批):PRD/design/implement + 产出文档审批通过后进入 core-skills-impl。

## 4. 风险与回退

- 主文档膨胀:执行性内容下沉 D 分册。
- 跨分册重复:收敛到单一权威源。
- v0.2 内容丢失:以 archive 原稿为基准逐节核查;缺漏即补。

## 5. 完成判定

- [ ] weekly-report/ 全部 8 文件就位
- [ ] AC1-AC5 满足
- [ ] 用户审批通过
