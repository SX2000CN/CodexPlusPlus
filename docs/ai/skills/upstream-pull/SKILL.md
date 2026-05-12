# 从上游拉取最新代码

本 skill 记录如何从上游原作者仓库（BigPizzaV3/CodexPlusPlus）拉取最新代码合并到本地 main 分支，并正确处理因本地 AI 配置文件导致的冲突。

## 触发场景

当上游仓库有新的提交（其他贡献者的 PR 被合并、原作者更新等），需要同步到本地 fork 时使用。

## 术语说明

- **上游（upstream）**：原作者仓库 `BigPizzaV3/CodexPlusPlus`，是 fork 的源头。
- **远程（origin）**：你自己的 fork `SX2000CN/CodexPlusPlus`。
- 说"从上游拉取"是标准 Git 用法，完全正确。

## 背景

本地 main 分支包含上游没有的私有文件：

- `docs/ai/` — AI 配置中枢
- `.agents/skills/` — Codex agent skill 入口
- `.claude/skills/` — Claude Code skill 入口（已 gitignore，不会冲突）

上游的更新可能修改与本地相同的文件（如 README.md、watcher.py 等），合并时会产生冲突。本地 AI 配置文件因为上游不存在，通常不会冲突，但需要确认合并后仍然保留。

## 操作步骤

### 1. 确保本地工作区干净

```bash
git status
```

如果有未提交的改动，先提交或暂存：

```bash
git stash  # 暂存未提交改动
```

### 2. 从上游拉取并合并

```bash
# 使用代理（本机 GitHub 直连不可用）
HTTPS_PROXY=http://127.0.0.1:7897 HTTP_PROXY=http://127.0.0.1:7897 \
  git pull --no-rebase https://github.com/BigPizzaV3/CodexPlusPlus.git main
```

说明：
- 使用 `--no-rebase`（即 merge）而非 rebase，因为本地有上游不存在的额外提交（AI 配置），rebase 会把这些提交反复重放，容易出问题。
- 直接用 URL 拉取，不需要配置 upstream remote（当然配了也行）。

### 3. 处理冲突

#### 情况 A：无冲突

直接完成，跳到步骤 4。

#### 情况 B：有冲突

常见冲突场景和处理方式：

| 冲突文件 | 处理方式 |
|---|---|
| `README.md` | 保留上游内容为主，把本地额外说明（如 watcher 自愈、代理探测）合并进去 |
| `codex_session_delete/*.py` | 逐个检查，通常保留双方改动（上游新功能 + 本地优化） |
| `tests/*.py` | 同上，保留双方测试 |
| `docs/ai/*` | 上游不会有这些文件，不应冲突；如果冲突说明操作有误 |

解决冲突后：

```bash
git add <冲突文件>
git commit  # Git 会自动生成 merge commit 信息
```

### 4. 验证合并结果

```bash
# 确认本地 AI 配置文件仍然存在
ls docs/ai/CURRENT.md
ls .agents/skills/

# 运行测试确认没有破坏
python -m pytest -q

# 检查 diff
git diff --check
```

### 5. 推送到 origin

```bash
HTTPS_PROXY=http://127.0.0.1:7897 HTTP_PROXY=http://127.0.0.1:7897 \
  git push origin main
```

### 6. 恢复暂存（如果步骤 1 用了 stash）

```bash
git stash pop
```

## 可选：配置 upstream remote

如果经常拉取上游，可以配置一个 remote 避免每次输入完整 URL：

```bash
git remote add upstream https://github.com/BigPizzaV3/CodexPlusPlus.git
```

之后拉取可以简写为：

```bash
HTTPS_PROXY=http://127.0.0.1:7897 HTTP_PROXY=http://127.0.0.1:7897 \
  git fetch upstream main

git merge upstream/main
```

## 注意事项

- 本机 GitHub 直连超时，所有远程操作需要加 `HTTPS_PROXY=http://127.0.0.1:7897`。
- 合并后务必确认 `docs/ai/`、`.agents/skills/` 等本地 AI 配置文件没有被删除。
- 如果上游删除了某个文件而本地修改了它，Git 会报冲突；通常选择保留本地版本或按实际情况决定。
- 合并提交信息建议用中文描述，如：`合并上游：<简述上游新增内容>`。

## 完整工作闭环

```
上游有更新 → 从上游拉取（本 skill）→ 本地开发/测试 → 提交到 main → 向上游提 PR（另一个 skill）
     ↑                                                                              |
     └──────────────────── 上游合并 PR ←────────────────────────────────────────────┘
```
