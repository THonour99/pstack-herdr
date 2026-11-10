---
name: setup-pstack
description: Configure which models pstack uses per role and at what reasoning budget. Detects your available models and writes an always-applied rule that overrides the skill defaults. Use for /setup-pstack, "configure pstack models", "pstack budget", or changing pstack's model choices.
---

# Setup pstack

Write `~/.cursor/rules/pstack-models.mdc`, an always-applied rule that sets pstack's model per role.

## Agent CLI list

The only place a local agent command name is written. Detection, the `# agents:` line, and every `hc:<program>` value use this list. Change a command here, then change the filled-in `hc:` values that spell it. Do not copy the list into another skill.

- `claude` (Claude, Max)
- `codex` (Codex, GPT)
- `grok` (Grok)
- `pi` (Pi, running DeepSeek)

## Steps

### 1. Detect available models

Enumerate the model slugs you can pass to a `Task` subagent in this session. That is the dependable source. If Cursor also exposes a models API or CLI that lists the user's entitled models, prefer it for completeness. If you cannot detect any, ask the user to paste the slugs they have access to. Never write a real slug you have not confirmed is available. The aliases `inherit-parent` and `auto` are always valid even though they are not detected slugs.

Also detect the agent CLIs on this machine. Run `command -v` for `hc`, `herdr`, and each command in the agent CLI list. Record every command that resolves. A value `hc:<program>` is valid only when both `hc` and that program's command were found, the same way a detected slug is valid. Never write an `hc:` value for a program you have not confirmed. If `hc` does not resolve, skip every `hc:` default below. Role lines stay the no-`hc` shape in step 5, the same as upstream, even when the other commands resolve.

### 2. Load current state

The default role-to-model mapping is the no-`hc` shape in step 5, unless step 3 says otherwise. If `~/.cursor/rules/pstack-models.mdc` already exists, read it and treat its `# budget` line and its role values as the current choices. Re-detect agents on every run. The `# agents:` comment is informational and is rewritten from this run's detection. Otherwise start from those defaults. A line whose role is not in step 5, such as `how critics`, is from a retired role. Drop it.

### 3. Budget, map, and confirm

**(a) Ask for a budget.** Prefer AskQuestion over free text. Offer these four options with these exact labels, and name the current budget when the rule records one. With no rule, say that `large` matches the skill defaults.

- `unlimited — max reasoning`
- `large — xhigh reasoning`
- `medium — high reasoning`
- `small — medium reasoning`

**(b) Apply it.** Build the working table from the skill defaults, and on a re-run keep a role the user set to something other than a default this skill writes. A role that still matches an older herdr default is not a user change. When `pi` was found, `swarm workers` left on `hc:codex` becomes `hc:pi`. Say what you replaced. When there is no existing rule and `hc` was found, start from the `hc` defaults below instead of the no-`hc` shape in step 5. Each missing program falls back on its own. Say which fallback you used. Never write an `hc:` value for a missing program, and never apply a fallback silently.

- `feature, refactoring`, `bug-fix`, `perf-issue`, and `hillclimb` use `hc:codex` when `codex` was found. When `codex` is missing, keep the skill default `grok-4.7-xhigh-fast` and say that Codex is missing.
- `how explorer`, `why investigators`, and `reflect tooling` use `hc:grok` when `grok` was found. When `grok` is missing, use `inherit-parent` and say that Grok is missing.
- `judgment and prose`, `hardest tasks`, `how explainer`, `why synthesizer`, and `reflect judgment, divergent, synthesizer` stay on `claude-opus-5-5-xhigh`.
- `swarm workers` uses `hc:pi` when `pi` was found. When `pi` is missing and `codex` was found, use `hc:codex` and say that Pi is missing so swarm workers moved to Codex. When both are missing, keep the skill default `grok-4.7-xhigh-fast` and say that Pi and Codex are missing.
- Panel roles (`arena runners`, `arena cross-judge pool`, `architect runners`, `interrogate reviewers`) are `claude-opus-5-5-xhigh`, then `hc:codex` if `codex` was found, then `hc:grok` if `grok` was found. Do not add Pi. DeepSeek is not a panel model and is not used for exploration, investigation, or reflect. When `codex` or `grok` is missing, omit that entry and say so. When both are missing, the panel is the Claude entry alone. Say so.

When `hc` was not found, the defaults are unchanged from upstream. `unlimited`, `large`, `medium`, and `small` set the effort token of every real slug, panel entries included, to `max`, `xhigh`, `high`, or `medium`. The effort token is the last token, or the one before a trailing `fast`, on the ladder `max` > `xhigh` > `high` > `medium` > `low`. If the result is not a detected slug, use the same family's detected slug with the highest effort at or below the target, else mark the role as needing a choice. `inherit-parent`, `auto`, and `hc:<program>` do not change. So `unlimited` turns `claude-opus-5-5-xhigh` into `claude-opus-5-5-max`. Grok slugs top out at `xhigh`, so under `unlimited` the fallback puts Grok at `xhigh` and keeps `grok-4.7-xhigh-fast` as it is. `large` keeps both defaults. `small` turns them into `claude-opus-5-5-medium` and `grok-4.7-medium-fast`.

**(c) Show the roles and confirm.** Show every role with its model, marking any real slug not in the detected set as needing a choice. Also list each line step 2 dropped, and the agents found in step 1. Ask whether to accept as-is or change specific roles, offering the detected models, each valid `hc:<program>` value, plus `inherit-parent` and `auto` (both mean this role runs on the parent chat model, which is how Auto users stay on Auto) as the options. Prefer AskQuestion over free text. For panel roles (arena runners, architect runners, interrogate reviewers) the value is a list, and one worker runs per entry, alias and `hc:` entries included, so the list length sets the count. `arena cross-judge pool` is also a list, but Arena runs one judge. It tries only values from a family other than the parent's, in list order, so order it by preference. An `inherit-parent` or `auto` value never judges. An `hc:<program>` value may judge when it is a different program from the parent. `swarm workers` is the default model for every worker unless a race or comparison assigns another model per arm.

**`hc:pi` (DeepSeek).** DeepSeek is good at explicit, well-specified edits and weak when it has to decide what to change itself. The parent is Claude. Dispatch to `hc:pi` only when the task names the exact files, the exact change, and a concrete verification command. If the task cannot be specified that precisely, do not send it to `hc:pi`. Run it on `hc:codex` when Codex was found, otherwise `inherit-parent`, and say so. The parent still verifies the result by running that verification.

**Review.** The model that wrote the code never reviews it. Code written by `hc:codex` is reviewed by Claude, by Grok, or by both. Code written by Claude is reviewed by Codex. Day-to-day acceptance is the parent Claude checking the result against the task brief by running the verification itself. Formal multi-model review is **interrogate**, and it uses the `interrogate reviewers` panel above. That panel does not include `hc:pi`.

### 4. Validate

Every real slug written must be in the detected set. `inherit-parent` and `auto` always pass. An `hc:<program>` value passes only when both `hc` and that program's command from the agent CLI list were detected in step 1. If a chosen real slug or `hc:` value is not available, stop and ask again.

### 5. Write the rule

Write `~/.cursor/rules/pstack-models.mdc` with `alwaysApply: true`, a `# agents:` comment listing whatever step 1 resolved (in this order when present: `hc`, `herdr`, then each command in the agent CLI list, or `none` if nothing resolved), a `# budget` line with the chosen label and its target effort, and one line per role, using the same labels poteto-mode uses. Overwrite the whole file so re-runs stay idempotent. When `hc` was not found, the role lines stay the no-`hc` shape below. When `hc` was found and every command in the agent CLI list resolved, use the second shape. When `hc` was found but a command is missing, use the second shape with that program's fallback from step 3. Shape when `hc` is absent:

```
---
description: pstack per-role model choices (overrides skill defaults)
alwaysApply: true
---
# pstack model configuration. One line per role. Delete a line to fall back to the skill default.
# `inherit-parent` or `auto` as a value: the role runs on the parent chat model (omit Task `model`). Alias entries in a panel list still count toward its fan-out.
# `hc:<program>` as a value: do not spawn a Task subagent. Open a named herdr window with that program, then verify the result yourself. See the Subagents section of poteto-mode.
# `hc:pi` (DeepSeek) only gets a task that names the exact files, the exact change, and a concrete verification command. If you cannot specify it that precisely, do not use `hc:pi`. Use `hc:codex`, or `inherit-parent` when Codex is missing, and say so. The parent still runs the verification.
# The writer never reviews its own code. Code from `hc:codex` is reviewed by Claude and/or Grok. Code from Claude is reviewed by Codex. Day-to-day acceptance is the parent Claude running the verification against the brief. Formal review is interrogate on the interrogate reviewers panel. `hc:pi` is not on that panel.
# agents: none
# budget: large (xhigh)
feature, refactoring: grok-4.7-xhigh-fast
bug-fix: grok-4.7-xhigh-fast
perf-issue: grok-4.7-xhigh-fast
hillclimb: grok-4.7-xhigh-fast
judgment and prose: claude-opus-5-5-xhigh
hardest tasks: claude-opus-5-5-xhigh
how explorer: grok-4.7-xhigh-fast
how explainer: claude-opus-5-5-xhigh
why investigators: grok-4.7-xhigh-fast
why synthesizer: claude-opus-5-5-xhigh
reflect tooling: grok-4.7-xhigh-fast
reflect judgment, divergent, synthesizer: claude-opus-5-5-xhigh
arena runners: claude-opus-5-5-xhigh, grok-4.7-xhigh-fast
arena cross-judge pool: claude-opus-5-5-xhigh, grok-4.7-xhigh-fast
swarm workers: grok-4.7-xhigh-fast
architect runners: claude-opus-5-5-xhigh, grok-4.7-xhigh-fast
interrogate reviewers: claude-opus-5-5-xhigh, grok-4.7-xhigh-fast
```

Shape when `hc` was found and every command in the agent CLI list resolved. Keep the same frontmatter and comment lines, set `# agents:` to the detected list, and keep the same `# budget` line. `hc:pi` is the Pi command from that list. It is not a second list.

```
# agents: hc, herdr, claude, codex, grok, pi
# budget: large (xhigh)
feature, refactoring: hc:codex
bug-fix: hc:codex
perf-issue: hc:codex
hillclimb: hc:codex
judgment and prose: claude-opus-5-5-xhigh
hardest tasks: claude-opus-5-5-xhigh
how explorer: hc:grok
how explainer: claude-opus-5-5-xhigh
why investigators: hc:grok
why synthesizer: claude-opus-5-5-xhigh
reflect tooling: hc:grok
reflect judgment, divergent, synthesizer: claude-opus-5-5-xhigh
arena runners: claude-opus-5-5-xhigh, hc:codex, hc:grok
arena cross-judge pool: claude-opus-5-5-xhigh, hc:codex, hc:grok
swarm workers: hc:pi
architect runners: claude-opus-5-5-xhigh, hc:codex, hc:grok
interrogate reviewers: claude-opus-5-5-xhigh, hc:codex, hc:grok
```

### 6. Confirm

Tell the user the rule was written and that it applies to new sessions. Re-running this skill updates it.

### 7. Offer a verification skill (optional)

Check whether the project has a way to drive the real app for proof (a `verify-*` skill, or an existing harness). If not, offer once: "want a project-local verification skill, so agents can drive the app the way a user does and prove changes work? I can generate one with /create-verification-skill." On yes, invoke `/create-verification-skill` (resolves wherever pstack is installed: workspace, user, or plugin). On no, move on without pushing.

### 8. Offer a formal verification skill (optional)

Check whether the project already has a formal verification skill (a project skill that runs a model checker, a proof, or another verifier against the code). If not, offer once: "want a formal verification skill, so agents can prove a key property of your code holds in every case in scope, not only the cases tests reach? I can generate one with /create-formal-verification-skill." On yes, invoke `/create-formal-verification-skill` (resolves wherever pstack is installed: workspace, user, or plugin). On no, move on without pushing.
