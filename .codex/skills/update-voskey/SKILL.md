---
name: update-voskey
description: Update the voskey repository by checking the latest GitHub release from misskey-dev/misskey, merging that release commit into the local main branch, resolving conflicts with preference for local voskey changes, updating the root package.json version to the Misskey release version plus -voskey, and documenting how container publishing should run from GitHub Actions on pushes to main. Use when Codex is asked to sync voskey with upstream Misskey releases or prepare the repository for release automation after upstream updates.
---

# Update Voskey

## Overview

Follow this workflow to update this repository against the latest upstream Misskey release and keep the repository aligned with GitHub Actions based container publishing.

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

If Git reports no changes because the release commit is already contained, continue with version checks only if the user still wants the repository state refreshed.

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

### 6. Document GitHub Actions based container publishing

Do not treat local `docker build` as part of the default update workflow.

When the user wants container publishing, explain that it should be handled by a GitHub Actions workflow triggered by pushes to `main`, then published to Docker Hub. Keep the explanation concrete:

- Create or update a workflow under [`.github/workflows/docker.yml`](/home/uboar/repo/voskey/.github/workflows/docker.yml).
- Change the trigger to:

```bash
on:
  push:
    branches:
      - main
  workflow_dispatch:
```

- Set `env.REGISTRY_IMAGE` to the target Docker Hub repository such as `icalo35/voskey-app`.
- Log in with `docker/login-action` using repository secrets, typically `DOCKER_USERNAME` and `DOCKER_PASSWORD` or a Docker Hub access token.
- Use `docker/setup-buildx-action` and `docker/build-push-action` for multi-platform builds if needed.
- Generate tags from the repository state. The simplest stable option for pushes to `main` is to publish `latest` and optionally a commit SHA tag. If the repository version in [`package.json`](/home/uboar/repo/voskey/package.json) should also be published as a tag, add an explicit metadata step or read the version before build.
- Push from Actions, not from the local workstation.

If the user asks for implementation details, direct them to this sequence:

1. Add Docker Hub credentials as GitHub Actions secrets.
2. Update the Docker workflow trigger from `release` to `push` on `main`.
3. Change the image name from the upstream default to `icalo35/voskey-app`.
4. Ensure the tagging strategy matches voskey expectations, for example `latest`, `<package.json version>`, and `sha-<short_commit>`.
5. Push a commit to `main` and confirm the workflow publishes successfully.

Existing upstream-style workflows such as [`.github/workflows/docker.yml`](/home/uboar/repo/voskey/.github/workflows/docker.yml) and [`.github/workflows/docker-develop.yml`](/home/uboar/repo/voskey/.github/workflows/docker-develop.yml) are useful references, but adapt their repository filters, triggers, and image names for voskey.

## Final Checks

Before finishing, verify:

- `git status` shows only intended changes.
- The root [`package.json`](/home/uboar/repo/voskey/package.json) version matches the upstream release plus `-voskey`.
- Report the upstream release tag, merged commit, conflict files if any, and any GitHub Actions or Docker Hub setup the user still needs to complete.
