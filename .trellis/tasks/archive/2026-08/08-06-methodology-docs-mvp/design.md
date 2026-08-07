# 设计文档 — 方法论文档 MVP(docs-mvp)

## 1. 架构总览

产出 `weekly-report/` 文档体系(主文档 v0.3 骨架 + 4 精简分册 + README + v0.2 归档)。完整方法论深写推迟 v0.4。

```
weekly-report/
├── README.md
├── 桃李未来-周报方法论主文档-v0.3.md
├── 分册/
│   ├── A-信号数据手册.md
│   ├── C-周报输出规范.md
│   ├── D-执行手册.md
│   └── E-评分锚点手册.md
├── skills/            # 精简 Skill 注册表(9 核心,占位)
│   └── registry.md
└── archive/
    └── v0.2-原稿.md
```

> B 分册完整注册表推迟 v0.4;MVP 以 `skills/registry.md` 精简登记 9 个核心 Skill。

## 2. MVP 精简策略

| 模块 | 完整版(推迟 v0.4) | MVP(本次) |
|---|---|---|
| 主文档 | 十二节完整 | 骨架:范围/名单/分级/权重表/工作流模板/抓取节奏/最小范围/版本管理 |
| A 信号数据 | 六态状态机 + merge/superseded + 对比/去重成章 | 信号 JSON schema + 精简状态(candidate/selected/rejected/frozen/published)+ JSONL 库 + 去重内嵌 |
| C 输出规范 | 九章完整模板 + 行话替换表完整版 | 模板骨架 + 简版语言规范 + 一条构造示例 |
| D 执行手册 | 十环节 + 6-check QA + 全量 checklist | 人机分工 + 周流程 + 简版 QA(1-2 核心检查)+ 精简 checklist |
| E 评分锚点 | 6×5 档锚点表 + 多案例 | 6 维权重 + 每维 3 档(高/中/低)+ 过滤规则 + 1 案例 |
| B Skill 注册表 | 27-Skill + 生命周期状态机 | `skills/registry.md` 登记 9 核心 |

## 3. 关键契约(MVP)

- **信号 JSON**:v0.2 原字段 + 精简状态流转。schema 由 A 分册定义,为 Skill 输入输出契约的数据源。
- **评分体系**:6 维 100 分制唯一口径;权重单点维护于主文档;E 分册提供 3 档锚点。
- **JSONL 库**:signals/signals-YYYY-WW.jsonl;首行 schema 元信息;.bak 备份;追加写入。
- **newsletter 模板**:期要点(≤60 字)/ 业务层+技术层正文 / 桃李建议(2-4 条)/ 附注。
- **去重归属**:由 D 分册「每日流程 D5」步骤负责(AI 读 dedup_key 做事件级去重,不单独立 Skill)。
- **发布归属**:由 D 分册「每周流程 节点 4 终审发布」人工操作负责(AI 不自动发布),动作含置 published + 留存发布版快照。
- **9 核心 Skill 契约**:采集(monitor-education-companies 可配置/默认 7 家 + monitor-china-models)/ 分析(3)/ 生成(3)/ 检查(check-source-quality);registry.md 仅登记这 9 个,QA 不引用 9 清单之外的 Skill 名。

## 4. 版本与兼容

- 分册独立版本号(A/C/D/E 各 v1.0 起);主文档版本管理章节定义修订流程。
- v0.2 归档不覆盖;非 git 仓库,JSONL 写入前 .bak 备份。
- 完整深写待办在各分册文档头标注"待 v0.4 深写"。

## 5. 权衡

- 主文档可读性优先;MVP 深度收敛,保留架构与数据契约,细节深写由真实周报驱动。
- 人机协作半自动界面由 D 分册定义。
- 跨分册引用用章节标题/分册名互链。

## 6. 演进

- 真实周报跑通后,v0.4 依据暴露问题深写各分册(依赖 first-real-weekly 的迭代建议)。
