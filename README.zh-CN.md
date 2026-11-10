[English](README.md) | 简体中文

# pstack-herdr

这是 [cursor/plugins](https://github.com/cursor/plugins) 里 pstack 插件的 fork。pstack 由 Lauren Tan 编写，使用 MIT 许可。这个 fork 保留该插件，并通过 herdr 派发角色。上游插件说明在 [pstack/README.md](pstack/README.md)。指南在 [pstack/docs/guide/README.md](pstack/docs/guide/README.md)。

## 目录

- [这个 fork 改了什么](#这个-fork-改了什么)
- [派发怎么工作](#派发怎么工作)
- [需要先装什么](#需要先装什么)
- [安装](#安装)
- [快速开始](#快速开始)
- [默认角色表](#默认角色表)
- [技能](#技能)
- [改一个角色](#改一个角色)
- [出了问题](#出了问题)
- [上游和许可](#上游和许可)
- [未验证](#未验证)

## 这个 fork 改了什么

上游 pstack 把角色交给 Cursor 的 Task，模型 slug 类似 `grok-4.7-xhigh-fast` 或 `claude-opus-5-5-xhigh`。这个 fork 增加了 `hc:<program>` 和 `hc:<program>:<model>`。这些值用 `hc` 开一个 herdr 窗口，不再开 Task。

`setup-pstack` 对 `claude`、`codex`、`grok`、`pi` 运行 `command -v`，也对 `hc` 和 `herdr` 运行。角色表按找到的命令来写。不要求一定有 `herdr`。`# agents:` 行只在 `command -v` 找到 `herdr` 时才列出它。没有 `hc` 时，每个角色都留在上游那张表上，不写 `hc:` 值。

`hc`、Claude、Codex、Grok、Pi 都装了时，写代码的角色用 `hc:codex`，调研角色用 `hc:grok`，swarm workers 用 `hc:pi`。面板是 Claude 加上 `hc:codex` 和 `hc:grok`。Pi 不进面板。

只装了 `hc` 和 Codex 时，表使用 `hc:codex:gpt-6-astra`、`hc:codex:gpt-6.1-sol` 和 `hc:codex:gpt-6-luna`。

`harness/pstack/claude-code/` 是生成的 Claude Code 树。`harness/pstack/codex/` 是生成的 Codex 树。Codex 树带 `.codex-plugin/plugin.json`。仓库 marketplace 是 `.agents/plugins/marketplace.json`。

## 派发怎么工作

规则写在 [pstack/skills/setup-pstack/SKILL.md](pstack/skills/setup-pstack/SKILL.md)，以及 [pstack/skills/poteto-mode/SKILL.md](pstack/skills/poteto-mode/SKILL.md) 的 Subagents 一节。

值 `hc:<program>` 不开 Task。父会话先读 `hc --skill`，再开一个有名字的窗口。

```bash
hc run --bg --name <short-name> <program> "<brief>. When done run: hc send $HC_TERMINAL_ID <result>"
```

父会话用 `hc wait <short-name>` 等，用 `hc read` 读，还可以用 `hc send` 继续说。然后父会话自己核对结果。四个 agent 都在时，父会话是 Claude。只有 Codex 时，父会话是当前 Codex session。日常验收就是这个父会话按简报把验证跑一遍。父会话不把子窗口自己的汇报当成验收。

`hc:<program>:<model>` 多一个模型参数。Codex 已核对过的写法是 `codex exec -m <model>`。

```bash
hc run --bg --name <short-name> --cwd <git-repo> codex exec -m gpt-6.1-sol "<brief>. When done run: hc send $HC_TERMINAL_ID <result>"
```

不带模型的 `hc:codex` 就是这条命令去掉 `-m`。Pi 的 `hc:pi:<model>` 传 `pi --model <model>`。核对过的 Pi 参数是 `--model`、`--thinking` 和 `-p`。不带模型的 `hc:pi` 不传 `--model`。别的 Pi 参数没有核对过，`--thinking` 的取值也没有。`claude` 和 `grok` 的模型参数没有核对过。不要编一个。不带模型参数去跑那个程序，并说明模型没有传下去。

写代码的模型不审查自己的代码。审查角色不复用写代码的那个窗口。四个 agent 的表上，`hc:codex` 写出的代码由 Claude、Grok，或两者一起审查。Claude 写出的代码由 Codex 审查。只有 Codex 的表上，`gpt-6.1-sol` 写的由 `gpt-6-astra` 审查，`gpt-6-astra` 写的由 `gpt-6.1-sol` 审查。同一家的面板比几家混在一起的面板更容易意见一致。正式审查是 [interrogate](pstack/skills/interrogate/SKILL.md)，用 `interrogate reviewers` 那一行。

`hc:pi` 是 DeepSeek。它只接写明了具体文件、具体改动和一条验证命令的修改。说不那么精确，就不要派给 `hc:pi`。装了 Codex 就改用 `hc:codex`，否则用 `inherit-parent`，并说出来。Pi 不用于探索、调查、reflect，也不进任何审查面板。

没有 `hc` 时，要说出来，这个角色改用 `inherit-parent`。缺一个程序时，要说出用了哪条回退。不要闷头把活做了。

- 没有 `codex`。写代码的角色留在 `grok-4.7-xhigh-fast`。面板里去掉 `hc:codex`。
- 没有 `grok`。`how explorer`、`why investigators`、`reflect tooling` 改成 `inherit-parent`。面板里去掉 `hc:grok`。
- 没有 `pi`。找到了 Codex 时，`swarm workers` 改成 `hc:codex`，否则是 `grok-4.7-xhigh-fast`。
- 没有 `claude`，但找到了 Codex，而且编排器还不是 `gpt-6-astra`。每个 `claude-opus-5-5-xhigh` 改成 `hc:codex:gpt-6-astra`。编排器已经是 `gpt-6-astra` 时，用 `inherit-parent`。

`gpt-6.1-terra` 是 Codex 里做写作、看图和操作电脑的模型。不要把它放进写代码的角色。

## 需要先装什么

`hc` 是启动器。没有它，setup 就保持上游角色表。

`herdr` 这个命令不必在 `PATH` 上。检测仍会跑 `command -v herdr`，只有命令能解析到才记下来。

setup 查找的 agent 命令是 `claude`、`codex`、`grok` 和 `pi`。Pi 跑的是 DeepSeek。命令名 `pi` 只写在 [setup-pstack](pstack/skills/setup-pstack/SKILL.md) 的那一张表里。

在 Windows 上，检测用 Git Bash 里的 `command -v`。`hc`、`codex` 和 `pi` 可以是 npm shim，分别是 `.ps1`、`.cmd` 和无扩展名文件。`hc run` 直接启动这些 shim，不需要再绕到 `.cmd` 或 `.exe`。这是在 Codex CLI 0.167.0 上核对过的，命令是 `hc run --bg --name x codex exec -m gpt-6-luna "<task>"` 和 `hc run --bg --name x pi ...`。

不在受信任的 git 仓库里时，Codex 会退出，并打印 `Not inside a trusted directory and --skip-git-repo-check was not specified`。把 `hc run --cwd` 指到正在改的仓库，或者在 `codex exec` 上加 `--skip-git-repo-check` 并说出来。优先用 `--cwd`。

Codex CLI 0.167.0 的默认是推理力度 `xhigh`、approval `never`、sandbox `workspace-write`。这样派出去的 Codex 改文件时不会先问。简报必须写明可以动哪些文件，以及不许动什么。

## 安装

### Claude Code

[pstack/README.md](pstack/README.md) 里的命令是上游 marketplace 的命令。

```bash
/plugin marketplace add cursor/plugins
/plugin install pstack@cursor-plugins
```

这两条装的是 [cursor/plugins](https://github.com/cursor/plugins) 上的 pstack，不是这个 fork。本仓库的 Claude Code 树是 `harness/pstack/claude-code`。`.claude-plugin/marketplace.json` 指向 `./harness/pstack/claude-code`。把这个私有仓库登记成 Claude Code marketplace 没有验证过。

[pstack/README.md](pstack/README.md) 写明 Claude Code 会给插件命令加前缀，所以 `/poteto-mode` 是 `/pstack:poteto-mode`，setup 是 `/pstack:setup-pstack`。这条命令写入 `~/.claude/rules/pstack-models.md`。

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

安装 `pstack`，新开一个 session，再跑 setup。文档里还有 `codex plugin marketplace list`、`codex plugin marketplace upgrade` 和 `codex plugin marketplace remove`。这些页面没有写 shell 上的 `codex plugin install`。Codex 的 IDE 扩展不加载插件。

`codex plugin marketplace add` 对这个私有仓库是否成功，没有验证过。Git 仍然需要读权限。有一份第三方说法：在 Windows 上，`codex plugin marketplace add owner/repo` 会失败在 Codex 自己的 git staging clone 里，而先 `git clone` 再执行 `codex plugin marketplace add .` 可以。这里没有复查过那份说法。

如果 marketplace 安装用不了，把 `harness/pstack/codex/skills` 复制或软链到 `$HOME/.agents/skills`，或者仓库里的 `.agents/skills`。

在 Codex 里用哪个名字启动 `setup-pstack`，没有验证过。技能一旦在跑，会把角色块写进 Codex 读取的全局说明文件。设置了 `CODEX_HOME` 且 override 文件非空时，文件是 `$CODEX_HOME/AGENTS.override.md`。否则是 `$CODEX_HOME/AGENTS.md`。没设置 `CODEX_HOME` 时是 `~/.codex/AGENTS.md`。技能只替换 `<!-- pstack-models:start -->` 和 `<!-- pstack-models:end -->` 之间的那一块。

## 快速开始

1. 按你用的宿主安装对应的树。见[安装](#安装)。
2. 做第一件实事之前先跑 setup。在 Claude Code 里，[pstack/README.md](pstack/README.md) 写的命令是 `/pstack:setup-pstack`。在 Codex 里，调用名没有验证过。见[未验证](#未验证)。
3. setup 检测 `hc` 和各 agent CLI，询问预算，展示角色表，你接受之后才写入规则。四个预算标签是 `unlimited`、`large`、`medium`、`small`。还没有规则时，`large` 对上技能默认值。`inherit-parent` 和 `auto` 表示这个角色跑在父会话的模型上。选预算不会改 `hc:` 值。
4. 用 poteto-mode 开工。在 Claude Code 里，插件 README 写的命令是 `/pstack:poteto-mode`。

例子：四个 agent 都装了，父会话是 Claude Code。你描述一个 bug，再调用 poteto-mode。[bug-fix 剧本](pstack/skills/poteto-mode/playbooks/bug-fix.md) 先自己复现，再把修复派给配置好的 `bug-fix` 角色。四个 agent 的表上，这个角色是 `hc:codex`，所以父会话为 `codex` 开一个 herdr 窗口，送出简报，等待，再读结果。剧本里的 `how` 用的是 `how explorer`，也就是 `hc:grok`。修复跨过函数边界时先跑 `architect`，`architect runners` 是 `claude-opus-5-5-xhigh`、`hc:codex` 和 `hc:grok`。然后父会话自己跑验证。Codex 那个窗口不审查自己的 diff。审查由 Claude、Grok 或两者做。有争议的设计交给 `interrogate`，审查者就是同样这三项。`swarm workers` 只有在简报写明具体文件、具体改动和一条验证命令时才是 `hc:pi`。否则父会话说明 Pi 不合适，改用 `hc:codex`。

做功能时走 [feature 剧本](pstack/skills/poteto-mode/playbooks/feature.md)。写代码的代表用 `feature, refactoring` 角色。在这张表上它同样是 `hc:codex`。`how explorer` 仍是 `hc:grok`。

例子：只装了 `hc` 和 Codex。父会话是 Codex session。bug-fix 或 feature 的代表是 `hc:codex:gpt-6.1-sol`。`how explorer`、`why investigators` 和 `reflect tooling` 也是 `hc:codex:gpt-6.1-sol`。判断、行文、最难的任务、解释者和综合者是 `hc:codex:gpt-6-astra`。编排器已经是 `gpt-6-astra` 时，这些角色是 `inherit-parent`。`swarm workers` 只在修改已经写全时才是 `hc:codex:gpt-6-luna`，否则是 `hc:codex:gpt-6.1-sol`。父会话仍然自己跑验证。Sol 写的由 Astra 审查。Astra 写的由 Sol 审查。

这一页不展示工具输出的样例。上面的角色名就是 [setup-pstack](pstack/skills/setup-pstack/SKILL.md) 里的那些行。

## 默认角色表

四个 agent，并且有 `hc`。不带模型后缀。

| 角色 | 值 |
| --- | --- |
| `feature, refactoring` | `hc:codex` |
| `bug-fix` | `hc:codex` |
| `perf-issue` | `hc:codex` |
| `hillclimb` | `hc:codex` |
| `judgment and prose` | `claude-opus-5-5-xhigh` |
| `hardest tasks` | `claude-opus-5-5-xhigh` |
| `how explorer` | `hc:grok` |
| `how explainer` | `claude-opus-5-5-xhigh` |
| `why investigators` | `hc:grok` |
| `why synthesizer` | `claude-opus-5-5-xhigh` |
| `reflect tooling` | `hc:grok` |
| `reflect judgment, divergent, synthesizer` | `claude-opus-5-5-xhigh` |
| `arena runners` | `claude-opus-5-5-xhigh`, `hc:codex`, `hc:grok` |
| `arena cross-judge pool` | `claude-opus-5-5-xhigh`, `hc:codex`, `hc:grok` |
| `swarm workers` | `hc:pi` |
| `architect runners` | `claude-opus-5-5-xhigh`, `hc:codex`, `hc:grok` |
| `interrogate reviewers` | `claude-opus-5-5-xhigh`, `hc:codex`, `hc:grok` |

只有 Codex，并且有 `hc`。编排器已经是 `gpt-6-astra` 时，五行判断角色用 `inherit-parent`，不用 `hc:codex:gpt-6-astra`。`swarm workers` 只在修改已经写全时才是 `hc:codex:gpt-6-luna`，否则是 `hc:codex:gpt-6.1-sol`。

| 角色 | 值 |
| --- | --- |
| `feature, refactoring` | `hc:codex:gpt-6.1-sol` |
| `bug-fix` | `hc:codex:gpt-6.1-sol` |
| `perf-issue` | `hc:codex:gpt-6.1-sol` |
| `hillclimb` | `hc:codex:gpt-6.1-sol` |
| `judgment and prose` | `hc:codex:gpt-6-astra` |
| `hardest tasks` | `hc:codex:gpt-6-astra` |
| `how explorer` | `hc:codex:gpt-6.1-sol` |
| `how explainer` | `hc:codex:gpt-6-astra` |
| `why investigators` | `hc:codex:gpt-6.1-sol` |
| `why synthesizer` | `hc:codex:gpt-6-astra` |
| `reflect tooling` | `hc:codex:gpt-6.1-sol` |
| `reflect judgment, divergent, synthesizer` | `hc:codex:gpt-6-astra` |
| `arena runners` | `hc:codex:gpt-6-astra`, `hc:codex:gpt-6.1-sol` |
| `arena cross-judge pool` | `hc:codex:gpt-6-astra`, `hc:codex:gpt-6.1-sol` |
| `swarm workers` | `hc:codex:gpt-6-luna` |
| `architect runners` | `hc:codex:gpt-6-astra`, `hc:codex:gpt-6.1-sol` |
| `interrogate reviewers` | `hc:codex:gpt-6-astra`, `hc:codex:gpt-6.1-sol` |

在这几个 Codex 模型里，Astra 能力最高。Sol 接近 Astra，更便宜。Luna 适合一条已经说清的、可重复的修改。其他安装组合从四个 agent 的表出发，再套用[派发怎么工作](#派发怎么工作)里的回退。setup 会说出它选了哪张表。

没有 `hc` 时用上游的表。写代码、调研和 swarm workers 是 `grok-4.7-xhigh-fast`。判断和综合留在 `claude-opus-5-5-xhigh`。面板是 `claude-opus-5-5-xhigh` 和 `grok-4.7-xhigh-fast`。

## 技能

每一句都是该技能自己的 description，缩短过。文件在 [pstack/skills](pstack/skills)。

- [poteto-mode](pstack/skills/poteto-mode/SKILL.md)。回答简洁但具体，有意使用子代理，行文去掉 AI 腔，代码简单，并且要验证。在 Claude Code 里，插件 README 写的命令是 `/pstack:poteto-mode`。
- [poteto-help](pstack/skills/poteto-help/SKILL.md)。帮助完成 setup、使用 `/poteto-mode`，以及挑选技能、剧本或原则。技能里的命令是 `/poteto-help`。
- [swarm](pstack/skills/swarm/SKILL.md)。派出 N 个并行 worker，收齐之后交回一份报告。worker 使用 `swarm workers` 角色。
- [arena](pstack/skills/arena/SKILL.md)。对同一任务派出 N 个候选，选定一个底子，再把其他候选里最强的部分嫁接进去。runner 来自 `arena runners`。
- [interrogate](pstack/skills/interrogate/SKILL.md)。对手尝试弄坏一处改动。只有一次运行证明了的发现才留下。审查者来自 `interrogate reviewers`。
- [architect](pstack/skills/architect/SKILL.md)。写代码之前先勾出类型、签名和模块结构。runner 来自 `architect runners`。
- [tdd](pstack/skills/tdd/SKILL.md)。只在你明确要求 TDD 或先写失败测试时使用，或者这个 bug 有一条便宜的本地测试。测试会很贵或说不清时跳过。
- [unslop](pstack/skills/unslop/SKILL.md)。从文字里去掉 AI 腔。
- [no-comments](pstack/skills/no-comments/SKILL.md)。派出 Comment Sicko，修掉接受的发现，并为声称的约束提供编码方式。
- [control-cli](pstack/skills/control-cli/SKILL.md)。搭一套本地工具，驱动、查看并分析交互式 CLI 或 TUI，不依赖外部服务。
- [control-ui](pstack/skills/control-ui/SKILL.md)。搭一套本地浏览器或 CDP 工具，驱动并查看 web、IDE 或 Electron 界面。
- [deslop](pstack/skills/deslop/SKILL.md)。去掉 AI 生成的代码废话，并收拾代码风格。

`how`、`why` 和 `reflect` 是 `how explorer`、`why investigators` 和 `reflect tooling` 背后的调研技能。文件是 [how](pstack/skills/how/SKILL.md)、[why](pstack/skills/why/SKILL.md) 和 [reflect](pstack/skills/reflect/SKILL.md)。

## 改一个角色

再跑一次 setup。它会列出每个角色，问你是接受这张表，还是改某一个角色。选项是检测到的模型、每一个合法的 `hc:<program>` 值、`inherit-parent` 和 `auto`。`inherit-parent` 和 `auto` 让这个角色跑在父会话的模型上。在 Cursor 里，这表示不传 Task 的 `model`。走 `hc:` 派发时，这个角色就在父会话里做，不再开一个窗口。

也可以直接改规则文件。删掉一行角色，就会退回技能默认值。Cursor 里的文件是 `~/.cursor/rules/pstack-models.mdc`。Claude Code 里是 `~/.claude/rules/pstack-models.md`。Codex 里是[安装](#安装)所说的 AGENTS 文件中的那一块。新 session 才会读到修改。setup 写完时会这样说。

以后再跑时，setup 会保留你改成「不是本技能会写的默认值」的角色。一行如果仍然等于旧的 herdr 默认值，就不算你的修改。特别是，`swarm workers` 还停在 `hc:codex`、同时又找到了 Pi 时，会改成 `hc:pi`，并且 setup 会说明它改了这一行。

## 出了问题

`command -v hc` 找不到。setup 保持上游的表，并且应该说明没有 `hc`。不要写 `hc:` 值。

Codex 打印 `Not inside a trusted directory and --skip-git-repo-check was not specified`。把 `hc run --cwd` 指到 git 仓库，或者在 `codex exec` 上加 `--skip-git-repo-check` 并说出来。

缺一个 agent CLI。用[派发怎么工作](#派发怎么工作)里那个程序的回退，并说出来。不要把角色留在一个没找到的程序的 `hc:` 值上。

任务要 Pi 自己决定改什么。不要把这个任务派给 `hc:pi`。改派给 `hc:codex` 或 `inherit-parent`，并说出来。

子窗口做完了，父会话还没有跑验证。任务还没验收。父会话按简报把验证跑一遍。

## 上游和许可

pstack 来自 [cursor/plugins](https://github.com/cursor/plugins)。[pstack/LICENSE](pstack/LICENSE) 没有改，其中写着 `Copyright (c) 2026 Lauren Tan`。本仓库（含 fork 的修改）使用的许可是 [LICENSE](LICENSE)。它保留那一行，并加上 `Copyright (c) 2026 THonour99`。两份都是 MIT 许可。

`control-cli`、`control-ui` 和 `deslop` 从上游 `cursor-team-kit` 原样复制。那个目录的 LICENSE 也是 MIT，版权行是 `Copyright (c) 2026 Cursor`。许可与上游相同。这三个文件没有写 Cursor 的 Task、Bugbot 或 cursor.com。`control-ui` 仍提到 `verify-this`，那是 `cursor-team-kit` 里的另一个技能，这次没有一起复制。

这个 fork 的历史是 cursor/plugins 的历史，再加上本仓库上的提交。把以后的上游提交合并进来的固定命令，没有验证过。

## 未验证

- 把这个私有仓库登记成 Claude Code marketplace 没有验证过。`/plugin marketplace add cursor/plugins` 装的是上游 pstack，不是这个 fork。
- `codex plugin marketplace add THonour99/pstack-herdr` 对这个私有仓库是否成功，没有验证过。Git 仍然需要读权限。
- 有一份第三方说法：在 Windows 上，`codex plugin marketplace add owner/repo` 会失败在 Codex 自己的 git staging clone 里，而先 `git clone` 再执行 `codex plugin marketplace add .` 可以。这里没有复查过那份说法。
- 文档里的 Codex 命令是 `codex plugin marketplace list`、`upgrade` 和 `remove`。这些页面没有写 shell 上的 `codex plugin install`。Codex 的 IDE 扩展不加载插件。
- 在 Codex 里用哪个名字启动 `setup-pstack`，没有验证过。
- 把以后的 [cursor/plugins](https://github.com/cursor/plugins) 提交合并进来的固定命令，没有验证过。
- `claude` 和 `grok` 的模型参数没有核对过。别的 Pi 参数，以及 `--thinking` 的取值，也没有核对过。
- 更多未验证事项列在 [pstack/docs/harness/codex.md](pstack/docs/harness/codex.md) 和 [pstack/docs/harness/claude-code.md](pstack/docs/harness/claude-code.md)。
