# deploy-ecs-service

Renders an ECS task definition with a new image, then either deploys it to a
service and waits for it to settle, or only registers it, for one-off tasks
such as migrations that a script then runs.

## Usage

Deploy a service:

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      id-token: write
    steps:
      - uses: actions/checkout@v7
        with:
          persist-credentials: false

      - uses: uptick-dev/deploy-ecs-service@v1
        with:
          task-definition: .aws/platform-staging.json
          container-name: platform-staging
          service: platform-staging
          cluster: staging-us-east-1
          image: ghcr.io/owner/app:${{ needs.build.outputs.tag }}
          role-to-assume: ${{ vars.AWS_DEPLOY_ROLE_ARN }}
```

Register a task definition without deploying it, with Datadog version tags:

```yaml
      - uses: uptick-dev/deploy-ecs-service@v1
        with:
          task-definition: .aws/event-runner-us-east-1.json
          container-name: dbt-event-runner
          image: ghcr.io/owner/app:sha-${{ steps.vars.outputs.short_sha }}
          environment-variables: |
            DD_VERSION=sha-${{ steps.vars.outputs.short_sha }}
          docker-labels: |
            com.datadoghq.tags.version=sha-${{ steps.vars.outputs.short_sha }}
```

## Inputs

| Name | Default | Description |
| --- | --- | --- |
| `task-definition` | _(required)_ | Task definition JSON file. |
| `container-name` | _(required)_ | Container that gets the image. |
| `image` | _(required)_ | Full image URI with tag. |
| `cluster` | `""` | ECS cluster. Required with `service`. |
| `service` | `""` | Service to deploy to. Empty only registers the task definition. |
| `environment-variables` | `""` | Newline-separated `KEY=value` pairs set on the container. |
| `docker-labels` | `""` | Newline-separated `key=value` Docker labels. |
| `wait-for-service-stability` | `"true"` | Wait for the service to settle after deploying. |
| `role-to-assume` | `""` | IAM role to assume over OIDC (needs `id-token: write`). Empty uses credentials already configured in the job. |
| `aws-region` | `us-east-1` | Region used with `role-to-assume`. |

## Outputs

| Name | Description |
| --- | --- |
| `task-definition` | Path to the rendered task definition. |
| `task-definition-arn` | ARN of the registered revision. |
