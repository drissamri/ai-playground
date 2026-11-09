# ai-playground

Playground for services built by AI coding agents on the devbox. What to build is tracked as GitHub issues.

## Working rules

- Work on a branch and open a pull request. Never push to `main`, never force-push, never merge: a human reviews and merges every PR.
- One ticket per branch and per PR. Keep PRs small.
- Test first (`/tdd`), and run the tests before you call work done.
- Everything in the repo, the issues and the PRs is in English.
- No secrets, tokens or personal data in code, commits, issues or PRs.

## Agent skills

### Issue tracker

Issues and specs are GitHub issues in `drissamri/ai-playground`, through `gh`. See `docs/agents/issue-tracker.md`.

### Domain docs

Single-context: one `GLOSSARY.md` and `docs/adr/` at the repo root, created when needed. See `docs/agents/domain.md`.
