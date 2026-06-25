# Repository Guidelines

## 项目结构

本仓库是 `recruit-assist` agent plugin，不是传统后端服务。

- `.codex-plugin/plugin.json`：Codex plugin 元数据。
- `.claude-plugin/plugin.json`：Claude plugin 元数据。
- `skills/resume-screening/SKILL.md`：简历筛选工作流。
- `skills/resume-screening/references/`：评分细则和报告模板。
- `skills/interview-prep/SKILL.md`：面试准备工作流。
- `skills/interview-prep/references/`：面试轮次策略和问题模板。
- `docs/superpowers/`：设计说明和实施计划。

生成物不要写入 plugin 源码目录；默认使用 `recruit-assist-output/`。

## 开发与验证命令

本仓库没有 build step、package manager script 或运行时服务。修改后做最小验证：

```sh
find . -maxdepth 3 -type f | sort
sed -n '1,160p' skills/resume-screening/SKILL.md
sed -n '1,160p' skills/interview-prep/SKILL.md
```

修改 plugin manifest 后校验 JSON：

```sh
python3 -m json.tool .codex-plugin/plugin.json
python3 -m json.tool .claude-plugin/plugin.json
```

## 编写规范

- 文档和 skill 内容使用 Markdown。
- 正文使用简体中文；命令、文件名、manifest key、API 名称保持英文。
- 目录和文件名使用 kebab-case，例如 `resume-screening`、`scoring-rubric.md`。
- 工作流用编号列表；约束和检查项用短 bullet。
- 不引入新的框架、依赖或目录结构，除非有明确收益。

## 测试要求

当前没有自动化测试。修改 `SKILL.md` 或 `references/` 后，至少做一次人工 dry run：

- 使用一个小型 JD。
- 使用一份代表性 resume 或文本夹具。
- 确认输出结构符合对应模板。
- 确认只基于 JD 和 resume 中的可见证据生成结论。
- 确认仍禁止基于年龄、性别、婚育、民族、宗教等无关或敏感属性评分、排序或提问。

## Commit 与 PR

当前 checkout 无可用 Git 历史，无法确认项目既有 commit 规范。默认使用：

```text
type(scope): description
```

示例：

```text
docs(skills): 调整简历筛选评分细则
```

PR 应包含：变更摘要、影响的 skill/reference 路径、验证方式、行为变化示例。涉及合规或评分逻辑时，必须说明风险控制。

## Agent 工作约束

修改前先读相关 `SKILL.md` 和 `references/`。优先做最小、兼容、可维护的改动。不要重命名现有 skill、文件或字段，除非任务明确要求。
