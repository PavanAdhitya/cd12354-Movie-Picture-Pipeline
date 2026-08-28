Backend Continuous Deployment workflow
====================================

Required GitHub Secrets
- `AWS_ACCESS_KEY_ID` — AWS IAM access key with ECR push permissions
- `AWS_SECRET_ACCESS_KEY` — AWS IAM secret
- `AWS_REGION` — AWS region (example: `us-east-1`)
- `ECR_REPOSITORY` — Full ECR repo URI (example: `123456789012.dkr.ecr.us-east-1.amazonaws.com/backend`)
- `KUBE_CONFIG` or `KUBE_CONFIG_BASE64` — kubeconfig content (plain) or base64-encoded kubeconfig string
- `KUBE_NAMESPACE` (optional) — Kubernetes namespace to deploy into (defaults to `default`)

Example: create ECR repo and add secrets

1. Create an ECR repository (replace region and account id):

```bash
aws ecr create-repository --repository-name backend --region us-east-1
```

2. Get the repository URI and add it to GitHub secrets as `ECR_REPOSITORY`.

3. Configure GitHub secrets using the GitHub CLI (example):

```bash
# Set simple text secrets (run from your project directory)
gh secret set AWS_ACCESS_KEY_ID --body "$AWS_ACCESS_KEY_ID"
gh secret set AWS_SECRET_ACCESS_KEY --body "$AWS_SECRET_ACCESS_KEY"
gh secret set AWS_REGION --body "us-east-1"
gh secret set ECR_REPOSITORY --body "123456789012.dkr.ecr.us-east-1.amazonaws.com/backend"

# If you prefer base64-encoded kubeconfig:
base64 ~/.kube/config | gh secret set KUBE_CONFIG_BASE64 --body -

# Or set raw kubeconfig (be careful with newlines):
gh secret set KUBE_CONFIG --body "$(cat ~/.kube/config)"

# Optionally set namespace
gh secret set KUBE_NAMESPACE --body "default"
```

Notes
- The workflow uses `pipenv run lint` and `pipenv run test` per the backend `Pipfile`.
- The built Docker image is tagged `${{ github.sha }}` and pushed to `${{ secrets.ECR_REPOSITORY }}`.
- The workflow will fail if tests fail or if kubeconfig/secrets are missing.
