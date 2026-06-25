---
name: resume-screening
description: 根据 JD 对多份候选人简历进行批量筛选、可解释评分、排序和 Markdown 报告生成。Use when the user asks to screen resumes, rank candidates, compare resumes against a job description, create a shortlist, or produce a resume evaluation report from JD plus PDF/DOCX resumes.
---

# 简历筛选

## 工作流

1. 确认用户提供了一个 JD 文件和一个简历目录或简历文件列表。
2. 读取 JD。Markdown 和 TXT 直接读取文本；如果用户提供其他格式，先说明首期支持范围并请求转换。
3. 读取简历。PDF 使用当前 agent 环境可用的 PDF 文本抽取能力；DOCX 使用当前 agent 环境可用的文档读取能力。
4. 如果单份简历无法解析，记录文件名、失败原因和“需要人工复核”，继续处理其他简历。
5. 从 JD 提取硬性要求、岗位职责、加分项、经验年限、技术栈、业务领域、语言和其他约束。
6. 从每份简历提取候选人姓名、联系方式线索、当前职位、年限、教育、公司经历、项目经历、技能、成果和风险点。
7. 读取 `references/scoring-rubric.md`，按 100 分制逐项评分。
8. 读取 `references/report-template.md`，生成 Markdown 报告。
9. 如果用户未指定输出路径，将报告写入 `recruit-assist-output/resume-screening-report.md`。

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
