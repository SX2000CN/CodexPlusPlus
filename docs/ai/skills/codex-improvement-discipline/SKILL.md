# 向上游提 PR 流程

本 skill 记录从本地 fork（含私有 AI 配置文件）向上游原作者仓库提交 Pull Request 时，如何创建纯净分支并排除本地 AI 配置。

## 触发场景

当用户要求向上游仓库（BigPizzaV3/CodexPlusPlus）提交 PR 时使用。

## 背景

本地 fork 的 main 分支包含私有 AI 配置文件（`docs/ai/`、`.agents/skills/`、`.claude/skills/` 等），这些文件不应出现在上游仓库中。每次向上游提 PR 都需要创建一个不含这些文件的纯净分支。

## 需要排除的文件和目录

- `docs/ai/` — 项目级 AI 配置中枢（CURRENT.md、archive、skills-registry、skills 事实源）
- `.claude/` — Claude Code 本地 skill 入口（已被 .gitignore 排除）
- `.agents/skills/` — Codex agent skill 入口
- `AGENTS.md` — 如果包含本地私有内容

## 操作步骤

### 1. 确保本地改动已提交到 main

```bash
git add <相关文件>
git commit -m "提交信息"
git push origin main
```

### 2. 从上游 main 创建纯净 PR 分支

```bash
# 使用代理拉取上游（本机 GitHub 直连不可用）
HTTPS_PROXY=http://127.0.0.1:7897 HTTP_PROXY=http://127.0.0.1:7897 \
  git fetch https://github.com/BigPizzaV3/CodexPlusPlus.git main

# 基于上游 main 创建新分支
git switch -c pr/<分支名> FETCH_HEAD
```

### 3. 只 checkout 需要提交的文件

从本地 main 中只取代码、测试和 README，不取 AI 配置：

```bash
git checkout main -- \
  README.md \
  codex_session_delete/ \
  tests/ \
  pyproject.toml \
  setup.bat
```

注意：如果 `git status` 显示了 `docs/ai/`、`.agents/skills/`、`AGENTS.md` 等文件，说明取多了，需要 `git restore --staged` 去掉。

### 4. 检查暂存区确认无 AI 配置

```bash
git diff --cached --name-only
```

确认列表中不包含：
- `docs/ai/*`
- `.agents/skills/*`
- `.claude/*`
- `AGENTS.md`（如果是私有内容）

### 5. 提交并推送

```bash
git commit -m "提交信息"

HTTPS_PROXY=http://127.0.0.1:7897 HTTP_PROXY=http://127.0.0.1:7897 \
  git push -u origin pr/<分支名>
```

### 6. 创建 PR

本机无 `gh` CLI，使用浏览器打开：

```
https://github.com/BigPizzaV3/CodexPlusPlus/compare/main...SX2000CN:CodexPlusPlus:pr/<分支名>?expand=1
```

### 7. 清理旧的 PR 临时分支

最多保留最近 3 个 PR 临时分支，超出的从本地和远程一并删除：

```bash
# 查看现有 PR 分支（本地 + 远程）
git branch --list 'pr/*'
git branch -r --list 'origin/pr/*'

# 删除本地旧分支
git branch -D pr/<旧分支名>

# 删除远程旧分支（需要代理）
HTTPS_PROXY=http://127.0.0.1:7897 HTTP_PROXY=http://127.0.0.1:7897 \
  git push origin --delete pr/<旧分支名>
```

规则：每次创建新 PR 分支后，检查是否超过 3 个，如果超过则删除最早的。

### 8. 切回 main

```bash
git switch main
```

## 注意事项

- 本机 GitHub 直连超时，所有 git 远程操作需要加 `HTTPS_PROXY=http://127.0.0.1:7897`。
- 上游仓库是 `BigPizzaV3/CodexPlusPlus`，fork 是 `SX2000CN/CodexPlusPlus`。
- PR 分支命名建议：`pr/<功能简述>`，如 `pr/watcher-reliability`。
- PR 临时分支最多保留 3 个（本地 + 远程），超出的必须清理。
