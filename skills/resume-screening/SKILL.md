---
name: resume-screening
description: 根据 JD 对多份候选人简历进行批量筛选、可解释评分、排序和 Markdown 报告生成。Use when the user asks to screen resumes, rank candidates, compare resumes against a job description, create a shortlist, or produce a resume evaluation report from JD plus resumes in flexible file or text formats.
---

# 简历筛选

## 输入读取规则

- 支持文件路径、目录、粘贴文本或用户在对话中明确标注的内容；批量简历场景可接收简历目录、文件列表或文本列表。
- Markdown、TXT、纯文本和粘贴内容直接读取；PDF、DOCX 和其他可读文本格式优先使用当前 agent 环境可用能力抽取文本。
- 不要求用户预先转换格式；仅在无法读取、抽取结果明显不可用或缺少必要内容时，记录来源和失败原因，并请求用户补充内容或转换格式。

## 输出路径规则

- 用户指定输出路径时，优先使用用户指定路径。
- 用户未指定输出路径且 JD 来源是文件路径时，将报告写入 JD 文件同目录。
- 默认文件名使用 `resume-screening-report-{yyyyMMddHHmmss}.md`，时间戳为报告完成时间，避免同一 JD 多次筛选时覆盖历史结果。
- 如果 JD 不是文件路径或无法确定同目录，将报告写入 `recruit-assist-output/resume-screening-report-{yyyyMMddHHmmss}.md`。

## 工作流

1. 确认用户提供了一个 JD 来源和一个简历来源。
2. 按“输入读取规则”读取 JD 和简历。
3. 如果 JD 或单份简历无法解析，记录来源、失败原因和“需要人工复核”；单份简历失败时继续处理其他简历。
4. 从 JD 提取硬性要求、岗位职责、加分项、经验年限、技术栈、业务领域、语言和其他约束。
5. 从每份简历提取候选人姓名、联系方式线索、当前职位、年限、教育、公司经历、项目经历、技能、成果和风险点。
6. 读取 `references/scoring-rubric.md`，按 100 分制逐项评分。
7. 读取 `references/report-template.md`，生成 Markdown 报告。
8. 按“输出路径规则”写入报告。

## 评分要求

- 只根据 JD 和简历中可见证据评分。
- 不臆造候选人经历；没有证据的信息写“未在简历中找到”。
- 不基于年龄、性别、婚育、民族、宗教等敏感或无关属性评分。
- JD 中如包含疑似不合规筛选条件，在报告的“风险与复核项”中提示人工复核。
- 每个分项分数必须有简短证据或扣分原因。
- 总分相同的候选人，优先排列硬性要求匹配分更高者。

## 输出要求

- 输出 Markdown。
- 包含岗位摘要、排名总览、候选人明细、风险与复核项、建议面试名单。
- 候选人明细中包含分项分数、总分、证据摘要、主要缺口、建议动作和后续面试验证问题。
- 对解析失败或证据不足的简历降低置信度，不从报告中静默删除。
