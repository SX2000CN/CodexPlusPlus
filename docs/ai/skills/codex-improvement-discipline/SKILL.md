# Codex++ 改进工作纪律

本 skill 定义了对 Codex++ 项目进行功能改进、稳定性优化和缺陷修复时应遵循的工作纪律。

## 触发场景

当 AI 编码工具对 Codex++ 进行非简单修改（跨模块、涉及 watcher/launcher/renderer/helper/installer 等核心组件）时，应按本纪律执行。

## 核心原则

1. **不破坏现有功能**：修改前先读取相关测试和代码，保留现有函数名和测试契约。
2. **小步验证**：每完成一个模块的修改，立即运行该模块的测试；全部完成后运行全量测试。
3. **分层提交**：本地 main 可包含 AI 配置文件（docs/ai、.claude/skills 等）；向上游提 PR 时必须创建纯净分支，排除所有本地 AI 配置。

## 修改前

- 读取 `docs/ai/CURRENT.md` 了解当前工作状态。
- 读取被修改模块的现有代码和测试。
- 确认 Release 仓库和 `updater.py` 是否在本轮修改范围内（默认不动）。

## 修改中

- renderer-inject.js：保持 bridge-only 删除链路，不引入 HTTP fetch fallback；DOM 选择器集中维护。
- helper_server.py：默认关闭 HTTP mutation，保留 bridge 主路径。
- watcher.py：降低接管侵入性；log() 永不崩溃；具备自愈重启能力。
- launcher.py：继承代理环境或自动探测本地代理端口。

## 修改后

- 运行分组测试：`python -m pytest tests/test_<module>.py -q`
- 运行全量测试：`python -m pytest -q`
- 静态检查：`git diff --check`
- 更新 `docs/ai/CURRENT.md` 记录进展。
- 更新 `README.md` 中受影响的用户可见说明。

## 向上游提 PR

1. 基于上游 main 创建新分支。
2. 只 checkout 代码、测试和 README（排除 docs/ai、.claude、.agents、AGENTS.md 等）。
3. 使用代理推送和创建 PR（本机 GitHub 直连不可用）。
4. PR 标题简洁，正文说明改动目的和验证结果。

## 归档

- AI 不得自行归档 `docs/ai/CURRENT.md`；只有用户明确确认后才可归档。
- 归档到 `docs/ai/archive/` 并重置 CURRENT.md。
