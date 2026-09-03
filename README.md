# Coding Workflow

一个面向复杂工程任务的完整协作 Skill：先由 Claude Code 使用 `/grill-with-docs` 完成本地分析与决策澄清，再由 Pi 使用 Gemini 3.8 Flash 补充网络检索；随后由 Codex CLI 通过 `/to-spec` 和 `/to-tickets` 生成规格与任务票据，最后按阻塞关系逐个执行 `/implement`。

## Install

Clone this repository into the Agent skills directory:

```powershell
git clone https://github.com/taozhuowei/coding-workflow.git ~/.agents/skills/coding-workflow
```

The Skill entrypoint is `SKILL.md`. It is intended for explicit invocation as `$coding-workflow`.

## Usage

Provide a feature, refactor, adjustment, or other complex coding task. The workflow stores durable discovery decisions, research, specifications, and tickets under `.scratch/<feature-slug>/` in the target project.
