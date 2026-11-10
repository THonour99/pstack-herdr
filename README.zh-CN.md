[English](README.md) | 简体中文

# pstack-herdr

这是 [cursor/plugins](https://github.com/cursor/plugins) 里 pstack 插件的 fork（作者 Lauren Tan，MIT），改成可以用 herdr 把 pstack 的角色派给别的 agent 程序。上游插件的说明在 [pstack/README.md](pstack/README.md)。`pstack/LICENSE` 没有改。本仓库（含 fork 的修改）使用的许可是 [LICENSE](LICENSE)。

## 这个 fork 改了什么

- 角色值 `hc:<program>` 和 `hc:<program>:<model>` 通过 herdr 的 `hc` 派发，不再走 Cursor 的 Task。Codex 的写法是 `hc run --bg --name <name> --cwd <git-repo> codex exec -m <model> ...`。不在受信任的 git 仓库里时，Codex 要求在 `codex exec` 上加 `--skip-git-repo-check`。
- `setup-pstack` 用 `command -v` 检测已安装的 agent：`claude`、`codex`、`grok`、`pi`，外加 `hc` 和 `herdr`。只有找到 `herdr` 时才会把它写进列表。角色表按实际检测到的程序来写。没有 `hc` 时，表保持上游的 no-`hc` 表。
- `hc`、Claude、Codex、Grok、Pi 都在时，写代码的角色用 `hc:codex`，调研角色用 `hc:grok`，swarm workers 用 `hc:pi`，评审面板是 Claude 加上 `hc:codex` 和 `hc:grok`。Pi 不进面板。
- 只找到 `hc` 和 Codex 时，用仅 Codex 的表：`hc:codex:gpt-6-astra`、`hc:codex:gpt-6.1-sol`、`hc:codex:gpt-6-luna`。判断类角色留在 Astra；如果当前编排器已经是 Astra，这些角色改成 `inherit-parent`。Luna 只接写明了具体文件、具体改动和一条验证命令的任务。
- `harness/pstack/claude-code/` 是生成的 Claude Code 树。`harness/pstack/codex/` 是生成的 Codex 树，带 `.codex-plugin/plugin.json`，仓库 marketplace 在 `.agents/plugins/marketplace.json`。

## 安装

### Claude Code

[pstack/README.md](pstack/README.md) 里写的是上游 marketplace 的命令：

```bash
/plugin marketplace add cursor/plugins
/plugin install pstack@cursor-plugins
```

这两条装的是 [cursor/plugins](https://github.com/cursor/plugins) 上的 pstack，不是这个 fork。本仓库的 Claude Code 树是 `harness/pstack/claude-code`，`.claude-plugin/marketplace.json` 指向它。把这个私有仓库登记成 Claude Code marketplace 这件事没有验证过。

### Codex

在已经能用 git 读取这个私有仓库的 shell 里：

```bash
codex plugin marketplace add THonour99/pstack-herdr
```

然后：

```bash
codex
/plugins
```

安装 `pstack`，新开一个 session，再跑 setup-pstack。文档里还有 `codex plugin marketplace list`、`codex plugin marketplace upgrade` 和 `codex plugin marketplace remove`。这些页面没有写 shell 上的 `codex plugin install`。Codex 的 IDE 扩展不加载插件。

`codex plugin marketplace add` 对这个私有仓库是否成功，没有验证过。Git 仍然需要读权限。有一份第三方说法：在 Windows 上，`codex plugin marketplace add owner/repo` 会失败在 Codex 自己的 git staging clone 里，而先 `git clone` 再在检出目录执行 `codex plugin marketplace add .` 可以。这里没有复查过那份说法。

如果 marketplace 安装用不了，把 `harness/pstack/codex/skills` 复制或软链到 `$HOME/.agents/skills`，或者仓库里的 `.agents/skills`。

未验证的事项列在 [pstack/docs/harness/codex.md](pstack/docs/harness/codex.md) 和 [pstack/docs/harness/claude-code.md](pstack/docs/harness/claude-code.md)。
