# 实施计划 — 核心 Skill 实现(core-skills-impl)

## 0. 目标

实现 9 个核心 Skill(落地为 `.claude/skills/` 平台技能),通过冒烟测试,登记于 registry.md。

> **负向约束**:去重不实现为 Skill(由 D 分册 D5 流程步骤负责);发布不实现为 Skill(由 D 分册节点 4 人工操作负责,AI 不自动发布)。本任务只实现 9 个核心 Skill。

## 1. 文件清单

```
.claude/skills/                      # 平台技能目录(复用现有 Trellis 技能目录机制)
├── monitor-education-companies/SKILL.md
├── monitor-china-models/SKILL.md
├── classify-education-signal/SKILL.md
├── score-taoli-relevance/SKILL.md
├── verify-source-evidence/SKILL.md
├── write-company-analysis/SKILL.md
├── write-taoli-actions/SKILL.md
├── format-all-staff-newsletter/SKILL.md
└── check-source-quality/SKILL.md
weekly-report/skills/registry.md     # 精简注册表(登记 9 Skill)
```

## 2. 实施清单(按依赖序)

### 阶段 0:前置
- [ ] 确认 docs-mvp 已产出 A 分册信号 JSON schema 与 D 分册流程规范(本任务契约依据)
- [ ] 创建 `.claude/skills/` 各 Skill 子目录

### 阶段 1:采集类(monitor-education-companies, monitor-china-models)
- [ ] `monitor-education-companies`:可配置企业监控。输入=企业清单(默认一级名单 7 家:好未来/新东方/猿辅导/作业帮/科大讯飞教育/希沃/网易有道)+ 时间窗;指令含每家企业检索策略(官网/官方公众号/财报/行业媒体);输出=每家候选信号 JSON(company/brand/product/title/summary/source_url/source_level/education_scenario)
- [ ] `monitor-china-models`:输入 9 模型名单 + 三类能力;指令含按能力点核对;输出能力变化信号 JSON

### 阶段 2:分析类(classify-education-signal, score-taoli-relevance, verify-source-evidence)
- [ ] `classify-education-signal`:输入候选信号;输出分类(教育业务/模型/政策/海外)+ education_scenario
- [ ] `score-taoli-relevance`:输入信号 + 桃李业务画像;按 E 分册锚点输出 6 维档位 + 总分 + 理由
- [ ] `verify-source-evidence`:输入信号 + source_url;WebSearch 回源核查;输出 source_level + 已核实结论

### 阶段 3:生成类(write-company-analysis, write-taoli-actions, format-all-staff-newsletter)
- [ ] `write-company-analysis`:输入企业 + 本期信号;输出业务层分析段(场景→变化→闭环→桃李含义)
- [ ] `write-taoli-actions`:输入本期 selected 条目;输出 2-4 条建议(动词开头 + 四类分级 + 来源条目编号)
- [ ] `format-all-staff-newsletter`:输入期元信息 + 各条目集;输出完整 newsletter markdown(C 模板)

### 阶段 4:检查类(check-source-quality)
- [ ] `check-source-quality`:输入草稿 + 信号库;输出来源质量报告(每条核心事实 ≥1 S 级,source_level 完整)

### 阶段 5:注册表 + 冒烟测试
- [x] `registry.md`:9 Skill 精简登记(编号 M-01/A-01…/G-01…/C-01/名称/分类/版本 v1.0.0/负责人)
- [x] 冒烟测试:每个 Skill 用最小输入样例跑通,输出符合 schema;测试记录存入 `weekly-report/skills/smoke-tests/`

## 3. 校验命令与评审门

### 3.1 冒烟测试
- 每个 Skill 最小输入 → 输出 schema 合规(Skill 指令中定义的输出字段齐全)
- 采集类:能否经 WebSearch 产出真实候选信号
- 评分类:能否按 E 锚点打分输出六维/总分
- 生成类:能否聚合为 newsletter 草稿

### 3.2 契约对齐
- 每个 Skill 的输入/输出字段与 A 分册信号 JSON 对齐(核对字段名/类型)
- D 分册流程步骤可引用 Skill 名

### 3.3 评审门
- 门 1(自审):冒烟测试全过,契约对齐。
- 门 2(trellis-check 子代理):审查 Skill 指令与契约一致性,修复问题。
- 门 3(用户审批):审批通过后进入 first-real-weekly。

## 4. 风险与回退

- WebSearch/WebFetch 不可用:采集类 Skill 降级为"人工采集标注",记录不阻塞。
- Skill 输出 schema 漂移:以 A 分册信号 JSON 为唯一基准回查。
- 平台技能目录冲突:检查 `.claude/skills/` 现有技能,避免命名冲突。

## 5. 完成判定

- [x] 9 个 SKILL.md 就位且可被调用
- [x] 冒烟测试记录留存且通过(weekly-report/skills/smoke-tests/README.md)
- [x] registry.md 登记完成
- [ ] 用户审批通过
