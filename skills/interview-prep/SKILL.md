---
name: interview-prep
description: 基于 JD、指定候选人简历和面试轮次，为面试官准备考察策略、问题清单、追问方向、风险验证点和评分建议。Use when the user asks to prepare an interview, generate interview questions, plan a technical/business/HR interview, or create interviewer notes from JD and one candidate resume in flexible file or text formats.
---

# 面试准备

## 输入读取规则

- 支持文件路径、粘贴文本或用户在对话中明确标注的内容。
- Markdown、TXT、纯文本和粘贴内容直接读取；PDF、DOCX 和其他可读文本格式优先使用当前 agent 环境可用能力抽取文本。
- 不要求用户预先转换格式；仅在无法读取、抽取结果明显不可用或缺少必要内容时，记录来源和失败原因，并请求用户补充内容或转换格式。

## 输出路径规则

- 用户指定输出路径时，优先使用用户指定路径。
- 用户未指定输出路径且简历来源是文件路径时，将文档写入对应简历文件同目录。
- 默认文件名使用 `{候选人姓名}-interview-prep.md`；候选人姓名必须来自简历或用户明确提供的信息。
- 候选人姓名用于文件名时，应移除或替换路径分隔符、控制字符和其他不适合文件名的字符，保留可读性。
- 如果无法可靠提取候选人姓名，使用简历文件名主干；仍无法确定时使用 `unknown-candidate-interview-prep.md`。
- 如果简历不是文件路径或无法确定同目录，将文档写入 `recruit-assist-output/{候选人姓名或回退名}-interview-prep.md`。

## 工作流

1. 确认用户提供了 JD 来源、指定候选人简历和面试轮次。
2. 按“输入读取规则”读取 JD 和简历。
3. 如果 JD 或简历无法解析，记录来源、失败原因和“需要人工复核”。
4. 提取 JD 的岗位目标、核心职责、硬性要求、加分项和关键能力。
5. 提取候选人的关键经历、优势、疑点、缺口和需要验证的项目。
6. 读取 `references/interview-rounds.md`，按面试轮次选择考察重点。
7. 读取 `references/question-template.md`，生成 Markdown 面试准备文档。
8. 如果用户未指定面试时长，默认按 45 分钟设计。
9. 按“输出路径规则”写入文档。

## 生成要求

- 每个问题必须标注考察目的。
- 每个核心问题必须给出优秀回答信号、风险信号和建议追问。
- 问题要围绕 JD 与简历交集，不生成与岗位无关的题目。
- 对简历中没有证据的信息，不假设为真实经历。
- 不围绕年龄、性别、婚育、民族、宗教等敏感或无关属性提问。
- 如果 JD 或简历信息不足，明确列出需要面试官补充确认的问题。
