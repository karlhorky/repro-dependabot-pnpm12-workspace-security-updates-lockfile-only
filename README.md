# pnpm workspace security updates with lockfile-only

Test whether Dependabot creates a root security PR when a pnpm workspace member pins `systeminformation@5.33.15` and `dependabot.yml` sets `versioning-strategy: lockfile-only`.

The root pins pnpm 12.11.2 and contains the shared two-document `pnpm-lock.yaml`. CI uses Node.js 24 and runs `pnpm install --frozen-lockfile --ignore-scripts`.

## Reproduce

1. Publish these files in a public GitHub repository
2. Enable Dependabot alerts and security updates
3. Wait for alerts against the workspace manifest and root lockfile
4. Inspect generated PRs, security update logs, and frozen-install checks

Scheduled version PRs are disabled to isolate security updates.

Expected: one security PR updates the workspace manifest and root lockfile together and passes CI.

Related issue: https://github.com/dependabot/dependabot-core/issues/16680.
