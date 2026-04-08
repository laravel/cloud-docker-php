# docker-php

This is a fork of [serversideup/docker-php](https://github.com/serversideup/docker-php).

## Purpose

We use this fork to rebuild the `fpm-nginx-bookworm` PHP base images and push them to a private ECR repository. Rebuilding from the same upstream tags picks up the latest PHP patch versions from the official `php:` Docker images, fixing CVEs without changing the serversideup version.

These ECR images are consumed by [cloud-customer-base-images](https://github.com/laravel/cloud-customer-base-images) as the base layer for Laravel Cloud's production Docker images.

## Tags

Tags are synced from upstream. To re-sync:

```bash
git fetch upstream --tags && git push origin --tags
```

## Current image matrix

| PHP Version | Git Tag | ECR Image Tag |
|-------------|---------|---------------|
| 8.2 | `v3.6.1` | `8.2-fpm-nginx-bookworm-v3.6.1` |
| 8.3 | `v3.6.1` | `8.3-fpm-nginx-bookworm-v3.6.1` |
| 8.4 | `v3.6.1` | `8.4-fpm-nginx-bookworm-v3.6.1` |
| 8.5 | `v4.1.0` | `8.5-fpm-nginx-bookworm-v4.1.0` |

## Running the workflow

The `Build ECR Images` workflow is triggered manually via `workflow_dispatch`. It checks out the repo at the specified git tag, builds the `fpm-nginx-bookworm` Dockerfile for `linux/arm64`, and pushes to ECR.

### Trigger all current builds

```bash
# PHP 8.2 + 8.3 at v3.6.1
gh workflow run build-ecr-images.yml --repo laravel/docker-php --ref main -f tag=v3.6.1 -f "php_versions=8.2,8.3"

# PHP 8.4 at v3.6.1
gh workflow run build-ecr-images.yml --repo laravel/docker-php --ref main -f tag=v3.6.1 -f "php_versions=8.4"

# PHP 8.5 at v4.1.0
gh workflow run build-ecr-images.yml --repo laravel/docker-php --ref main -f tag=v4.1.0 -f "php_versions=8.5"
```

### Workflow inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `tag` | Yes | - | Git tag to checkout (e.g. `v3.6.1`) |
| `php_versions` | Yes | `8.2,8.3,8.4,8.5` | Comma-separated PHP minor versions to build |
| `dry_run` | No | `false` | Build without pushing to ECR |

### Required secrets

| Secret | Description |
|--------|-------------|
| `GHA_ROLE` | AWS IAM role ARN for OIDC federation |
| `IMG_REPO` | ECR repository URL |
