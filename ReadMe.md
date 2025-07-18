# GitHub Actions Workflow Templates

This repository provides a collection of reusable GitHub Actions workflows designed to automate CI/CD tasks for a variety of tech stacks and deployment scenarios.

## Workflows Included

| Workflow File                  | Description                                                                 |
|-------------------------------|-----------------------------------------------------------------------------|
| `deploy-to-vps.yml`           | Deploys application artifacts or containers to a self-hosted VPS.                       |
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
    secrets:
      ssh_key: \${{ secrets.VPS_SSH_KEY }}
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
