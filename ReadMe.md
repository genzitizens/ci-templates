# GitHub Actions Workflow Templates

This repository provides a collection of reusable GitHub Actions workflows designed to automate CI/CD tasks for a variety of tech stacks and deployment scenarios.

## Workflows Included

| Workflow File                  | Description                                                                 |
|-------------------------------|-----------------------------------------------------------------------------|
| `deploy-to-vps.yml`           | Deploys application artifacts or containers to a self-hosted VPS.                       |
| `sync-compose-to-vps.yml`     | Uploads docker-compose assets to a VPS before running a deployment.        |
| `java-ci.yml`                 | Java CI workflow: builds, tests, and verifies Java applications using Maven or Gradle.  |
| `node-ci.yml`                 | Node.js CI workflow: installs dependencies, runs tests, and builds the app. |
| `push-to-ghcr.yml`            | Pushes container images to GitHub Container Registry (GHCR).               |
| `react-ci.yml`                | Builds and tests React applications with optional deployment step.         |
| `sonarcloud.yml`              | Integrates SonarCloud for static code analysis and quality gate checks.    |
| `springboot-push-to-ghcr.yml` | Builds Spring Boot app and pushes the OCI image to GHCR.                   |
| `java-scan.yml`             | Scan Source code for vulnerabilities.                   |

## Usage

Each workflow is designed to be **reusable** via `workflow_call`.
<br>To use one in your project, reference it in your workflow like so:

### Example: Java CI
```yaml
name: Java CI

on:
  push:
    branches: [ main ]

jobs:
  call-java-ci:
    uses: your-org/your-repo/.github/workflows/java-ci.yml@main
    with:
      java-version: '21'
```

### Example: Node CI
```yaml
name: Node CI

on:
  push:
    branches: [ main ]

jobs:
  call-node-ci:
    uses: your-org/your-repo/.github/workflows/node-ci.yml@main
    with:
      node-version: '20'
```

### Example: Sync Compose Assets to a VPS

**Required secrets**
- `SSH_PRIVATE_KEY`
- `VPS_HOST`
- `VPS_USER`

**Inputs**
- `compose-path`: Path (file or directory) in your repository that should be uploaded.
- `remote-path`: Destination path on the VPS for the `docker-compose.yml` file.

**Outputs**
- `remote-compose-path`: The compose file path on the VPS (useful for deployment jobs).
- `remote-directory`: The directory containing the synced files.

```yaml
name: Sync Compose Assets

on:
  push:
    branches: [ main ]

jobs:
  sync-compose:
    uses: your-org/your-repo/.github/workflows/sync-compose-to-vps.yml@main
    with:
      compose-path: infra/docker
      remote-path: /opt/my-app/docker-compose.yml
    secrets:
      SSH_PRIVATE_KEY: ${{ secrets.VPS_SSH_KEY }}
      VPS_HOST: ${{ secrets.VPS_HOST }}
      VPS_USER: ${{ secrets.VPS_USER }}
```

### Example: Sync & Deploy to a VPS

```yaml
name: Sync and Deploy

on:
  workflow_dispatch:

jobs:
  sync-compose:
    uses: your-org/your-repo/.github/workflows/sync-compose-to-vps.yml@main
    with:
      compose-path: infra/docker
      remote-path: /opt/my-app/docker-compose.yml
    secrets:
      SSH_PRIVATE_KEY: ${{ secrets.VPS_SSH_KEY }}
      VPS_HOST: ${{ secrets.VPS_HOST }}
      VPS_USER: ${{ secrets.VPS_USER }}

  deploy:
    needs: sync-compose
    uses: your-org/your-repo/.github/workflows/deploy-to-vps.yml@main
    with:
      IMAGE_NAME: my-app
      COMPOSE_PATH: ${{ needs.sync-compose.outputs.remote-compose-path }}
      RUN_COMPOSE_PULL: true # optional but keeps services current
    secrets:
      SSH_PRIVATE_KEY: ${{ secrets.VPS_SSH_KEY }}
      VPS_HOST: ${{ secrets.VPS_HOST }}
      VPS_USER: ${{ secrets.VPS_USER }}
      GHCR_TOKEN: ${{ secrets.GHCR_TOKEN }}
      GHCR_USERNAME: ${{ secrets.GHCR_USERNAME }}
      GHCR_ORG: ${{ secrets.GHCR_ORG }}
      DISCORD_WEBHOOK: ${{ secrets.DISCORD_WEBHOOK }}
```

### Example: Deploy to VPS
```yaml
name: Deploy to VPS

on:
  workflow_run:
    workflows: ["Build and Push"]
    types:
      - completed

jobs:
  deploy:
    uses: your-org/your-repo/.github/workflows/deploy-to-vps.yml@main
    with:
      IMAGE_NAME: my-app
      COMPOSE_PATH: /opt/my-app/docker-compose.yml
      RUN_COMPOSE_PULL: true # optional, defaults to false
    secrets:
      SSH_PRIVATE_KEY: \${{ secrets.VPS_SSH_KEY }}
      VPS_HOST: \${{ secrets.VPS_HOST }}
      VPS_USER: \${{ secrets.VPS_USER }}
      GHCR_TOKEN: \${{ secrets.GHCR_TOKEN }}
      GHCR_USERNAME: \${{ secrets.GHCR_USERNAME }}
      GHCR_ORG: \${{ secrets.GHCR_ORG }}
      DISCORD_WEBHOOK: \${{ secrets.DISCORD_WEBHOOK }}
```

### Example: Push to GHCR
```yaml
name: Push Docker Image to GHCR

on:
  push:
    branches: [ main ]

jobs:
  push-image:
    uses: your-org/your-repo/.github/workflows/push-to-ghcr.yml@main
    with:
      image-name: my-app
    secrets:
      ghcr_token: \${{ secrets.GHCR_TOKEN }}
```

### Example: React CI
```yaml
name: React CI

on:
  push:
    branches: [ main ]

jobs:
  react-build:
    uses: your-org/your-repo/.github/workflows/react-ci.yml@main
```

### Example: SonarCloud
```yaml
name: SonarCloud Analysis

on:
  push:
    branches: [ main ]

jobs:
  sonar:
    uses: your-org/your-repo/.github/workflows/sonarcloud.yml@main
    secrets:
      sonar_token: \${{ secrets.SONAR_TOKEN }}
```

### Example: Spring Boot → GHCR
```yaml
name: Spring Boot Build & Push

on:
  push:
    branches: [ main ]

jobs:
  spring-ghcr:
    uses: your-org/your-repo/.github/workflows/springboot-push-to-ghcr.yml@main
    with:
      image-name: spring-app
    secrets:
      ghcr_token: \${{ secrets.GHCR_TOKEN }}
```

## Benefits

- Modular and maintainable
- Designed for reuse across multiple projects
- Built-in support for container builds, SonarCloud scanning, and deployments

## Requirements

- GitHub Actions enabled
- Optional: Secrets like `GHCR_TOKEN`, `SONAR_TOKEN`, or `VPS_SSH_KEY` depending on workflow
