# renovate-config

Shareable Renovate config presets used across LilyCalico's repositories.

Dependencies are managed with a hybrid of Renovate and Dependabot Alerts.

| Tool              | Role                                                                       |
| ----------------- | -------------------------------------------------------------------------- |
| Renovate          | Detects updates, opens PRs, handles scheduling, grouping and automerge     |
| Dependabot Alerts | Detects and surfaces vulnerabilities (Version / Security Updates disabled) |

## Presets

| Preset         | Purpose                                | How to reference                           |
| -------------- | -------------------------------------- | ------------------------------------------ |
| `default.json` | Base settings shared by every repo     | `local>LilyCalico/renovate-config:default` |
| `nextjs.json`  | Next.js / React projects               | `local>LilyCalico/renovate-config:nextjs`  |

### default.json

| Setting           | Value                                                                      |
| ----------------- | -------------------------------------------------------------------------- |
| Schedule          | Weekdays, 9:00–18:00 (Europe/Stockholm)                                    |
| minimumReleaseAge | Only versions released at least 7 days ago                                 |
| Grouping          | All patch updates in a single PR                                           |
| Automerge         | Patch updates, minor / patch of `@types/*`, minor / patch of linters and formatters |
| Concurrent PRs    | Up to 10                                                                   |
| Security          | Vulnerability fixes are opened immediately, ignoring the schedule, with the `security` label |

### nextjs.json

Extends `default` and groups each of the following into a single PR:

- Next.js (`next`, `@next/*`, `eslint-config-next`)
- React (`react`, `react-dom`, `@types/react`, `@types/react-dom`)
- Storybook (`storybook`, `@storybook/*`)
- TanStack (`@tanstack/*`)

## Setup

> [!IMPORTANT]
> Merge the `renovate.json` from step 2 into the default branch **before** granting the Renovate app access in step 3.
> `renovate-config` is a reserved repository name: Renovate implicitly applies `default.json` to any repository without a `renovate.json`, so automerge and everything else would start running right away.

1. **Enable Dependabot Alerts**
   In Settings → Advanced Security, enable `Dependabot alerts`.
   Leave `Dependabot security updates` and `Dependabot version updates` disabled (Renovate opens the update PRs).

2. **Add `renovate.json` to the repository root and merge it**

   ```json
   {
     "$schema": "https://docs.renovatebot.com/renovate-schema.json",
     "extends": ["local>LilyCalico/renovate-config:nextjs"]
   }
   ```

   Repository-specific settings can override the preset in the same file:

   ```json
   {
     "$schema": "https://docs.renovatebot.com/renovate-schema.json",
     "extends": ["local>LilyCalico/renovate-config:nextjs"],
     "ignoreDeps": ["some-problematic-package"]
   }
   ```

3. **Grant the Renovate app access to the repository**
   Go to https://github.com/apps/renovate → Configure → Only select repositories and add the repository.
   On first setup you are redirected to the Mend Developer Portal: choose **Renovate Only**, then **Scan and Alert**.

> [!NOTE]
> Do not grant the Renovate app access to this repository (`renovate-config`). It is public, so the presets can be read without access.

## Automerge requirements

Renovate automerges a PR only when all of its status checks pass.
Repositories without any status checks are never automerged.

If you enable GitHub's `Allow auto-merge`, also require status checks in branch protection. Without required checks, PRs can be merged before CI finishes.

## Adding or changing presets

- Add a new preset as `<name>.json` at the root and inherit the base settings with `"extends": ["local>LilyCalico/renovate-config:default"]`
- Changes affect every repository, so make them through a PR and merge only after the `Validate Renovate presets` workflow passes
- Validate locally: `npx --package renovate -- renovate-config-validator --strict --no-global *.json`
