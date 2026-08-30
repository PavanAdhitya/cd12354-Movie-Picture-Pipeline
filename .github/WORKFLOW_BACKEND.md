Backend Continuous Deployment workflow
====================================

## Workflow name
`Backend Continuous Deployment`

## Triggers
- **Automatic**: Runs on every `push` to `main` when files under `starter/backend/` change.
- **Manual**: Supports `workflow_dispatch` (can be triggered manually from the Actions tab).

## Pipeline stages

| Job        | Runs on        | Purpose                                                   |
|------------|----------------|-----------------------------------------------------------|
| **lint**   | ubuntu-latest  | Runs `pipenv run lint` (flake8) against backend code      |
| **test**   | ubuntu-latest  | Runs `pipenv run test` (pytest -v) against backend code   |
| **build**  | ubuntu-latest  | Builds Docker image, pushes to ECR, deploys to Kubernetes |

The `build` job only runs after **both** `lint` and `test` pass.

## Required GitHub Secrets

| Secret                 | Description                                                                          | Example                                                          |
|------------------------|--------------------------------------------------------------------------------------|------------------------------------------------------------------|
| `AWS_ACCESS_KEY_ID`    | AWS IAM access key with ECR push and EKS access permissions                          | `AKIAIOSFODNN7EXAMPLE`                                           |
| `AWS_SECRET_ACCESS_KEY`| AWS IAM secret key                                                                   | `wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY`                     |
| `AWS_REGION`           | AWS region where the ECR repository and EKS cluster live                             | `us-east-1`                                                      |
| `ECR_REPOSITORY`       | Full ECR repository URI (without tag)                                                | `123456789012.dkr.ecr.us-east-1.amazonaws.com/backend`           |

## Setting secrets via GitHub CLI

```bash
gh secret set AWS_ACCESS_KEY_ID --body "$AWS_ACCESS_KEY_ID"
gh secret set AWS_SECRET_ACCESS_KEY --body "$AWS_SECRET_ACCESS_KEY"
gh secret set AWS_REGION --body "us-east-1"
gh secret set ECR_REPOSITORY --body "123456789012.dkr.ecr.us-east-1.amazonaws.com/backend"
```

## Notes
- The workflow uses `pipenv run lint` (flake8) and `pipenv run test` (pytest) as defined in the backend `Pipfile`.
- The Docker image is tagged with `${{ github.sha }}` for deterministic, traceable deployments.
- Kubeconfig is obtained via `aws eks update-kubeconfig --name cluster` (the EKS cluster is named `cluster` per the Terraform configuration).
- Kubernetes manifests are applied via `kustomize build`, then `kubectl set image` updates the container to the newly pushed ECR image.
- The workflow waits for the rollout to complete (`kubectl rollout status`) and **fails** if the deployment does not become ready within 180 seconds.
- No credentials are hardcoded — all come from GitHub Secrets.

## Successful Pipeline Run Proof
![Backend Continuous Deployment Success](../screenshots/backend_continuous_deployment_success.png)
