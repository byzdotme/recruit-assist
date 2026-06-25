# Recruit Assist

Recruit Assist 是一个招聘辅助 agent plugin，用于帮助招聘者根据 JD 筛选简历、准备面试问题、整理面试策略并生成面试评价。

## 能力

- 简历筛选：读取 Markdown/TXT JD 和 PDF/DOCX 简历，生成可解释评分排序报告。
- 面试准备：根据 JD、指定简历和面试轮次，生成考察策略、问题清单、追问方向和评分建议。
- 面试评价：根据 JD、指定简历和面试官记录的面试过程，生成分项评价、打分和后续建议。

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
│   ├── interview-prep/
│   └── interview-evaluation/
└── README.md
```

## Codex 安装

本仓库包含 Codex marketplace 文件：`.agents/plugins/marketplace.json`，marketplace 名为 `byzdotme`。

从 GitHub 仓库安装 marketplace 并安装插件：

```bash
codex plugin marketplace add byzdotme/recruit-assist --ref main
codex plugin add recruit-assist@byzdotme
```

安装后重启 Codex App 或新开 Codex CLI 会话，让 Codex 重新加载插件 skills。

## Claude Code 安装

本仓库包含 Claude Code catalog 文件：`.claude-plugin/marketplace.json`，marketplace 名为 `byzdotme`。

从 GitHub 仓库安装 marketplace 并安装插件：

```bash
claude plugin marketplace add byzdotme/recruit-assist@main
claude plugin install recruit-assist@byzdotme
```

安装后新开 Claude Code 会话，让 Claude Code 重新加载插件 skills。

## 环境依赖

- 需要安装并登录 Codex App、Codex CLI 或 Claude Code CLI 中的至少一种。
- 需要能访问 `git@github.com:byzdotme/recruit-assist.git` 或 `https://github.com/byzdotme/recruit-assist`。如果仓库是私有仓库，同事需要对应 GitHub 权限和可用的 Git/SSH 或 HTTPS 凭据。
- 插件本身不依赖 Python、Node.js、包管理器、数据库或后台服务。
- 插件不会调用外部 API，也不会连接 ATS、邮箱、数据库或第三方招聘系统。
- JD 建议使用 Markdown 或 TXT。面试过程记录建议使用 TXT 或 Markdown。PDF/DOCX 简历的读取依赖当前 agent 环境可用的文件解析能力；如果遇到扫描版 PDF、图片型 PDF 或解析乱码，需要先人工转换为可复制文本、Markdown、TXT 或可解析 PDF。
- 插件没有 macOS 专属依赖；Windows、Linux、macOS 都可以使用，前提是对应 coding agent 支持插件安装和文件读取。

## 使用示例

```text
根据 jobs/backend-engineer.md 筛选 resumes/ 里的简历，输出 Markdown 排名报告。
```

```text
根据 jobs/backend-engineer.md 和 resumes/zhang-san.pdf，为技术面准备 45 分钟的问题清单。
```

```text
根据 jobs/backend-engineer.md、resumes/zhang-san.pdf 和 notes/zhang-san-interview.txt 生成面试评价。
```

## 隐私与合规

- 不要把密钥或 token 放入 JD、简历或输出报告。
- 不要基于年龄、性别、婚育、民族、宗教等敏感或无关属性筛选或评价候选人。
- 如果 JD 包含疑似不合规筛选条件，应在报告中提示人工复核。
- 插件首期不连接外部招聘系统、数据库或第三方 API。
