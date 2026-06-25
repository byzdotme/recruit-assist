---
name: interview-prep
description: 基于 JD、指定候选人简历和面试轮次，为面试官准备考察策略、问题清单、追问方向、风险验证点和评分建议。Use when the user asks to prepare an interview, generate interview questions, plan a technical/business/HR interview, or create interviewer notes from a JD and one candidate resume.
---

# 面试准备

## 工作流

1. 确认用户提供了 JD 文件、指定候选人简历和面试轮次。
2. 读取 JD。Markdown 和 TXT 直接读取文本；其他格式请求用户转换。
3. 读取指定简历。PDF 使用当前 agent 环境可用的 PDF 文本抽取能力；DOCX 使用当前 agent 环境可用的文档读取能力。
4. 提取 JD 的岗位目标、核心职责、硬性要求、加分项和关键能力。
5. 提取候选人的关键经历、优势、疑点、缺口和需要验证的项目。
6. 读取 `references/interview-rounds.md`，按面试轮次选择考察重点。
7. 读取 `references/question-template.md`，生成 Markdown 面试准备文档。
8. 如果用户未指定面试时长，默认按 45 分钟设计。
9. 如果用户未指定输出路径，将文档写入 `recruit-assist-output/interview-prep.md`。

## 生成要求

- 每个问题必须标注考察目的。
- 每个核心问题必须给出优秀回答信号、风险信号和建议追问。
- 问题要围绕 JD 与简历交集，不生成与岗位无关的题目。
- 对简历中没有证据的信息，不假设为真实经历。
- 不围绕年龄、性别、婚育、民族、宗教等敏感或无关属性提问。
- 如果 JD 或简历信息不足，明确列出需要面试官补充确认的问题。
