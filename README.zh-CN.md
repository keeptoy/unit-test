# pwf-codex-cloud-hooks：将 planning-with-files Hook 适配到 Codex Cloud

**文档：** [English（规范版本）](README.md) · 简体中文

这是一个开源的 **Codex Cloud Hook 适配器与 Managed Runtime 原型**。它把
[`OthmanAdi/planning-with-files`](https://github.com/OthmanAdi/planning-with-files)
（PWF）面向本地 Codex 的 Skill/Hook 行为，以固定来源、校验哈希和受管安装的方式部署
到远程 Codex Cloud 沙盒。

项目覆盖的关键概念包括：Codex Cloud Hooks、Managed Hooks、Codex Skill 云端适配、
Codex Plugin runtime、`/etc/codex/requirements.toml`、`managed_dir`、远程 Agent
Hook、会话恢复、上下文恢复和持久化文件规划。

> **版本状态：** `v0.2.2` 是已经发布并通过 Cloud 黑盒验证的回滚基线；`v0.3.0`
> 是尚未正式发布的 Managed Runtime Modernization 开发版本。当前仅支持
> `planning-with-files`，不能宣称能够自动转换任意 Skill。

## 为什么需要这个项目？

在个人电脑上安装 Skill，只会把说明和脚本保存在本机。电脑关机后，本地 Codex、
Plugin Hook 和 Skill 脚本都不会继续运行；而远程 Codex Cloud 任务虽然可以脱离本机
继续执行，却不会自动获得本机已经安装的 Skill、Hook、依赖和信任状态。

本地 PWF Hook 还会使用一些在 Cloud 中不一定成立的假设，例如：

- 项目 `.codex/hooks.json` 或用户 `~/.codex/hooks.json`；
- 相对于项目或 `$HOME/.codex` 的命令路径；
- `~/.codex/sessions` session store；
- 通过安装路径中的 `/.codex/` 推断当前 Runtime；
- 可以交互确认并长期保存的普通 Plugin Hook 信任状态。

当前已验证的 Codex Cloud 模型不同：

- 系统 Hook 策略由 `/etc/codex/requirements.toml` 提供；
- Managed Hook 命令必须使用位于 `managed_dir` 下的绝对路径；
- 当前观察到的 Codex 根目录和 session store 位于 `/opt/codex`；
- PWF Skill 由 skills CLI 安装到 `/root/.agents/skills/planning-with-files`；
- 冷启动沙盒需要通过初始化脚本重新建立运行环境；
- 初始化 Shell 的临时环境不能被当成后续 Hook 进程的永久契约。

因此本仓库实现的是下面这条桥接链路：

```text
本地 planning-with-files Skill + Codex Hooks
                       |
                       | 固定、导入、适配、安装、校验
                       v
Codex Cloud system-managed Hooks + owned runtime bundle
```

## 当前能做什么？

当前发布基线只启用两个只读 Managed Hooks：

| Event | 当前行为 |
|---|---|
| `SessionStart` | 输出 canary；在 resume 等场景运行 catch-up；注入活动计划和最近进度 |
| `UserPromptSubmit` | 输出 canary；在每次用户提示时注入活动计划和最近进度 |

当前 Cloud 兼容层还处理：

1. 显式设置 `PWF_RUNTIME=codex`，避免 `.agents/skills` 路径造成 Runtime 误判；
2. 优先使用 `CODEX_SESSIONS_DIR` 或 `$CODEX_HOME/sessions`；
3. 同时识别 `.planning/<slug>/task_plan.md` 和旧版根目录计划；
4. 对长 Cloud wrapper 有界保留头尾，避免真实用户指令在尾部被截掉；
5. 以绝对路径注册 Managed Hook，并维护安装清单、备份、doctor、repair 和卸载；
6. 固定上游版本与文件哈希，拒绝未知 runtime 漂移和 symlink 逃逸。

详细、规范的当前行为以英文 [`README.md`](README.md) 为准。

## 它不是什么？

本项目目前不是：

- 第二套 planning 方法；
- PWF 的永久 fork；
- 任意 Skill 的自动 Cloud 转换器；
- 自动扫描并以系统权限执行所有 Skill Hook 的工具；
- OpenAI 官方项目或企业 Managed Hooks 产品；
- 对未来所有 Codex Cloud 镜像路径和权限的承诺。

“Managed”表示 Hook 来自 Codex 的系统或管理配置层，并且 runtime 具有明确所有权；
它不表示本项目声称需要 Enterprise 订阅。当前个人 Cloud 沙盒路径经过真实验证，但仍然
是带日期的平台观察，而不是永久产品契约。

## 谁应该关注？

如果你遇到下面任一问题，这个仓库可能有参考价值：

- 本地 Codex Skill 或 Plugin Hook 在 Codex Cloud 中不执行；
- Skill 已安装，但 lifecycle Hook 没有注册；
- 冷启动 Cloud 沙盒无法稳定保存普通 Plugin Hook 的交互信任；
- 需要把 `.codex/hooks.json` 或 `${PLUGIN_ROOT}` 命令适配成 Cloud 绝对路径；
- 需要远程 `SessionStart`、`UserPromptSubmit`、resume、memory、audit、notification
  或 safety Hook；
- 需要 `requirements.toml` 合并、固定 runtime、hash inventory、doctor、repair、
  rollback 和卸载示例；
- 希望为另一个 Skill/Plugin 设计经过审核的 Cloud Managed Hook adapter。

## 术语

| 术语 | 本仓库中的含义 |
|---|---|
| Skill | Codex 发现的说明和可选脚本；安装 Skill 不等于在 Cloud 中激活其 Hook |
| Plugin Hook | Codex Plugin 携带的 Hook，仍受普通发现、启用和信任机制影响 |
| Managed Hook | 由系统或管理配置层提供的 Hook；本项目通过 `requirements.toml` 安装 |
| Host adapter | 校验 Codex Hook 输入、监督业务 runtime、输出有界 Codex JSON 的薄边界 |
| Owned runtime | 被 allowlist 和哈希验证、安装在 `managed_dir` 下的精确可执行文件 |
| Cloud adapter | 将经过审核的本地 Hook 行为变成可复现 Cloud runtime 的部署与 Host 兼容层 |

## 未来是否会支持更多插件？

可能，但必须先证明当前 PWF 垂直实现可靠。合理路线是：

```text
现在：PWF 本地 Hook → PWF Cloud Managed Hook
  ↓
v0.3.0：固定、可验证、可恢复的 PWF owned runtime
  ↓
提取通用能力：
  - Managed Hook policy 安装与合并
  - runtime provenance / allowlist / manifest
  - Codex Cloud Host 输入输出契约
  - subprocess supervision 与 output budget
  - diagnostics、doctor、repair、rollback
  - Cloud fixtures 与黑盒 testkit
  ↓
适配第二个低风险、只读 Plugin，验证抽象是否真实可复用
```

即使将来泛化，也不应该自动执行任意 Skill 中的脚本。每个 adapter 至少应具有：

- 明确的上游仓库、版本、commit 和 archive SHA；
- runtime 文件 allowlist 与逐文件哈希；
- 许可证和第三方来源记录；
- 允许启用的 Hook event 清单；
- 解释器、依赖、路径和 timeout 契约；
- Cloud fixtures、canary 和回滚方案；
- 管理员或维护者人工审核。

如果你希望贡献第二个适配器，请先在 Issue 中提供：Hook manifest、runtime 依赖、
本地路径假设、需要的 lifecycle events、是否读写项目、预期 Cloud 行为以及可重复测试。

## 快速开始与验证

### 本地开发检查

```bash
npm test
python3 -m py_compile hooks/hook_adapter.py
node --check install.js
python3 tools/import_upstream_runtime.py check
python3 tools/build_release.py check \
  --archive dist/pwf-codex-cloud-hooks-v0.3.0-alpha.1.zip
bash -n init-cloud-sandbox-v0.3.0.bash
git diff --check
```

这些测试使用临时 Codex home 和 requirements 文件，不应修改真实 `/opt/codex` 或
`/etc/codex/requirements.toml`。

### Installer 示例

```bash
# 只预览，不写入
node install.js install --dry-run --json \
  --codex-home /absolute/test/codex \
  --skill-root /absolute/planning-with-files \
  --managed-requirements /absolute/test/requirements.toml

# Cloud 生产路径示例
node install.js install --dry-run --json --codex-home /opt/codex
sudo node install.js install --json --codex-home /opt/codex
node install.js doctor --json --codex-home /opt/codex
sudo node install.js install --repair --json --codex-home /opt/codex
sudo node install.js uninstall --json --codex-home /opt/codex
```

不要从移动的 `main`、`latest` 或未校验下载直接执行生产安装。发布和 Cloud 黑盒步骤
请阅读：

- [英文规范 README](README.md)；
- [项目心智模型](PROJECT_UNDERSTANDING.md)；
- [Cloud 黑盒验证手册](黑盒验证.md)；
- [v0.3.0-alpha.1 Cloud smoke](docs/v0.3.0-alpha.1-cloud-smoke.md)；
- [Phase 1 Runtime 契约](docs/phase-1-runtime-contracts.md)。

## 维护者入口

开始维护前按顺序阅读：

1. `PROJECT_UNDERSTANDING.md`；
2. `.planning/.active_plan`；
3. 活动计划的 `task_plan.md`、`progress.md`、`findings.md`；
4. `work_plan.md`；
5. `README.md`、`黑盒验证.md`；
6. `git status --short --branch`。

Agent 和维护者的唯一规范指令文件是 [`AGENTS.md`](AGENTS.md)。本仓库不提供翻译版
`AGENTS.*.md`，以避免两份维护规则发生漂移或被误认为具有相同优先级。

## License

本仓库使用 MIT License。重新分发的上游 runtime 保留完整 MIT 声明，见
[`THIRD_PARTY_NOTICES.md`](THIRD_PARTY_NOTICES.md)。
