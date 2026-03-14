---
name: update-voskey
description: Update the voskey repository by checking the latest GitHub release from misskey-dev/misskey, merging that release commit into the local main branch, resolving conflicts with preference for local voskey changes, updating the root package.json version to the Misskey release version plus -voskey, and building a Docker image tagged as icalo35/voskey-app with the release-based version. Use when Codex is asked to sync voskey with upstream Misskey releases or prepare a release image after upstream updates.
---

# Update Voskey

## Overview

Follow this workflow to update this repository against the latest upstream Misskey release and produce a versioned Docker image.

Work from the repository root on the local `main` branch unless the user explicitly asks for a different branch.

## Workflow

### 1. Confirm repository state

Check these items before changing anything:

- Current branch is `main`.
- Remotes include `origin` for `voskey-dev/voskey` and `upstream` for `misskey-dev/misskey`.
- The working tree is clean, or any existing changes are understood and preserved.

If unrelated local changes would interfere with the merge, stop and ask the user how to proceed rather than overwriting them.

### 2. Identify the latest upstream release

Determine the latest release from `misskey-dev/misskey` before merging.

Preferred sources:

- GitHub Releases for `misskey-dev/misskey`
- `gh release list --repo misskey-dev/misskey --limit 1` if `gh` is available

After identifying the release tag, fetch upstream tags and resolve the release commit locally. A typical sequence is:

```bash
git fetch upstream --tags
git rev-list -n 1 <release-tag>
```

Use the release tag value later as the version base, for example `2025.12.1`.

### 3. Merge the upstream release commit into local main

Merge the release commit, not an arbitrary branch tip.

Typical sequence:

```bash
git checkout main
git fetch upstream --tags
git merge <release-commit>
```

If Git reports no changes because the release commit is already contained, continue with version and image checks only if the user still wants a rebuild.

### 4. Resolve conflicts with voskey precedence

When conflicts occur, inspect each file and prefer the existing voskey side unless the upstream change is clearly required.

Rules:

- Treat local `main` as the preferred source of truth.
- Do not blanket-accept `--ours` for every file without inspection.
- For files with mixed intent, manually combine upstream changes with voskey-specific behavior preserved.
- Re-run status checks until all conflicts are resolved before continuing.

Helpful commands:

```bash
git status
git checkout --ours -- <path>
git checkout --theirs -- <path>
git add <path>
```

Use `--theirs` only when the upstream version is intentionally adopted.

### 5. Update the root package version

Edit the root `package.json` and set `version` to:

```text
<misskey-version>-voskey
```

Example:

```text
2025.12.1-voskey
```

Only the root [`package.json`](/home/uboar/repo/voskey/package.json) needs this version change unless the user asks for broader version alignment.

### 6. Build the Docker image with the version tag

Build from the repository root using the same version string used in `package.json`.

```bash
docker build -t icalo35/voskey-app:<misskey-version>-voskey .
```

Example:

```bash
docker build -t icalo35/voskey-app:2025.12.1-voskey .
```

If the user asks for multi-platform or push steps, treat that as an extension to this workflow rather than part of the default process.

## Final Checks

Before finishing, verify:

- `git status` shows only intended changes.
- The root [`package.json`](/home/uboar/repo/voskey/package.json) version matches the upstream release plus `-voskey`.
- The Docker image tag matches that same version string.
- Report the upstream release tag, merged commit, conflict files if any, and the final Docker tag to the user.
