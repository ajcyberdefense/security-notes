# Claude Code settings credential remediation

Date: 2026-11-08  
Scope: 14 repositories available through the connected GitHub account

## Trigger

Apiiro reported that Claude Code can store the full text of a shell command in `.claude/settings.local.json` when a user chooses **Yes, and don't ask again**. If the command contains a credential, that credential can be committed with the settings file and remain recoverable from Git history after the file is deleted.

## Audit performed

1. Enumerated all 14 repositories available to the connected GitHub account, including public and private repositories.
2. Searched every default branch for:
   - `.claude/settings.local.json`
   - `.claude/settings.json`
3. Enumerated the commit history returned by GitHub for each repository and inspected the changed-file list for every returned commit.
4. Inspected 276 commits in total without printing file patches or potential credential values.
5. Checked any matching Claude settings patch for common credential indicators. No matching settings files were found, so no credential values were accessed and no provider-specific rotation was required.

## Findings

- Current default-branch matches: **0**
- Historical commit matches: **0**
- Credential indicators found in the target files: **0**
- Credentials rotated: **0**, because the audit found no exposed credential to identify or rotate

## Remediation applied

Every connected repository now contains this Git ignore rule:

```gitignore
# Claude Code local settings may contain approved shell commands and secrets
**/.claude/settings.local.json
```

Existing `.gitignore` files were preserved and updated. A new `.gitignore` was created only where one did not already exist. `.claude/settings.json` was deliberately not ignored because it is Claude Code's shared, reviewable project configuration; it must still be kept free of secrets.

Post-change verification confirmed the ignore rule in all 14 repositories and confirmed that neither target settings file is present on any default branch.

## Operating practice going forward

- Never place passwords, tokens, keys, or connection strings directly in an approved shell command.
- Put secrets in a secret manager or the process environment, and reference only the variable name in commands.
- Prefer narrowly scoped permission rules such as a command name and fixed safe arguments over saving a command containing runtime values.
- Review `.claude/settings.json` like executable configuration before merging it; it can define permissions, hooks, environment entries, and integrations.
- If a secret is ever found in Git history, revoke or rotate it first, then remove it from current files and rewrite history where appropriate. History rewriting does not invalidate an already exposed secret.

## Local-machine follow-up completed

The local root was audited separately after command execution became available.

- 13 local Git repositories and linked worktrees were inventoried.
- 3,416 commits reachable from local refs were checked with full-history pathspecs.
- No tracked, staged, nonignored-untracked, or historical target files were found.
- Two ignored dependency copies of `.claude/settings.local.json` were found under `node_modules`; neither contained a recognized credential indicator and neither was tracked.
- Git's standard per-user global excludes file now contains `**/.claude/settings.local.json`.
- Root and nested local settings paths were verified ignored in every local repository.
- Shared `.claude/settings.json` remained unignored in every compatibility check.

No credential rotation was required because neither the connected GitHub audit nor the local audit found a credential in either target settings path. Five uninitialized submodules were absent locally, and remote-only or unreachable Git objects remain outside the local-ref audit.
