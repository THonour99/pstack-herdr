# pstack in Claude Code

pstack skills use Cursor names. In Claude Code, read this file once per session, then map each Cursor name in a skill with the tables below. `<pstack>` means the folder that holds `skills/` and `docs/`. If a Cursor field has no row, drop it and say so. Never pass a field or value the tool schema rejects.

## Tools

| Cursor | Claude Code |
| --- | --- |
| `Task` | `Agent` |
| `AskQuestion` | `AskUserQuestion`. In `-p` mode it is absent, so ask in plain text. |
| Open a todolist | Use `TodoWrite` if the session offers it. Otherwise keep the list in your reply and update it at each phase. |
| Run another pstack skill (`/name` or "the **name** skill") | `Read` `<pstack>/skills/<name>/SKILL.md` and follow it. The `Skill` tool refuses most pstack skills. When you tell the user to type it, write `/pstack:<name>`. |
| `create-skill` | Cursor built-in for authoring a SKILL.md. It is not in this repo. No Claude Code equivalent was verified. Follow the playbook in prose and say the built-in is absent. |
| `/loop` | Cursor built-in. It is not in this repo. No Claude Code equivalent was verified. Skip with the Cursor-only line. |
| Custom Mode, option+enter or alt+enter | Cursor feature. No Claude Code equivalent was verified. Attach the skill for one message instead, and say so. |
| `Grep`, `Glob` | `Bash` with `rg` when the session has no `Grep` or `Glob`. |

## Subagent fields

| Cursor | Claude Code |
| --- | --- |
| `subagent_type: generalPurpose` | `subagent_type: "general-purpose"` |
| `poteto-agent`, `comment-sicko` | `pstack:poteto-agent`, `pstack:comment-sicko` |
| `readonly: true` | `subagent_type: "Explore"`. It keeps `Bash`, so the brief also says "do not edit files", and you check `git status` afterward. |
| `readonly: false` | `general-purpose`. It keeps the session's MCP servers. |
| `environment: "cloud"` | `isolation: "worktree"`. Name the branch or commit the worker checks out. |
| `environment: "local"` | Omit. |
| `cloud_base_branch: <b>` | Tell the worker to check out `<b>` first. |
| `run_in_background: true` | Omit. Subagents run in the background by default and notify you when they finish. |
| `resume` | Spawn a new agent and paste the earlier result into its brief. |
| `file_attachments` | List the paths in the brief. |

## Models

| Cursor | Claude Code |
| --- | --- |
| `claude-<family>-<version>-<effort>[-fast]`, such as `claude-opus-5-5-xhigh` | `model: "opus"` and `effort: "xhigh"`. `model` takes only `opus`, `sonnet`, `haiku`, `fable`. Drop `-fast`. |
| A `grok-*`, `gpt-*`, or other non-Claude slug | If `hc` and a program matching the slug family are installed (`grok-*` to `grok`, `gpt-*` to `codex`), treat the role as `hc:<program>`. Otherwise omit `model` so the agent runs on the session model, and say so. |
| `hc:<program>`, such as `hc:codex`, `hc:grok`, or `hc:pi` | Do not spawn an Agent subagent. Open a named herdr window with `hc run --bg --name <short-name> <program> ...`, brief it, then `hc wait` and `hc read`. Verify the result yourself. Command names come from the agent CLI list in `setup-pstack`. `hc:<program>:<model>`, such as `hc:codex:gpt-6.1-sol`, is the same dispatch. For `codex`, use `hc run --bg --name <short-name> --cwd <git-repo> codex exec -m <model> "..."`. Verified on Codex CLI 0.167.0. Codex exits with `Not inside a trusted directory and --skip-git-repo-check was not specified` outside a trusted git repo. Pass `hc run --cwd` pointed at that repo, or add `--skip-git-repo-check` to `codex exec` and say so. Those defaults on that CLI are reasoning effort `xhigh`, approval `never`, and sandbox `workspace-write`, so the child edits files without prompting. The brief must name the files in scope and what it must not change. For `pi`, verified flags are `--model`, `--thinking`, and `-p`. Pass `--model` only for `hc:pi:<model>`. Do not invent any other Pi flag or a `--thinking` level. For `claude` or `grok`, a model flag was not verified. Do not invent one. Run that program with no model argument and say so. `hc:pi` is DeepSeek. Send it only a task that names the exact files, the exact change, and a concrete verification command. If you cannot specify it that precisely, do not use `hc:pi`. Use `hc:codex`, or `inherit-parent` when Codex is missing, and say so. The parent still runs the verification. The writer never reviews its own code. Code from `hc:codex` is reviewed by Claude and/or Grok. Code from Claude is reviewed by Codex. On the Codex-only table, a `gpt-6.1-sol` writer is reviewed by `gpt-6-astra`, and a `gpt-6-astra` writer by `gpt-6.1-sol`. Day-to-day acceptance is the parent running the verification against the brief. Formal review is interrogate on the interrogate reviewers panel, which does not include `hc:pi`. If `hc` is missing, say so and fall back to `inherit-parent`. If the program is missing, say so and use that role's setup fallback (`hc:pi` to `hc:codex` when Codex exists, else `inherit-parent`; `hc:grok` to `inherit-parent`; `hc:codex` to the skill default). Never do the work silently. |
| `auto` or `inherit-parent` | Omit `model`. |
| The `pstack-models.mdc` rule, `~/.cursor/rules/pstack-models.mdc` | `~/.claude/rules/pstack-models.md`. Claude Code loads it into every new session, so its role lines are already in context. If they are not, read the file. If the file or the role's line is missing, use the skill's default slug. |
| A role value `<model> <effort>`, such as `opus high` | `model: "opus"` and `effort: "high"`. |

## setup-pstack

Claude Code loads every `.md` file in `~/.claude/rules/` into each new session, so `setup-pstack` writes its rule there. Follow its steps with these changes:

- Detect. The detected models are the `model` values that the `Agent` tool schema lists, each with an effort from its `effort` values. Write a real model as `<model> <effort>`, such as `opus xhigh`. The effort is the last token, so the budget step works as written. Also run `command -v` for `hc`, `herdr`, and each command in the agent CLI list in `setup-pstack`. Do not copy that list. `hc:<program>` is a valid value only when both `hc` and that command were found. If `hc` is missing, keep the no-`hc` role lines, mapped with the rows above. That is the upstream behavior.
- Defaults. When `hc` is absent, map each default slug with the rows above. `claude-opus-5-5-xhigh` is `opus xhigh`, and `grok-4.7-xhigh-fast` is `inherit-parent`. When `hc`, `claude`, `codex`, `grok`, and `pi` are all found, keep the four-agent table. Judgment, prose, hardest tasks, and synthesizer roles stay on the Claude model. `feature, refactoring`, `bug-fix`, `perf-issue`, and `hillclimb` use `hc:codex`. `how explorer`, `why investigators`, and `reflect tooling` use `hc:grok`. `swarm workers` uses `hc:pi`. Panel roles are the Claude model, then `hc:codex`, then `hc:grok`. Do not add a `:<model>` suffix on that table. When `hc` and `codex` are found and `claude`, `grok`, and `pi` are all missing, use the Codex-only table in `setup-pstack` (`hc:codex:gpt-6-astra`, `hc:codex:gpt-6.1-sol`, and `hc:codex:gpt-6-luna`). Any other mix uses the substitutions in `setup-pstack`. Say which table and which fallback you used. Never apply one silently.
- Write. Write `~/.claude/rules/pstack-models.md` with no frontmatter, then the comment lines, a `# agents: <detected list>` line, the `# budget` line, and one line per role from step 5. Overwrite the whole file. Never put the role lines in `CLAUDE.md` or `AGENTS.md`.

## Paths

| Cursor | Claude Code |
| --- | --- |
| `.cursor/skills/`, `~/.cursor/skills/` | `.claude/skills/`, `~/.claude/skills/` |
| `~/.cursor/plugins/`, `.cursor-plugin/plugin.json` | `~/.claude/plugins/`, `.claude-plugin/plugin.json` |
| `.cursor/rules/*.mdc`, always-applied rules | `CLAUDE.md`, or `AGENTS.md` when no `CLAUDE.md` exists. Claude Code never reads `.mdc` files. The model rule is the exception. See Models. |
| `https://github.com/cursor/plugins/blob/main/pstack/<path>` | `<pstack>/<path>` when that file exists. The plugin ships the skills, `docs/guide/`, and `README.md`. Read the local copy and never fetch it. Give the user the URL when they need a link. |
| Transcripts in `~/.cursor/projects/<slug>/agent-transcripts/` | `~/.claude/projects/<slug>/<session>.jsonl`. The slug is the workspace path with each `/` and `.` changed to `-`, so `/Users/you/proj` is `-Users-you-proj`. The newest `.jsonl` directly in that folder is the current session. Its `<session>/subagents/` folder holds subagent transcripts. Read only this workspace's folder. |

## Cursor-only features

Claude Code has no Cursor Automations, Bugbot, Cursor cloud agents, or built-in PR tool. When a step needs one, put one line in your reply, `Skipped in Claude Code: <step>. <reason>.`, then go on to the next step. Do not stop, ask, or retry. When a later step needs what the skipped step made, such as a URL or a key, leave a placeholder and name it in the line.

| Cursor | Claude Code |
| --- | --- |
| Cursor Automations: routines, `update_state` routine calls, `api2.cursor.sh/automations` webhooks, routine wakes, and `SendToUser` secret requests | Skip with the line. |
| Bugbot | Bugbot comments on a GitHub PR are ordinary review threads, so triage them with the forge command that the step gives. Skip a step that starts, waits for, or configures Bugbot when the PR has no Bugbot check or comment. |
| Cursor cloud agents | `environment: "cloud"` maps to a worktree, as in Subagent fields. Skip a step that needs the cloud agent itself, such as its dashboard status or its own machine. |
| A built-in PR tool | Use the forge command that the step gives, such as `gh pr create`. That is not a skip, so write no line. |
