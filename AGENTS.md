# AGENTS.md (Windows)

Canonical global rules for Cursor, Claude, Codex, and OpenCode. A repo `AGENTS.md` overrides this file when they conflict. Tool-only deltas stay in `~/.codex/AGENTS.md`, `~/.cursor/AGENTS.md`, and `~/.config/opencode/AGENTS.md`. Do not paste this file into those. OpenCode loads this body via `~/.config/opencode/opencode.json` `instructions`; the opencode file is a delta only.

## 1. Windows

- Assume Windows and PowerShell. Use Windows paths.
- Run the work. Do not hand the user a command unless they asked for the command.
- WSL: do not call `powershell.exe` or `cmd.exe` from the sandbox.
- `Documents\` is Google Drive synced. Before a Docker build or a reparse-sensitive tool, use the `gdrive-reparse` skill.
- Windows stdout is cp1252. Keep Python stdout ASCII (`OK`, `DONE`, `ERROR`).
- Do not put remote bash inside a PowerShell double-quoted `ssh`. Use the `powershell-ssh` skill.
- Line endings: a repo-local `.gitattributes`, not a global Git EOL rule. `eol=lf` for code and docs; `eol=crlf` only for `*.bat` and `*.cmd`.

## 2. How to work

- The user relies on the agent for coding, review, debugging, architecture, security, and git.
- Comments in code stay short. Long explanations go in the chat.
- Duplicate two or three times before extracting a helper.
- Keep a visible todo list for non-trivial work, and mark items done when they are done.
- Ask a short question when scope, risk, or success criteria are unclear.
- **Reusable code.** Before writing new code, search this repo and the existing skills for something that already does the job. Extend that, or extract a small shared function, instead of a one-off that only fits this call site. If the same need appears twice, promote it to a named function, script, or skill with a clear input and output. Leave a one-off only when the behaviour is truly local and a shared helper would hide that.
- **Next steps.** End every substantive reply with at least 5 concrete follow-ups, plus at least 3 new paths that are not the next micro-step. Skip only a one-line acknowledgment.
- **Subagents.** See § 10.

## 3. Secrets

- Never print secrets, and never commit `.env`, API keys, tokens, or credentials.
- Prefer `secrets-mcp`. Never print secret values. Details, folder taxonomy, and the CLI pin: skill `/vaultwarden-secrets` and `vaultwarden-infra` (SoT). Pin Bitwarden CLI **2026.8.0** at `%LOCALAPPDATA%\secrets-mcp\bin\bw.exe`. Do not use 2026.7.x or 2026.9.0+ against this Vaultwarden. 2024.12.x is fallback only.
- Login `/vaultwarden-login`. Lock `/vaultwarden-lock`. No TOTP in `.env`.
- Any script that touches a secret value runs through `~/.agents/skills/vaultwarden-secrets/scripts/Invoke-SecretScript.ps1` (errors reduced to exception type, output redacted). Every commit on this machine is scanned by gitleaks (global `core.hooksPath` `~/.config/git/hooks`). Never bypass it with `--no-verify`.
- Do not expose ports without a reason. Default to local-only.

## 4. Git

- The agent runs fetch, merge, branch, commit, and push. No rebase and no force-push unless the user asks. Pull with `git pull --no-rebase`.
- GitHub trunk is `main`. GitLab trunk is `trunk`. Verify with the remote.
- Author email: GitHub `scholtzap@gmail.com`, GitLab `arnos@plentify.co.za`.
- **Feature work on a branch:** short-lived `feat/` `fix/` `docs/` `chore/` from an up-to-date trunk. Never build a feature on trunk.
- **Scratch stays out of repos:** probes, temp scripts, outputs and logs go in the session scratchpad.
- If the tree is dirty before a merge, stash or commit first.
- **End every turn clean** for files you changed: commit (only your files) and push, delete or ignore junk, or say in one line why it stays. Skip secrets and other-agent dirt. Stop hook `~/.claude/hooks/git_hygiene_guard.py` enforces this. Skill `repo-hygiene` (one repo: branch fit, junk, ignore lines, pruning); `git-fleet-audit` (all repos).
- Agent config (this file, rules, hooks, settings) is versioned in private repo scholtzap/agent-config via `~/.claude/scripts/agent-config.sh` (bare repo, work tree $HOME); commit config changes with it.

## 5. Docs and knowledge files

- Same turn: patch the existing ops doc, runbook, SPEC, `AGENTS.md`, or `MISTAKES.md`. Do not add a new markdown file when one already fits.
- **CI owns KB sync.** Commit the docs. Do not run `/doc-kb-save` or the sync scripts by hand. A manual run from a feature branch publishes unreviewed content. Use `/doc-kb-save` only when the user explicitly asks, or the repo has no sync pipeline.
- `MISTAKES.md` at each git root, newest first, when a mistake is known. The stop hook checks this. Skill: `repo-wishlist` is for the wishlist, not the mistakes log.
- `WISHLIST.md` at each git root for missing tooling and for facts you had to hunt. Not a human task list. Hook: `.cursor/hooks/wishlist_audit.py`.
- `ARCHITECTURE.md` at each git root. Read §§1–3 before a wide search. Skill: `repo-architecture`. Graphify does not replace the map.
- **`docs/plans/` is the work queue.** Every plan, TODO, deferred item and handover goes there as its own file with full frontmatter (`type`, `status`, `priority`, `rank`), then `python scripts/ops/refresh_plans_index.py --write` — and then **read the diff to confirm your row is in `PLANS_INDEX.md`**. `--write` rewrites the index with a fresh timestamp whether or not it indexed anything, so a dropped item looks like a clean run. Never leave a plan in a chat message, a TODO comment or a scratch file. Found a stray plan (plan frontmatter outside `docs/plans/`) or an older one? `python scripts/ops/audit_plans.py` lists strays; read an existing plan before starting a new one on the same topic, and supersede explicitly rather than duplicating. Full rule: `~/.claude/rules/plans-system.md` (loads when you touch `docs/plans/`). No registry in the repo yet? Offer `/doc-kb-setup` (Phase 3.5) or `/plans-audit`; do not install it unasked. Skill: `plans-audit`. Hook: `.cursor/hooks/plans_registry_audit.py`.

## 6. Plentify

When the task is company architecture, an internal API, or proprietary business logic, query the Plentify MCP before guessing.

**Google Drive / Sheets / Docs (read + write):** MCP `google-workspace` (workspace-mcp). Every tool takes `user_google_email`: `arnos@plentify.co.za` (Plentify) or `scholtzap@gmail.com` (personal). Scripts reuse the same grant via `~/.agents/skills/google-workspace-automation/scripts/board_rows.py`; it appends or updates rows on the **Agent Projects Board** (`16ApeiYY7SH4HOuzb_w8yC7z7ZbpIcqvYa2WWWuaQbgE`, tab Tasks) by Task ID. Never stage rows for the user to paste. Install/repair for every agent: `vaultwarden-infra/scripts/ops/install_google_workspace_mcp.ps1`. The claude.ai Drive connector is read/create only and can't edit cells.

## 7. Skills

Use the installed skill catalog. Do not paste skill lists into this file. Before a non-trivial answer, check whether a skill already covers it. If you are unsure which one, use `skill-router`.

## 8. Repository memory

When the repo has `.memory/main.sqlite`: load recent rows at session start (`repo-memory-load`), save durable facts during the work (`repo-memory-save`), and initialize with `repo-memory-initialization` if the database is missing. MCP server: `project-memory`. Do not store secrets. Notion stays a separate store. Policy file: `~/.claude/NOTION_AGENT_MEMORY.md`.

## 9. Graphify

`/graphify` invokes the `graphify` skill before any other step.

## 10. Subagents

Default to a subagent when the work is independent, would flood the parent, or is a specialist pass. Skip a single edit already in context, and skip when setup costs more than doing it inline.

Every spawn gets three things: relevant context (the worker does not see this chat), a concrete definition of done, and fail criteria (when to stop or escalate). Never pass secrets into a worker prompt.

**Model fit.** The parent already holds the session. Give a worker only a task it can finish from the brief. Do not spawn work that needs the parent's transcript, open files, or a long diff. Pick the smallest model for that narrow job. Keep orchestration in the parent.

**Parallelise.** Independent tasks start together. Each worker gets a small brief. Do not serialize work that does not share files or results.

**Session limit.** A `PreToolUse` hook (`~/.claude/hooks/subagent_limit_guard.py`) refuses new spawns while this session's usage limit is still in force. If it blocks you, do the work inline. Do not turn the hook off to get around a real limit. Escape hatch: `CLAUDE_SUBAGENT_GUARD=off`.
