# Codex Daily Report Skill

一个用于根据本地 Codex 会话记录生成当日工作日报的通用 skill。

它的目标不是从 Git 反推工作内容，而是从某一天、某个工作区里的真实对话记录中，还原当天做了什么，再整理成非技术同事也能看懂的中文日报。

## 适用场景

- 想按日期汇总某个工作区当天的 Codex 对话
- 想排除测试线程、闲聊线程和“写日报”本身的线程
- 想把当天工作归并成 3 到 6 条清晰的工作事项
- 想输出成中文日报，便于发给业务、产品或管理侧

## 数据来源

这个 skill 默认基于本机 Codex 的本地数据：

- `~/.codex/state_5.sqlite`
- `~/.codex/sessions/.../*.jsonl`

## 特点

- 不绑定特定项目
- 默认优先使用当前工作区
- 支持按工作区根目录连同子目录一起筛选线程
- 输出路径不写死，可由用户指定，也可沿用当前工作区已有日报目录约定
- 以人话表达为主，避免把日报写成技术审计报告

## 仓库结构

```text
.
├── agents/
│   └── openai.yaml
├── SKILL.md
└── references/
    └── commands.md
```

## 使用方式

把这个目录安装到你的全局 skills 目录，或复制到你自己的 agent/客户端技能目录中。

常见结构示例：

- Codex: `~/.codex/skills/codex-daily-report-skill`
- Anti Gravity: `~/.gemini/antigravity/skills/codex-daily-report-skill`

## 调用示例

在支持 skills 的客户端里，可以直接用类似下面的提示：

```text
Use $codex-daily-report to summarize today's Codex conversations in the current workspace into an 8h Chinese daily report.
```

## 输出约定

输出路径优先级：

1. 用户明确给了文件路径或目录，就直接使用。
2. 用户没给时，先沿用当前工作区里已存在的日报目录约定。
3. 如果没有明显约定，默认回退到当前工作区下的 `./daily/{date}.md`。

## 说明

- 这个 skill 只依赖本地会话记录，不包含任何项目私有逻辑。
- 如果你要把它分享给团队成员，建议团队统一约定日报目录命名和格式。

## License

MIT
