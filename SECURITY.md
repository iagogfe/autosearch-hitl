# Security Policy

## Supported Versions

The latest released version receives security updates.

| Version | Supported |
| --- | --- |
| latest (`0.x`) | ✅ |
| older | ❌ |

## Reporting a Vulnerability

Please **do not** open a public issue for security problems.

Use GitHub's private vulnerability reporting:
**Security → Report a vulnerability**, or
<https://github.com/iagogfe/autosearch-hitl/security/advisories/new>.

You'll get an acknowledgement within a few days, and we'll coordinate a fix and
disclosure from there.

## Scope & posture

This project is an **agent skill** (Markdown instructions) plus a release pipeline
(GitHub Actions). Relevant notes:

- The skill runs with your agent's permissions — review it before use, like any skill.
- The release workflow (`.github/workflows/release.yml`):
  - triggers **only** on `push` to `master` and `workflow_dispatch` — never
    `pull_request`/`pull_request_target`, so untrusted fork code never runs with the
    repository token;
  - uses least-privilege permissions (`contents: write`, scoped to the job);
  - pins third-party actions by **commit SHA** (mitigates tag-move supply-chain attacks);
  - verifies the downloaded `git-cliff` binary by **SHA-256 checksum**.
- The PR validation workflow (`.github/workflows/ci.yml`) runs with read-only
  permissions and no secrets.
