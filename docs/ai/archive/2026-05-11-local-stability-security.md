# 工作归档：本地稳定性与安全优化

时间：2026-05-11
状态：用户已确认归档

## 结果

已完成本地稳定性与安全优化，并按用户要求暂不修改 Release 仓库和 `codex_session_delete/updater.py`。

主要结果：

- 降低 `renderer-inject.js` 对 Codex 前端 DOM 的脆弱依赖：集中维护关键选择器，并为插件入口、归档行定位和归档标题定位增加受限 fallback。
- 渲染端删除、撤销和归档查询保持 CDP bridge-only，不新增 HTTP fallback。
- `helper_server.py` 默认拒绝 `/delete`、`/undo`、`/archived-thread` HTTP POST；测试或显式本地调试可用 `allow_http_mutation=True` 或 `X-Codex-Session-Delete-Token`。
- `watcher.py` 增加接管前等待、kill 前 CDP 二次确认、失败 backoff、成功 cooldown，并修复 watcher 非默认 debug port 启动 launcher 时端口不一致的问题。
- 真实启动测试发现 Codex 技能页访问 GitHub 失败；已在 launcher 中加入代理环境继承和常见本地代理端口自动探测。
- 已按用户要求安装并启动 Windows 自动接管 watcher，写入 HKCU Run 和 Startup 快捷方式。

## 关键决策

- Release 仓库和 `updater.py` 本轮不动，只做本地测试阶段优化。
- 删除类操作主路径继续走 CDP bridge，默认不开放本地 HTTP mutation，减少本机其他页面误触发删除操作的风险。
- watcher 本轮不扩大进程匹配范围，只降低触发频率和端口错配风险；更精确的进程路径匹配可留到后续独立优化。
- GitHub skills 加载失败按本机网络问题处理：优先继承已有代理环境变量，未设置时自动探测常见本地代理端口。

## 修改位置

- `codex_session_delete/inject/renderer-inject.js`：集中 DOM 选择器，增加插件入口和归档页受限 fallback，保持 bridge-only 删除链路。
- `codex_session_delete/helper_server.py`：默认关闭 HTTP mutation，增加显式开关和 token header 授权。
- `codex_session_delete/launcher.py`：增加代理环境继承、常见本地代理端口探测，并在 Windows packaged activation 前临时注入代理环境。
- `codex_session_delete/watcher.py`：降低自动接管误杀和频繁闪烁风险，并向 launcher 传递 watcher 使用的 debug port。
- `tests/test_renderer_script.py`：覆盖 renderer 选择器集中化、受限 fallback 和 bridge-only 契约。
- `tests/test_helper_server.py`：覆盖默认拒绝 HTTP mutation、显式允许和 token 授权。
- `tests/test_launcher_cli.py`：覆盖代理自动探测和显式代理优先。
- `tests/test_watcher.py`：覆盖 debug port 传递、CDP 二次确认和 backoff 下限。
- `README.md`：同步说明 bridge-only 删除路径、watcher 新行为和技能页 GitHub 访问失败的代理处理方式。
- `docs/ai/CURRENT.md`：记录本轮状态，归档后重置为暂无未归档工作。

## 验证

- `python -m pytest tests/test_renderer_script.py -q`：通过，17 passed。
- `python -m pytest tests/test_helper_server.py tests/test_launcher_cli.py -q`：通过。
- `python -m pytest tests/test_watcher.py -q`：通过，13 passed。
- `python -m pytest tests/test_launcher_cli.py -q`：代理修复后通过，33 passed。
- `python -m pytest -q`：最终全量通过，120 passed。
- `git diff --check`：通过。
- 真实启动验证：helper `/health` 返回 200，CDP `/json` 返回 200。
- HTTP mutation 默认拒绝验证：直接 POST `/delete` 返回 403。
- 技能仓库访问验证：清空代理环境后，launcher 自动探测到 `http://127.0.0.1:7897`，`git ls-remote https://github.com/openai/skills.git HEAD` 成功。
- 自动接管安装验证：`watch-install` 已写入 HKCU Run、Startup 快捷方式，并启动 watcher 进程。

## 残留问题

- 仍需用户在真实 Codex App UI 中手动确认：`Codex++` 菜单、配置面板、插件入口、技能页、会话删除、归档删除和撤销。
- 若后续仍发现 watcher 误杀或闪烁，可继续做更精确的 Codex 进程路径/命令行匹配。
- Release 更新源仍保持当前项目原状；如后续要改 Release 仓库，需要单独确认并处理发版链路。

## 关联记录

- commit：本归档随用户要求的 Git 提交一并发布。
- PR / issue：无。
