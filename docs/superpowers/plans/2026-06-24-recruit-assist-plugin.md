# Recruit Assist Plugin Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 创建一个可在 Codex 和 Claude Code 中安装使用的招聘辅助 agent plugin，提供简历筛选和面试准备两个共享 skill。

**Architecture:** 采用双 manifest、共享 skills 的结构：`.codex-plugin/plugin.json` 面向 Codex，`.claude-plugin/plugin.json` 面向 Claude Code，两个平台共用 `skills/` 下的工作流说明和 reference 模板。插件不内置外部服务、OCR、数据库或网络依赖，首期以 agent 当前环境可用的文件读取能力完成 JD、PDF 和 DOCX 处理。

**Tech Stack:** Codex plugin manifest、Claude Code plugin manifest、Agent Skills、Markdown reference templates、shell validation commands。

---

## 文件结构

创建以下文件：

- `.codex-plugin/plugin.json`：Codex 插件 manifest，包含插件元数据、`skills` 路径和 Codex UI 信息。
- `.claude-plugin/plugin.json`：Claude Code 插件 manifest，包含插件元数据和仓库分发信息。
- `skills/resume-screening/SKILL.md`：简历筛选 skill 入口，定义触发条件、文件读取流程、评分约束和输出要求。
- `skills/resume-screening/references/scoring-rubric.md`：100 分可解释评分细则。
- `skills/resume-screening/references/report-template.md`：简历筛选报告模板。
- `skills/interview-prep/SKILL.md`：面试准备 skill 入口，定义触发条件、轮次策略选择和输出要求。
- `skills/interview-prep/references/interview-rounds.md`：业务面、技术面、HR 面、终面等轮次策略。
- `skills/interview-prep/references/question-template.md`：面试准备 Markdown 模板。
- `README.md`：中文说明，覆盖插件目标、目录结构、Codex 安装、Claude Code 安装、本地使用示例和隐私注意事项。

---

### Task 1: 创建插件 manifest

**Files:**
- Create: `.codex-plugin/plugin.json`
- Create: `.claude-plugin/plugin.json`

- [ ] **Step 1: 创建 manifest 目录**

Run:

```bash
mkdir -p .codex-plugin .claude-plugin
```

Expected: 命令成功，无输出。

- [ ] **Step 2: 写入 Codex manifest**

Create `.codex-plugin/plugin.json` with:

```json
{
  "name": "recruit-assist",
  "version": "0.1.0",
  "description": "招聘辅助插件，用于根据 JD 筛选简历并准备面试问题。",
  "author": {
    "name": "Recruit Assist Team"
  },
  "license": "MIT",
  "keywords": [
    "recruiting",
    "resume-screening",
    "interview-prep",
    "hiring"
  ],
  "skills": "./skills/",
  "interface": {
    "displayName": "Recruit Assist",
    "shortDescription": "协助招聘者完成简历筛选和面试准备。",
    "longDescription": "Recruit Assist 提供简历筛选和面试准备两个 agent skills。它可以读取项目中的 JD、PDF 简历和 DOCX 简历，生成可解释评分报告、候选人排序、面试策略和问题清单。",
    "developerName": "Recruit Assist Team",
    "category": "Productivity",
    "capabilities": [
      "Write"
    ],
    "defaultPrompt": [
      "根据这个 JD 批量筛选 resumes 目录里的简历",
      "为这位候选人的技术面准备面试问题",
      "根据筛选报告生成推荐面试名单"
    ],
    "brandColor": "#2563EB"
  }
}
```

- [ ] **Step 3: 写入 Claude Code manifest**

Create `.claude-plugin/plugin.json` with:

```json
{
  "name": "recruit-assist",
  "version": "0.1.0",
  "description": "招聘辅助插件，用于根据 JD 筛选简历并准备面试问题。",
  "author": {
    "name": "Recruit Assist Team"
  },
  "license": "MIT",
  "keywords": [
    "recruiting",
    "resume-screening",
    "interview-prep",
    "hiring"
  ]
}
```

- [ ] **Step 4: 校验 manifest JSON 语法**

Run:

```bash
python3 -m json.tool .codex-plugin/plugin.json >/tmp/recruit-assist-codex-plugin.json
python3 -m json.tool .claude-plugin/plugin.json >/tmp/recruit-assist-claude-plugin.json
```

Expected: 两条命令均成功，无错误输出。

---

### Task 2: 创建简历筛选 skill

**Files:**
- Create: `skills/resume-screening/SKILL.md`
- Create: `skills/resume-screening/references/scoring-rubric.md`
- Create: `skills/resume-screening/references/report-template.md`

- [ ] **Step 1: 创建 skill 目录**

Run:

```bash
mkdir -p skills/resume-screening/references
```

Expected: 命令成功，无输出。

- [ ] **Step 2: 写入 skill 入口**

Create `skills/resume-screening/SKILL.md` with:

```markdown
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
5. 从 JD 提取硬性要求、岗位职责、加分项、经验年限、技术栈、业务领域、地点、语言和其他约束。
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
```

- [ ] **Step 3: 写入评分细则**

Create `skills/resume-screening/references/scoring-rubric.md` with:

```markdown
# 简历筛选评分细则

总分 100 分。评分时以 JD 为唯一基准，不因为 JD 未要求的经历额外加分。

## 评分维度

| 维度 | 分值 | 评分说明 |
| --- | ---: | --- |
| 硬性要求匹配 | 30 | 必须技能、经验年限、地点、语言、资质、行业限制等。完全满足给高分；关键硬性要求缺失应明显扣分。 |
| 相关项目和业务经验 | 25 | 项目职责、业务领域、场景复杂度、成果质量和 JD 的贴合度。 |
| 技术能力或岗位能力深度 | 20 | 技术栈、工具链、方法论、问题解决深度、独立负责程度。 |
| 能力递进与项目连续性 | 10 | 只评价简历中与岗位能力递进、职责范围扩大、项目连续性和协作线索相关的证据；不得基于年龄、毕业年份、职业空窗原因、家庭状态等作推断。 |
| 加分项 | 10 | 仅使用 JD 明确提到的 preferred qualifications 或明显相关亮点。 |
| 风险和缺口控制 | 5 | 关键缺失、频繁跳槽、信息矛盾、描述泛化、解析质量问题。 |

## 推荐动作

| 总分区间 | 建议动作 |
| --- | --- |
| 85-100 | 推荐面试 |
| 70-84 | 备选，建议人工复核后决定 |
| 55-69 | 暂缓，除非候选池不足或有特殊亮点 |
| 0-54 | 不推荐 |

## 置信度

- 高：JD 和简历都解析完整，关键经历有明确证据。
- 中：信息基本完整，但部分关键要求需要面试确认。
- 低：简历解析不完整、证据不足或存在明显信息缺口。
```

- [ ] **Step 4: 写入报告模板**

Create `skills/resume-screening/references/report-template.md` with:

```markdown
# 简历筛选报告

## 岗位摘要

- 岗位名称：
- 核心职责：
- 硬性要求：
- 加分项：
- 需要人工复核的 JD 条件：

## 排名总览

| 排名 | 候选人 | 文件 | 总分 | 置信度 | 建议动作 | 核心理由 |
| ---: | --- | --- | ---: | --- | --- | --- |

## 候选人明细

### 1. 候选人姓名

- 文件：
- 解析状态：
- 总分：
- 置信度：
- 建议动作：

| 维度 | 分数 | 证据或扣分原因 |
| --- | ---: | --- |
| 硬性要求匹配 | 0/30 |  |
| 相关项目和业务经验 | 0/25 |  |
| 技术能力或岗位能力深度 | 0/20 |  |
| 能力递进与项目连续性 | 0/10 |  |
| 加分项 | 0/10 |  |
| 风险和缺口控制 | 0/5 |  |

**证据摘要：**

- 

**主要缺口和风险：**

- 

**后续面试验证问题：**

- 

## 风险与复核项

- 

## 建议面试名单

| 优先级 | 候选人 | 建议轮次 | 需要重点验证 |
| ---: | --- | --- | --- |
```

- [ ] **Step 5: 校验 skill 文件存在**

Run:

```bash
test -f skills/resume-screening/SKILL.md
test -f skills/resume-screening/references/scoring-rubric.md
test -f skills/resume-screening/references/report-template.md
```

Expected: 三条命令均成功，无输出。

---

### Task 3: 创建面试准备 skill

**Files:**
- Create: `skills/interview-prep/SKILL.md`
- Create: `skills/interview-prep/references/interview-rounds.md`
- Create: `skills/interview-prep/references/question-template.md`

- [ ] **Step 1: 创建 skill 目录**

Run:

```bash
mkdir -p skills/interview-prep/references
```

Expected: 命令成功，无输出。

- [ ] **Step 2: 写入 skill 入口**

Create `skills/interview-prep/SKILL.md` with:

```markdown
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
```

- [ ] **Step 3: 写入轮次策略**

Create `skills/interview-prep/references/interview-rounds.md` with:

```markdown
# 面试轮次策略

## 业务面

重点验证业务理解、过往场景、问题拆解、协作方式和结果质量。

建议问题类型：

- 让候选人复盘一个与 JD 场景接近的项目。
- 追问目标、约束、决策过程、跨团队协作和最终结果。
- 验证候选人是否理解岗位所在业务的关键指标和用户价值。

## 技术面

重点验证技术深度、架构判断、工程实践、问题定位和项目真实性。

建议问题类型：

- 围绕简历中最相关项目做深挖。
- 要求候选人解释设计取舍、性能瓶颈、可靠性问题和故障处理。
- 针对 JD 必须技能提出渐进式问题，从基础理解到真实场景。

## HR 面

重点验证动机、稳定性、薪资期望、文化匹配、沟通风格和入职风险。

建议问题类型：

- 离职动机和下一份工作的选择标准。
- 对岗位职责、团队协作和工作节奏的期待。
- 薪资期望、到岗时间、offer 决策因素。

## 终面

重点验证岗位匹配度、长期潜力、关键风险和录用决策所需信息。

建议问题类型：

- 候选人对岗位挑战的理解。
- 过往关键选择和高压场景中的判断。
- 对未来 6-12 个月产出的预期和自我规划。
```

- [ ] **Step 4: 写入问题模板**

Create `skills/interview-prep/references/question-template.md` with:

```markdown
# 面试准备

## 候选人与岗位匹配摘要

- 候选人：
- 简历文件：
- 岗位：
- 匹配亮点：
- 主要缺口：
- 需要重点验证的风险：

## 本轮面试目标

- 面试轮次：
- 建议时长：
- 本轮必须确认的问题：

## 建议时间分配

| 环节 | 时长 | 目的 |
| --- | ---: | --- |
| 开场与背景确认 | 5 分钟 | 建立上下文，确认候选人当前状态 |
| 核心经历深挖 | 15 分钟 | 验证简历真实性和岗位相关能力 |
| 场景题或案例题 | 15 分钟 | 验证问题拆解和实际能力 |
| 候选人提问与收尾 | 10 分钟 | 观察动机，补充关键信息 |

## 核心问题与追问

### 问题 1：

- 考察目的：
- 关联 JD 要求：
- 关联简历证据：
- 优秀回答信号：
- 风险信号：
- 建议追问：

## 重点风险验证

| 风险 | 验证问题 | 通过信号 | 风险信号 |
| --- | --- | --- | --- |

## 评分建议

| 维度 | 权重 | 观察点 | 评分 |
| --- | ---: | --- | --- |
| 岗位核心能力 | 40% |  |  |
| 相关经验真实性 | 25% |  |  |
| 问题拆解与表达 | 20% |  |  |
| 动机与协作匹配 | 15% |  |  |

## 面试官记录区

- 关键正向信号：
- 关键风险：
- 需要下一轮继续验证：
- 初步结论：
```

- [ ] **Step 5: 校验 skill 文件存在**

Run:

```bash
test -f skills/interview-prep/SKILL.md
test -f skills/interview-prep/references/interview-rounds.md
test -f skills/interview-prep/references/question-template.md
```

Expected: 三条命令均成功，无输出。

---

### Task 4: 创建 README

**Files:**
- Create: `README.md`

- [ ] **Step 1: 写入 README**

Create `README.md` with:

```markdown
# Recruit Assist

Recruit Assist 是一个招聘辅助 agent plugin，用于帮助招聘者根据 JD 筛选简历、准备面试问题和整理面试策略。

## 能力

- 简历筛选：读取 Markdown/TXT JD 和 PDF/DOCX 简历，生成可解释评分排序报告。
- 面试准备：根据 JD、指定简历和面试轮次，生成考察策略、问题清单、追问方向和评分建议。

## 目录结构

```text
recruit-assist/
├── .codex-plugin/
│   └── plugin.json
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   ├── resume-screening/
│   └── interview-prep/
└── README.md
```

## Codex 安装

从本地目录安装时，将本仓库作为本地插件源加入 Codex 插件市场或复制到团队约定的插件市场目录。安装后开启新线程，让 Codex 重新加载插件 skills。

## Claude Code 安装

Claude Code 支持从插件 marketplace 或 Git 仓库安装插件。团队发布公司仓库后，可按 Claude Code 当前插件安装命令添加公司插件源，再安装 `recruit-assist`。

## 使用示例

```text
根据 jobs/backend-engineer.md 筛选 resumes/ 里的简历，输出 Markdown 排名报告。
```

```text
根据 jobs/backend-engineer.md 和 resumes/zhang-san.pdf，为技术面准备 45 分钟的问题清单。
```

## 隐私与合规

- 不要把密钥或 token 放入 JD、简历或输出报告。
- 不要基于年龄、性别、婚育、民族、宗教等敏感或无关属性筛选候选人。
- 如果 JD 包含疑似不合规筛选条件，应在报告中提示人工复核。
- 插件首期不连接外部招聘系统、数据库或第三方 API。
```

- [ ] **Step 2: 校验 README 存在**

Run:

```bash
test -f README.md
```

Expected: 命令成功，无输出。

---

### Task 5: 运行校验并修正问题

**Files:**
- Verify: `.codex-plugin/plugin.json`
- Verify: `.claude-plugin/plugin.json`
- Verify: `skills/resume-screening/SKILL.md`
- Verify: `skills/interview-prep/SKILL.md`
- Verify: `README.md`

- [ ] **Step 1: 校验 JSON**

Run:

```bash
python3 -m json.tool .codex-plugin/plugin.json >/tmp/recruit-assist-codex-plugin.json
python3 -m json.tool .claude-plugin/plugin.json >/tmp/recruit-assist-claude-plugin.json
```

Expected: 两条命令均成功，无错误输出。

- [ ] **Step 2: 校验必需文件**

Run:

```bash
test -f .codex-plugin/plugin.json
test -f .claude-plugin/plugin.json
test -f skills/resume-screening/SKILL.md
test -f skills/resume-screening/references/scoring-rubric.md
test -f skills/resume-screening/references/report-template.md
test -f skills/interview-prep/SKILL.md
test -f skills/interview-prep/references/interview-rounds.md
test -f skills/interview-prep/references/question-template.md
test -f README.md
```

Expected: 所有命令均成功，无输出。

- [ ] **Step 3: 检查占位符和敏感输出风险**

Run:

```bash
rg -n "T[O]DO|T[B]D|PLACE[H]OLDER|\\[T[O]DO|token|secret|a[p]i key|a[p]ikey|pass[word]" .codex-plugin .claude-plugin skills README.md
```

Expected: 只允许 README 的隐私提示中出现 `token`；不应出现占位标记或凭据类敏感词。

- [ ] **Step 4: 校验 skill frontmatter**

Run:

```bash
sed -n '1,8p' skills/resume-screening/SKILL.md
sed -n '1,8p' skills/interview-prep/SKILL.md
```

Expected: 两个文件都以 `---` 开始，并包含 `name` 和 `description` 字段。

- [ ] **Step 5: 如果当前目录已初始化为 git 仓库，则提交**

Run:

```bash
git rev-parse --is-inside-work-tree
```

Expected when repository exists: 输出 `true`。

If it outputs `true`, run:

```bash
git add .codex-plugin .claude-plugin skills README.md docs/superpowers
git commit -m "feat(plugin): 初始化招聘辅助插件"
```

Expected: commit 成功。

If it fails with `fatal: not a git repository`, do not initialize git automatically. Record that the workspace is not a git repository and leave commit for the company repository setup step.

---

## 自检

- 规格中的双 manifest 方案由 Task 1 覆盖。
- `resume-screening` skill 和评分报告由 Task 2 覆盖。
- `interview-prep` skill 和轮次策略由 Task 3 覆盖。
- 团队共享说明由 Task 4 覆盖。
- JSON、文件存在性、frontmatter、占位符和 git 状态由 Task 5 覆盖。
- 首期非目标保持不实现：无外部 API、无 OCR、无 ATS、无 Web UI、无数据库。
