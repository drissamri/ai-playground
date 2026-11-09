# ai-playground
Playground for services built by AI coding agents on the devbox

## Agent skills

A selection of [Matt Pocock's skills](https://github.com/mattpocock/skills) (MIT, see `.claude/skills/LICENSE-mattpocock-skills`), pinned to release `v1.3.2` in `skills-lock.json`, for Claude Code. Installed with [skills.sh](https://skills.sh) (needs Node 22.20 or newer):

```bash
DO_NOT_TRACK=1 npx -y skills@1.7.6 add https://github.com/mattpocock/skills/tree/v1.3.2 -a claude-code --copy -y \
  -s setup-matt-pocock-skills -s grill-with-docs -s grilling -s domain-modeling -s to-spec -s to-tickets \
  -s implement -s tdd -s codebase-design -s code-review -s diagnosing-bugs -s retro -s writing-for-agents
```

Updating means a new tag in that command and a PR, so every change to what the agents follow is reviewed. Codex will only review (read-only) and does not need these skills; it reads `.agents/skills/`, so add `-a codex` if that changes.

| Phase | Skill |
|---|---|
| Once per repo | `setup-matt-pocock-skills` |
| Sharpen the idea | `grill-with-docs` (uses `grilling`, `domain-modeling`) |
| Spec and tickets | `to-spec`, `to-tickets` |
| Build | `implement` (one ticket, one branch, one PR), `tdd` (uses `codebase-design`) |
| Review | `code-review` (standards + spec). It replaces the built-in `/code-review` here; the built-in bug review still runs as `/review` |
| Hard bugs | `diagnosing-bugs` |
| Close | `retro` |
| Docs for agents | `writing-for-agents` |

Left out on purpose: `implement-spec` (one big integration branch instead of small PRs), `git-guardrails-claude-code` (blocks every push, so no PRs), `grill-me` (duplicate), `handoff`, `pr`, `wayfinder`, `triage` and the rest.

## Guardrails

`.claude/settings.json` denies, in Claude Code, pushing to `main`, a bare `git push`, force-pushing, deleting or mirroring branches, and `gh pr merge`. Deny rules win over every permission mode, including `--dangerously-skip-permissions`.

They stop the usual way to write these commands, not every way: `git -C . push origin main` or `sh -c '...'` is not matched ([Claude Code docs](https://code.claude.com/docs/en/permissions.md#bash-rule-limits)). They catch mistakes; they are not a security boundary. The only real block is a GitHub ruleset on `main` (require a pull request, no force-push, no deletion), which on a private repo needs GitHub Pro.
