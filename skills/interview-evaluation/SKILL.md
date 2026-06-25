---
name: interview-evaluation
description: 基于 JD、候选人简历和面试官记录生成面试评价、分项打分和录用建议。Use when the user asks to evaluate an interview, summarize interview notes, score a candidate after an interview, or produce interviewer feedback from JD plus resume plus messy TXT interview notes.
---

# 面试评价

## 工作流

1. 确认用户提供了 JD 文件、指定候选人简历和面试过程记录。
2. 读取 JD。Markdown 和 TXT 直接读取文本；其他格式请求用户转换。
3. 读取指定简历。PDF 使用当前 agent 环境可用的 PDF 文本抽取能力；DOCX 使用当前 agent 环境可用的文档读取能力。
4. 读取面试过程记录。TXT 和 Markdown 直接读取；如果记录格式凌乱、存在错别字或口语化表达，先做语义整理，不改变候选人原意。
5. 提取 JD 的岗位目标、核心职责、硬性要求、加分项和关键能力。
6. 提取简历中的关键经历、项目、技能、成果、疑点和岗位相关缺口。
7. 从面试记录中区分面试官问题、候选人回答、面试官即时判断、未回答或未追问的信息。
8. 读取 `references/evaluation-rubric.md`，按默认 100 分制评价。
9. 读取 `references/evaluation-template.md`，生成 Markdown 面试评价。
10. 如果用户未指定输出路径，将评价写入 `recruit-assist-output/interview-evaluation.md`。

## 生成要求

- 默认输出三个评价项：专业水平 50 分、项目经验 25 分、团队适配及综合素质 25 分。
- 每个评价项包含 200 字以内的文字评价和分项得分。
- 未收到特殊格式要求时，严格使用 `references/evaluation-template.md` 的三个编号评价项；只有存在证据不足、记录矛盾或不合规信息时，才添加可选补充。
- 评价结论必须基于 JD、简历和面试记录中的可见证据。
- 面试记录有错别字、简称、语序混乱时，可以纠正表述并标注推断边界；不得补写候选人没有表达过的内容。
- 如果面试记录与简历不一致，列为风险或待复核项，不直接假定任一方真实。
- 对没有被问到、没有回答或证据不足的信息，写“面试记录中未充分覆盖”。
- 不基于年龄、性别、婚育、民族、宗教、籍贯、外貌等敏感或无关属性评价、扣分或排序。
- 如果记录中包含敏感或不合规问题，只做合规风险提示，不纳入评分。
- 如果用户要求其他格式或维度，优先遵循用户指定格式，但仍保留证据边界和合规约束。
