# 项目级 Skills 清单

最后更新时间：2026-05-11

当前没有已创建的项目级 skill。

| Skill | 状态 | 事实源 | Claude Code 入口 | Codex 入口 | 备注 |
|---|---|---|---|---|---|

## 状态说明

- `planned`：计划中，尚未实现。
- `active`：已启用。
- `partial`：部分可用。
- `deprecated`：已废弃。
- `compat`：仅为历史兼容保留。

## 维护约定

- 新建项目级 skill 时，优先把事实源放在 `docs/ai/skills/<skill-name>/`。
- Claude Code 入口放在 `.claude/skills/<skill-name>/SKILL.md`，只保留薄入口。
- Codex 入口放在 `.agents/skills/<skill-name>/SKILL.md`，只保留薄入口。
- `.codex/skills/<skill-name>/SKILL.md` 仅用于历史兼容。
