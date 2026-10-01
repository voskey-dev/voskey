# GitHub Actions container publishing

Publishing should run in GitHub Actions on pushes to `main`, not from the local workstation. Workflow edits and pushes require the user's request.

## Workflow configuration

Create or update [`.github/workflows/docker.yml`](../../../../../.github/workflows/docker.yml). Use this trigger:

```yaml
on:
  push:
    branches:
      - main
  workflow_dispatch:
```

- Set `env.REGISTRY_IMAGE` to the target Docker Hub repository, such as `icalo35/voskey-app`.
- Log in with `docker/login-action` using repository secrets such as `DOCKER_USERNAME` and `DOCKER_PASSWORD` (a Docker Hub access token can serve as the password).
- Use `docker/setup-buildx-action` and `docker/build-push-action` for multi-platform builds if needed.
- Generate tags from repository state. For pushes to `main`, publish `latest` and optionally `sha-<short_commit>`.
- To publish the version from [package.json](../../../../../package.json), read it before the build or add an explicit metadata step.
- Push images from Actions, not from the local workstation. Never put credentials in repository files.

## Setup sequence

1. Ask the user to configure Docker Hub credentials as GitHub Actions secrets.
2. Change the Docker workflow trigger from `release` to pushes on `main`.
3. Change the upstream image name to the intended voskey repository, for example `icalo35/voskey-app`.
4. Align tags with voskey expectations: `latest`, `<package.json version>`, and optionally `sha-<short_commit>`.
5. Once the user explicitly requests a push, push the commit and confirm the workflow publishes successfully. Otherwise, hand back the changes and explain this remaining step.

The existing [docker.yml](../../../../../.github/workflows/docker.yml) and [docker-develop.yml](../../../../../.github/workflows/docker-develop.yml) are useful references. Adapt repository filters, triggers, and image names for voskey rather than copying upstream defaults unchanged.
