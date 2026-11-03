# pnpm workspace security updates with systeminformation

First test whether Dependabot creates a root security PR for a workspace member pinning `systeminformation@5.33.15`, using the default versioning strategy. This changes only the dependency compared with the earlier pnpm 12 lodash reproduction.

After recording that result, test `versioning-strategy: lockfile-only` separately. The initial commit included that setting, which was removed before any alerts or security update jobs were observed.

The root pins pnpm 12.11.2 and contains the shared two-document `pnpm-lock.yaml`. CI uses Node.js 24 and runs `pnpm install --frozen-lockfile --ignore-scripts`.

## Reproduce

1. Publish these files in a public GitHub repository
2. Enable Dependabot alerts and security updates
3. Wait for alerts against the workspace manifest and root lockfile
4. Inspect generated PRs, security update logs, and frozen-install checks

Scheduled version PRs are disabled to isolate security updates.

Expected: one security PR updates the workspace manifest and root lockfile together and passes CI.

Related issue: https://github.com/dependabot/dependabot-core/issues/16680.

## Hosted result (November 3, 2026)

GitHub eventually detected alerts against both the workspace manifest and the root lockfile. A manual alert refresh was requested while the repository had no alerts.

The [initial commit installs successfully](https://github.com/karlhorky/repro-dependabot-pnpm12-workspace-security-updates-lockfile-only/actions/runs/41661938424). With the default versioning strategy, Dependabot opens:

1. [PR #1](https://github.com/karlhorky/repro-dependabot-pnpm12-workspace-security-updates-lockfile-only/pull/1), commit `09f07e3d7fdc97a8c87e0852f2e7b3adfc4c8c42`, changing only `packages/app/package.json` from systeminformation 5.33.15 to 5.33.16. [CI fails](https://github.com/karlhorky/repro-dependabot-pnpm12-workspace-security-updates-lockfile-only/actions/runs/41664386315)
2. [PR #2](https://github.com/karlhorky/repro-dependabot-pnpm12-workspace-security-updates-lockfile-only/pull/2), commit `f7531bdaf5ee6b2601afb8944d72e1ff69ce78c4`, changing both the workspace manifest and root lockfile. [CI passes](https://github.com/karlhorky/repro-dependabot-pnpm12-workspace-security-updates-lockfile-only/actions/runs/41664558518/job/126016294884)

The [workspace security job](https://github.com/karlhorky/repro-dependabot-pnpm12-workspace-security-updates-lockfile-only/actions/runs/41664227684) and [root security job](https://github.com/karlhorky/repro-dependabot-pnpm12-workspace-security-updates-lockfile-only/actions/runs/41664226169) both completed successfully.

PR #1's actual pnpm output, with timestamps removed:

```text
$ pnpm install --frozen-lockfile --ignore-scripts
Scope: all 2 workspace projects
Error: ERR_PNPM_OUTDATED_LOCKFILE

  × installing dependencies
  ╰─▶ Cannot install with "frozen-lockfile" because pnpm-lock.yaml is not up
      to date with package.json.
      
        Failure reason:
        specifiers in the lockfile don't match specifiers in package.json:
      * in importers["packages/app"]:
      * 1 dependency is mismatched:
        - systeminformation (lockfile: 5.33.15, manifest: 5.33.16)
  help: Regenerate the lockfile with `pnpm install --lockfile-only` so that
        pnpm-lock.yaml reflects the current package.json, then re-run `pnpm
        install --frozen-lockfile`.

Process completed with exit code 1.
```

The dependency-only test therefore reproduces the same two-PR outcome as lodash. In the [separate lodash strategy-only test](https://github.com/karlhorky/repro-dependabot-pnpm12-lodash-workspace-security-updates-lockfile-only), adding only `versioning-strategy: lockfile-only` makes the root security job fail and leaves only the broken workspace PR. That exact-pin strategy behavior is reported in https://github.com/dependabot/dependabot-core/issues/16809.
