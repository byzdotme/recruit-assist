# Recruit Assist

Recruit Assist 是一个招聘辅助 agent plugin，用于帮助招聘者根据 JD 筛选简历、准备面试问题和整理面试策略。

## 能力

- 简历筛选：读取 Markdown/TXT JD 和 PDF/DOCX 简历，生成可解释评分排序报告。
- 面试准备：根据 JD、指定简历和面试轮次，生成考察策略、问题清单、追问方向和评分建议。

## 目录结构

```text
recruit-assist/
├── .agents/
│   └── plugins/
│       └── marketplace.json
├── .codex-plugin/
│   └── plugin.json
├── .claude-plugin/
│   ├── marketplace.json
│   └── plugin.json
├── skills/
│   ├── resume-screening/
│   └── interview-prep/
└── README.md
```

## Codex 安装

本仓库包含 Codex marketplace 文件：`.agents/plugins/marketplace.json`，marketplace 名为 `byzdotme`。

本地安装：

```bash
codex plugin marketplace add /path/to/recruit-assist
codex plugin add recruit-assist@byzdotme
```

安装后重启 Codex App 或新开 Codex CLI 会话，让 Codex 重新加载插件 skills。

## Claude Code 安装

本仓库包含 Claude Code catalog 文件：`.claude-plugin/marketplace.json`，marketplace 名为 `byzdotme`。

本地安装：

```bash
claude plugin marketplace add /path/to/recruit-assist
claude plugin install recruit-assist@byzdotme
```

安装后新开 Claude Code 会话，让 Claude Code 重新加载插件 skills。

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
