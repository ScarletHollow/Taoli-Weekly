# 方法论文档 MVP(docs-mvp)

## Goal

产出《桃李未来教育 AI 周报》方法论 v0.3 的 **MVP 文档体系**:主文档 v0.3 骨架 + 4 个精简分册(A 信号数据 / C 周报输出规范 / D 执行手册 / E 评分锚点)+ README + v0.2 原稿归档。为后续 9 个核心 Skill 实现与一期真实周报提供规范依据;完整方法论深写(27-Skill 全注册表、六态状态机全细节、6×5 全量锚点)推迟 v0.4。

## Confirmed Facts

- 父任务:08-06-weekly-report-methodology-v03(MVP 落地方向)。
- 仓库非 git 仓库,仅一个 v0.2 周报.md;`.trellis/spec/` 为前端规范不适用。
- v0.2 缺失环节(MVP 需补齐的):输出格式 / 状态流转与历史存储 / 评分锚点 / 流程↔Skill 映射 / QA gate。
- 已确认决策:读者=公司全员版(期要点≤60字/分层正文/桃李建议/每周5-8条);目录=weekly-report/;历史信号库=轻量 JSONL;版本演进=主文档版本管理章节;执行=人机协作半自动;v0.2 归档不覆盖。

## Requirements

- R1: 主文档 v0.3(骨架):监测范围(50/25/15/10)/ 一二级名单 / 信息源 S-A-B-C 分级 / 评分权重表(6 维 100 分)/ 工作流拆解模板(九步)/ 抓取节奏 / 最小观察范围 / 版本管理章节。保留 v0.2 全部有效内容。
- R2: 分册 A(信号数据):信号 JSON schema + 精简状态流转(candidate/selected/rejected + frozen/published)+ JSONL 库(signals-YYYY-WW.jsonl + .bak 备份)。去重/跨周对比规则内嵌(MVP 不独立成章完整深写)。
- R3: 分册 C(周报输出规范):newsletter 全员版模板(期要点/分层正文/桃李建议/5-8 条)+ 简版写作语言规范 + 一条构造示例条目(标注"构造示例")。
- R4: 分册 D(执行手册):人机分工界面 + 每日/每周流程(北京时区)+ 简版 QA(首期用 1-2 个核心检查项)+ 人工审核 checklist(精简版)+ AI 触发方式。
- R5: 分册 E(评分锚点):6 维权重 + 每维简版锚点(3 档:高/中/低,而非完整 5 档)+ 过滤规则 + 1 个打分案例。
- R6: README(体系导航)+ archive/v0.2-原稿.md(v0.2 全文归档)。
- R7: B 分册 Skill 注册表(`skills/registry.md`):仅登记 **9 个核心 Skill**(monitor-education-companies 可配置企业监控(默认一级名单 7 家)、monitor-china-models、classify-education-signal、score-taoli-relevance、verify-source-evidence、write-company-analysis、write-taoli-actions、format-all-staff-newsletter、check-source-quality),完整注册表推迟 v0.4(留"待 v0.4 深写"占位)。
- R8: 分册 D 明确两个归属:**去重**由「每日流程 D5」步骤负责(AI 读 dedup_key 做事件级去重,不单独立 Skill);**发布**由「每周流程 节点 4 终审发布」人工操作负责(AI 不自动发布)。简版 QA 仅用已注册检查 Skill check-source-quality,不引入 9 清单外的检查 Skill 名。

## Acceptance Criteria

- [ ] AC1: v0.2 的监测比例、一二级名单、S/A/B/C 分级、信号 JSON、100 分评分表、工作流拆解模板、抓取节奏全部保留且在 v0.3 主文档或分册中可找到,无丢失。
- [ ] AC2: 主文档可导航到 4 分册 + 精简 Skill 注册表,引用为章节标题/分册名(非 v0.2 节号)。
- [ ] AC3: 分册 A/C/D/E 各自独立版本号 + 修订记录。
- [ ] AC4: 主文档 v0.3 篇幅适合通读(相对 v0.2 不显著膨胀)。
- [ ] AC5: 完整方法论深写(v0.4 待办)在文档中明确标注为后续任务。

## Out of Scope

- 不写完整 27-Skill 注册表、六态状态机全细节、6×5 全量锚点、多层 QA(推迟 v0.4)。
- 不实现可执行 Skill(由 core-skills-impl 子任务负责)。
- 不涉及实际采集或真实周报(由 first-real-weekly 子任务负责)。

## Dependencies / Open Questions

- 本子任务先行;core-skills-impl 依赖其信号 JSON / 流程规范。
- 无阻塞问题;分册章节深度按 MVP 精简执行。
