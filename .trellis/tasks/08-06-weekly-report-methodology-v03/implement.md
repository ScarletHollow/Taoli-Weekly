# 实施计划 — AI周报方法论 v0.3 MVP 落地

> **需求唯一来源**:`prd.md`。实施按 3 个子任务顺序推进(依赖已写入各子任务 prd.md)。

## 0. 实施目标

按依赖序完成 3 个子任务,构成可运行闭环:①精简方法论文档 → ②9 个核心 Skill → ③一期真实周报。完整深写推迟 v0.4。

## 1. 子任务实施顺序(依赖链)

```
[1] 08-06-methodology-docs-mvp   先行:提供规范(信号 JSON/流程/模板/锚点)
      ↓ (core-skills-impl 依赖其信号 JSON 与流程规范)
[2] 08-06-core-skills-impl       依赖 [1]:实现 9 个核心 Skill + 注册表
      ↓ (first-real-weekly 依赖其 Skill 可调用)
[3] 08-06-first-real-weekly      依赖 [2]:2026-W31 真实周报验证 + v0.4 迭代建议
```

各子任务独立实施、校验、验收(见各自 implement.md)。父任务负责集成交叉验收。

## 2. 各子任务实施要点(对齐 PRD 与 MVP 设计)

### [1] methodology-docs-mvp
- 产出:主文档 v0.3 + 4 精简分册(A/C/D/E)+ `skills/registry.md`(9 核心登记)+ README + archive/v0.2 归档。
- 主文档骨架:监测范围(50/25/15/10)/ 一二级名单 / S-A-B-C 分级 / 评分权重表(6 维 100 分)/ 工作流拆解模板 / 抓取节奏 / 最小观察范围 / 版本管理。
- 分册 A:信号 JSON schema + 精简状态(candidate/selected/rejected/frozen/published)+ JSONL 库 + 去重规则(内嵌,归属 D5)。
- 分册 C:全员版模板(期要点≤60 字 / 分层正文 / 桃李建议 / 5-8 条)+ 简版语言规范 + 构造示例。
- 分册 D:人机分工 + 每日 5 步(D1-D5,去重归 D5)+ 每周 4 节点(冻结/初稿/审核/发布,发布归节点 4)+ 简版 QA + 精简 checklist。
- 分册 E:6 维权重 + 每维 3 档锚点(高/中/低)+ 过滤规则 + 1 案例。
- **registry.md 登记 9 Skill**:monitor-education-companies(取代 monitor-tal)、monitor-china-models、classify-education-signal、score-taoli-relevance、verify-source-evidence、write-company-analysis、write-taoli-actions、format-all-staff-newsletter、check-source-quality。

### [2] core-skills-impl
- 实现 9 个核心 Skill(落地 `.claude/skills/` 平台技能,各含 SKILL.md:元数据 + 输入/输出 schema + 触发时机 + 写作规范)。
- **monitor-education-companies**:可配置企业清单(默认一级名单 7 家),每家执行检索产出候选信号 JSON。
- 冒烟测试:每 Skill 最小输入 → schema 合规输出。
- registry.md 登记完成。

### [3] first-real-weekly
- 时间窗 2026-W31(07-27~08-02),公开信息经 WebSearch/WebFetch 采集。
- 按 D 流程执行:D1-D5(含去重)→ 评分 → 冻结 → 生成 → QA → 审核 → 发布。
- 产出:`signals/signals-2026-W31.jsonl` + `drafts/2026-W31/draft.md` + `published/2026-W31-final.md` + `iterations-v0.4.md`(卡点≥3)。

## 3. 校验与评审门

### 3.1 三文件一致性(本实施计划完成时)
- [ ] design.md 与 prd.md 一致:精简方法论 / 9 Skill / 一期真实周报 / 去重归 D5 / 发布归节点 4 / monitor-education-companies。
- [ ] implement.md 与 prd.md / design.md 一致:子任务顺序与要点对齐。
- [ ] 三份文件对「核心 Skill 清单」「精简范围」「去重/发布归属」表述一致。

### 3.2 集成交叉验收(父任务持有)
- [ ] 主文档可导航到分册与 registry.md;D 流程步骤可引用已实现 Skill。
- [ ] 验证期周报按 C 模板 + D 流程 + 已实现 Skill 实际跑出(AC3)。
- [ ] v0.2 内容无丢失(AC1);完整深写待办已标注(AC6)。

### 3.3 评审门
- 门 1(自审):三文件一致性核对通过。
- 门 2(子代理审查):trellis-check 审查产出衔接一致性,修复问题。
- 门 3(用户审批):三件套 + 产出文档审批通过。

## 4. 风险与回退

- **文档范围漂移**:以 prd.md 为唯一需求来源,发现 design/implement 与 PRD 冲突即回写修正(本次已按此处理)。
- **跨子任务依赖延迟**:子任务 [2]/[3] 依赖先行子任务,若前置未完成则以文档契约先行推进,标注"临时实现"。
- **网络检索受限**:采集降级为"人工代采集"标注,不阻塞流程。
- **回退**:以 archive/v0.2 原稿为基准,v0.2 内容缺漏即补。

## 5. 完成判定

- [ ] 3 个子任务各自 AC 满足并归档。
- [ ] 父任务三文件(PRD/Design/Implement)一致,MVP 闭环跑通。
- [ ] 用户审批通过。
