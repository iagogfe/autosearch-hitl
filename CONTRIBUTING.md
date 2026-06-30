# Contributing

Thanks for your interest in improving **autosearch-hitl**! This is a small,
focused repo — contributions are welcome.

## Repository layout

```
skills/<name>/SKILL.md   # a skill (YAML frontmatter: name + description) + reference files
docs/                    # design spec, implementation plan, executive summary
.github/workflows/       # ci.yml (PR validation) + release.yml (auto release notes)
```

The main skill lives in `skills/autosearch-hitl/` (`SKILL.md` router +
`layer-llm.md` + `engine-general.md`).

## Development setup

The skill is just Markdown — no build or runtime. To try your changes with an agent:

```bash
# from your clone/branch, install into your agent locally for testing
npx skills add <your-user>/<your-fork> --skill autosearch-hitl
# or copy the folder into your agent's skills dir, e.g. Claude Code:
cp -r skills/autosearch-hitl ~/.claude/skills/autosearch-hitl
```

## Commit messages — Conventional Commits (required)

Release notes and version bumps are **generated automatically** from commit
messages, so please follow [Conventional Commits](https://www.conventionalcommits.org/),
**in English**:

| Prefix | Use for | Version bump |
| --- | --- | --- |
| `feat:` | new capability | minor |
| `fix:` | bug fix | patch |
| `docs:` | documentation | patch |
| `refactor:`, `perf:`, `test:`, `ci:`, `build:`, `chore:` | other changes | patch |

Examples:
```
feat: add stagnation-window override flag
fix: handle repos without a git worktree
docs: clarify the pre-condition gatekeeper
```

## Pull requests

1. Fork the repo and create a branch.
2. Make your change. If you touch a skill, keep its `SKILL.md` frontmatter valid
   (`name` + `description`) — the **CI checks this on every PR**.
3. Use Conventional Commit messages (English).
4. Open the PR with a short description of the *what* and *why*.

PRs run a read-only validation workflow; no secrets are exposed to fork PRs.

## Releases

Maintainers don't tag by hand. On push to `master`, the release pipeline
(`git-cliff`) computes the next version, regenerates `CHANGELOG.md`, and publishes a
GitHub Release — all from the Conventional Commit history.

## Security

Please report vulnerabilities privately — see [SECURITY.md](./SECURITY.md). Do not
open public issues for security problems.

## License

By contributing, you agree that your contributions are licensed under the
[MIT License](./LICENSE).
