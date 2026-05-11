# 项目级 AI 配置中枢

本目录记录 Codex++ 项目的 AI 协作状态、项目级 skills、工具入口和维护约定。

## 目录

- [CURRENT.md](CURRENT.md)：当前未归档工作的 AI 协作交接状态。
- [archive/](archive/)：用户确认归档后的历史状态摘要。
- [skills-registry.md](skills-registry.md)：项目级 skills 清单。
- [skills/](skills/)：项目级 skill 事实源。

## 当前工作状态

如果存在 [CURRENT.md](CURRENT.md)，开始非简单任务前应先读取它，用于恢复当前工作现场。当目标、关键结论、已尝试或已排除方向、卡点、下一步发生明显变化时，应刷新该文件。

AI 不得自行归档当前状态；只有用户明确确认通过、没问题、任务结束或要求归档时，才可归档。若 AI 认为一轮工作已完成但用户尚未确认，应把状态保持为待用户审核。

## 约定

- [CURRENT.md](CURRENT.md) 只保存当前未归档工作的交接状态，不替代 README、CHANGELOG、issue 或 git log。
- [skills/](skills/) 下的 `docs/ai/skills/<skill-name>/` 是具体 skill 的事实源。
- `.claude/skills/<skill-name>/SKILL.md` 是 Claude Code 工具入口。
- `.agents/skills/<skill-name>/SKILL.md` 是 Codex 工具入口。
- `.codex/skills/<skill-name>/SKILL.md` 只在历史兼容需要时维护。
- 工具入口只做薄入口，不复制完整业务规则。
