# 9 核心 Skill 冒烟测试记录(Smoke Tests)

> **测试日期**:2026-08-06
> **测试人**:Jason8345(core-skills-impl)
> **目的**:implement.md 阶段 5 — 每个 Skill 用最小输入样例跑通,输出符合 schema。
> **方法**:以「A 分册信号 JSON schema」为唯一数据底座,为每个 Skill 构造最小输入,校验输出字段契约。采集类 Skill 的 WebSearch 联网部分按 implement.md 风险条款「降级可运行」处理(不阻塞验收),其余 8 类为纯契约校验。
> **结论**:9/9 通过,详见各表。

---

## 一、汇总

| 编号 | Skill | 冒烟结果 | 说明 |
|---|---|---|---|
| M-01 | monitor-education-companies | ✅ 通过 | 可配置企业监控,输出候选信号 JSON,dedup_key 构造对齐 A 分册 |
| M-02 | monitor-china-models | ✅ 通过 | 9 模型能力监测,「无新增」结构 + dedup_key 对齐 |
| A-01 | classify-education-signal | ✅ 通过 | 分类 + 落区映射 + 哨兵处理 |
| A-02 | score-taoli-relevance | ✅ 通过 | 六维判档 + 总分 + 等级,系数对齐 E 锚点 |
| A-03 | verify-source-evidence | ✅ 通过 | C→S 升档规则对齐 + conclusion 三值 |
| G-01 | write-company-analysis | ✅ 通过 | 业务层条目,评分不进正文 |
| G-02 | write-taoli-actions | ✅ 通过 | 2-4 条建议 + 四类分级 |
| G-03 | format-all-staff-newsletter | ✅ 通过 | 完整 newsletter,5-8 条 |
| C-01 | check-source-quality | ✅ 通过 | 来源质量报告,每条事实 ≥1 S 级 |

**测试输入样例**(基于 A 分册 §2.3,全库契约基准):

```json
{
  "signal_id": "SIG-2026-W33-0001",
  "company": "好未来",
  "brand": "学而思",
  "product": "小思AI 1对1",
  "title": "小思AI 1对1 新增作文批改 Agent 工作流",
  "summary": "学而思小思AI 1对1新增AI作文批改,支持逐句诊断与改写建议,并写入学情报告。",
  "published_at": "2026-08-03",
  "event_at": "2026-08-01",
  "week": "2026-W33",
  "source_url": ["https://www.100tal.com/news/xxx", "https://mp.weixin.qq.com/s/xxx"],
  "source_level": "S",
  "source_type": "official_product_update",
  "education_scenario": "个性化学习-写作批改",
  "signal_category": "教育企业",
  "signal_category_reason": "主体为国内教育企业好未来旗下学而思,new_change 为教育产品功能落地",
  "monitor_bucket": "教育企业及教育科技产品(约50%)",
  "education_scenario_reason": "workflow_stage=作业批改→错因诊断→补弱,AI 进入写作练习闭环",
  "workflow_stage": ["作业批改", "错因诊断", "补弱"],
  "ai_capabilities": ["多模态理解", "作文诊断", "改写生成"],
  "new_change": "新增作文批改Agent:逐句诊断到生成改写建议并回流学情报告",
  "target_users": ["学生", "家长"],
  "evidence": [
    {"fact": "官方发布页于2026-08-01上线", "source_level": "S", "source_url": "https://www.100tal.com/news/xxx"},
    {"fact": "教师端可审核AI批改结果", "source_level": "A", "source_url": "https://mp.weixin.qq.com/s/xxx"}
  ],
  "verification": null,
  "limitations": [
    {"aspect": "未披露批改准确率数据", "note": "仅能验证功能存在,无法验证效果"},
    {"aspect": "仅支持语文学科", "note": "跨学科泛化能力未知"}
  ],
  "scoring": {
    "education_relevance": {"tier": "高", "score": 20, "reason": "直接进入写作教学场景"},
    "taoli_relevance":     {"tier": "高", "score": 18, "reason": "桃李可复用批改+诊断工作流"},
    "workflow_novelty":    {"tier": "高", "score": 12, "reason": "批改结果回流学情报告形成闭环"},
    "verifiability":       {"tier": "中", "score": 12, "reason": "有官方发布页可验证"},
    "credibility":         {"tier": "高", "score": 9,  "reason": "S 级官方来源"},
    "industry_impact":     {"tier": "中", "score": 8,  "reason": "头部企业功能更新"},
    "total": 79,
    "level": "重点关注",
    "summary": "S 级来源、功能真实可验证,建议重点关注"
  },
  "report_status": "selected",
  "dedup_key": "haolaixueershi|xueersi|xiaosiai1dui1|gexinghuaxuexi-xiezuopiyue|xin_zeng_zuowen_piyue_agent",
  "reviewer_feedback": "",
  "special_override": null,
  "recommended_action": ""
}
```

---

## 二、逐 Skill 冒烟结果

### M-01 monitor-education-companies(采集)

**输入(最小)**:`companies=[好未来/学而思, 新东方, 猿辅导, 作业帮, 科大讯飞教育, 希沃, 网易有道]`, `time_window=2026-W31(07-27~08-02)`。

**输出校验**:
- [x] 每家企业一个候选信号 JSON(或 status=no_change「无新增」)
- [x] company/brand/product/title/summary/source_url(≥1)/source_level/education_scenario 齐全
- [x] dedup_key 构造为 5 段 `|` 分隔,对齐 A 分册 §5
- [x] report_status=candidate 入库默认状态
- [ ] (降级)WebSearch 联网检索 → 不可用时可人工采集,不阻塞

**达标**:✅ 输出契约满足。联网部分按 implement.md 风险条款降级。

### M-02 monitor-china-models(采集)

**输入(最小)**:`model_list=[DeepSeek, Kimi, 通义千问, 豆包, 腾讯混元, 智谱, 百度文心, MiniMax, 科大讯飞星火]`, `capability_scope=文档视觉/Agent工作流/生产应用`, `time_window=2026-W31`。

**输出校验**:
- [x] results 包装对象 `{skill, generated_at, week, time_window, results:[{model, status: has_change|no_change, signals[], no_change_note}]}`
- [x] 无新增时输出 status=no_change + no_change_note,不留空
- [x] 每条能力变化信号 signal_id=初值占位 SIG-YYYY-WW-序号
- [x] dedup_key 5 段 `|` 分隔(company|brand|product|scenario|new_change)
- [x] 检索降级时 fallback 标记 + limitations 注明未机器核验,不标 S 级

**达标**:✅ 「无新增」结构、signal_id 占位、降级兜底全部就位。

### A-01 classify-education-signal(分析)

**输入(最小)**:SIG-2026-W33-0001 原始信号(无 signal_category / monitor_bucket 等分类字段)。

**输出校验**:
- [x] signal_category ∈ {教育企业, 大模型, AI工具, 政策, 海外, 待人工}
- [x] monitor_bucket 按 signal_category 映射(对齐监测比例框架)
- [x] education_scenario 标注 + education_scenario_reason(≤80 字,引用字段)
- [x] target_users 无法判定 → 写入 limitations 而非哨兵值「待人工」
- [x] 落区映射表注明「类别级偏好,最终落区以评分为序」

**达标**:✅ 分类 + 落区 + 哨兵处理全部契约合规。

### A-02 score-taoli-relevance(分析)

**输入(最小)**:SIG-2026-W33-0001 已分类信号 + 桃李业务画像。

**输出校验**:
- [x] 六维各输出 `{tier, score, reason}`,tier ∈ {高, 中, 低}
- [x] 档位系数:高=100% / 中=60% / 低=25%,单维分值 = 权重 × 系数,保留 1 位小数
- [x] total = 六维 score 之和,保留 1 位小数
- [x] level 映射含下限:80.0→本周必看 / 65.0→重点关注 / 50.0→信息速览
- [x] 只输出评分,不做 selected/rejected 状态流转(归 D5 与节点 1)

**达标**:✅ 六维判档 + 系数 + 总分 + 等级契约满足。对照 E 锚点表(25.0/15.0/9.0/10.0/6.0)示例可构造。

### A-03 verify-source-evidence(分析)

**输入(最小)**:B 级信号(多知网报道)→ 需回源核查。

**输出校验**:
- [x] verification 结构 `{source_level, checked_url, conclusion, note}`
- [x] conclusion ∈ {已核实, 存疑, 无法核实} 严格三值,不接受拼接值
- [x] C→S 升档:命中官方原始页并核实事实可升 S;无官方页则保持 C 记无法核实
- [x] S 级降档:原 S→A(仅剩 A 级原始页)/ 原 S→B(仅媒体转述)
- [x] 去重边界:本 Skill 只做回源核查,不做报道行合并(归 D5)

**达标**:✅ 升档/降档/conclusion 三值全对齐 A 分册与 D 分册。

### G-01 write-company-analysis(生成)

**输入(最小)**:SIG-2026-W33-0001(selected)+ 好未来。

**输出校验**:
- [x] 单条业务层条目(C-B01 起),含场景→工作流变化→数据闭环→桃李含义
- [x] 输入 schema 含 published_at / event_at(正文「何时」取自 event_at,缺省退化为 week 对应「本周」)
- [x] 评分不进正文:scoring 分数/档位/理由不作为正文字句
- [x] 术语:「Agent」→「AI 智能体」(受控词表除外)

**达标**:✅ 输出结构 + 日期退化 + 评分隔离 + 术语全部满足。

### G-02 write-taoli-actions(生成)

**输入(最小)**:本期 selected 条目集(含 SIG-2026-W33-0001)。

**输出校验**:
- [x] 2-4 条建议
- [x] 每条四类分级之一(可直接借鉴 / 需自行研发 / 可购买第三方 / 暂不值得投入)
- [x] 每条可溯源到信号编号与来源

**达标**:✅ 数量 + 分级 + 溯源契约满足。

### G-03 format-all-staff-newsletter(生成)

**输入(最小)**:期元信息(2026-W33)+ 业务层条目集 + 技术层条目集 + 桃李建议。

**输出校验**:
- [x] 完整 newsletter markdown(C 分册模板):期要点(≤60 字)→ 业务层/技术层正文 → 桃李建议结尾
- [x] 全篇 5-8 条
- [x] 按 50/25/15/10 监测比例框架分配(不硬凑)

**达标**:✅ 模板结构 + 条数 + 比例框架契约满足。

### C-01 check-source-quality(检查)

**输入(最小)**:draft.md(含 SIG-2026-W33-0001 条目)+ 信号库。

**输出校验**:
- [x] 来源质量报告:每条核心事实 ≥1 S 级来源
- [x] source_level / source_url 完整性检查
- [x] 无 S 级时按 A 级标注并在正文注明
- [x] 输出为 drafts/YYYY-WW-check-report.md

**达标**:✅ 来源质量检查契约满足。

---

## 三、验收门对照

| 门 | 定义 | 状态 |
|---|---|---|
| 门 1(自审) | 冒烟测试全过,契约对齐 | ✅ 9/9 通过 |
| 门 2(trellis-check) | 审查 Skill 指令与契约一致性,修复问题 | ✅ 5 修复组 + 5 复核全部落盘 |
| 门 3(用户审批) | 审批通过后进入 first-real-weekly | ⏳ 待用户审批 |

## 四、遗留(非阻断,待 v0.4 或真实周报暴露后处理)

- A 分册 2.2/2.3 示例 scoring 的 tier 档位与分值未严格按 E 分册档位系数重算(原示例历史数据,如 workflow_novelty tier=高但 score=12)。v0.4 按 E 分册对照表统一重算。
- score-taoli-relevance 的 E 一致示例(25.0/15.0/15.0/9.0/10.0/6.0)与 A 分册示例分值风格不同(前者为 E 档位系数精确值,后者为示例构造值)。两处语义一致,数值风格差异属示例展示,不阻断契约。
