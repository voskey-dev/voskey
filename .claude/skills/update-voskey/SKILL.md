---
name: update-voskey
description: Use when asked to update voskey to the latest upstream Misskey release, sync with misskey-dev/misskey, resolve upstream merge conflicts, or prepare container publishing after an upstream update. Preserves voskey changes, sets the root version to the Misskey release plus -voskey, and guides GitHub Actions publishing.
---

# Update Voskey

Update this repository against the latest upstream Misskey release while preserving voskey-specific behavior. Work from the repository root on local `main` unless the user explicitly requests another branch.

Follow [AGENTS.md](../../../AGENTS.md). This skill does not authorize unrelated workflow changes, commits, pushes, or external publishing.

## 1. Confirm repository state

- Confirm the current branch is `main` (or the branch requested by the user).
- Confirm `origin` points to `voskey-dev/voskey` and `upstream` to `misskey-dev/misskey`.
- Check that the working tree is clean, or that existing changes are understood and preserved.
- If unrelated changes would interfere with the merge, stop and ask rather than overwriting them.

## 2. Identify the latest upstream release

Determine the latest release from GitHub Releases for `misskey-dev/misskey`, or use:

```bash
gh release list --repo misskey-dev/misskey --limit 1
git fetch upstream --tags
git rev-list -n 1 <release-tag>
```

Record the release tag and resolved commit. Use the release version later as the version base, for example `2025.12.1`.

## 3. Merge the release commit

Merge the resolved release commit, not an arbitrary branch tip:

```bash
git switch main
git merge <release-commit>
```

Use the requested branch instead of `main` when applicable. If the release is already contained, continue with version checks only if the user still wants the repository state refreshed.

## 4. Resolve conflicts with voskey precedence

- Inspect every conflicted file and prefer existing voskey behavior unless an upstream change is clearly required.
- Do not blanket-accept `--ours`; manually combine files with mixed intent.
- Adopt the upstream side only intentionally, and preserve unrelated local work.
- Before editing `packages/backend/` or `packages/frontend/`, load `working-on-backend` or `working-on-frontend`, respectively.
- Follow AGENTS.md for locale and migration safety; ask if a conflict cannot be resolved within those rules.
- Stage resolved files individually with `git add <path>` and recheck status until no conflicts remain.
- Before completing the merge, follow `shipping-misskey-change` for validation.

## 5. Update the root package version

Set `version` in the root [package.json](../../../package.json) to `<misskey-version>-voskey`, for example `2025.12.1-voskey`.

Only the root package needs this version change unless the user requests broader version alignment.

## 6. Container publishing, when requested

Local `docker build` is not part of the default update workflow. Explain GitHub Actions publishing on pushes to `main`; only implement workflow changes when requested.

Read [container-publishing.md](references/tasks/container-publishing.md) for triggers, Docker Hub credentials, image names, and tags.

## Final checks and handoff

- Follow [shipping-misskey-change](../shipping-misskey-change/SKILL.md) before committing, opening a PR, completing a merge, or handing work back.
- Verify `git status` contains only intended changes and the root version matches the upstream release plus `-voskey`.
- Report the upstream tag, merged commit, conflict files, validation results, and any remaining GitHub Actions or Docker Hub setup.
