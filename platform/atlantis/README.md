# Atlantis

![Version: 5.25.0](https://img.shields.io/badge/Version-5.25.0-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: v0.39.0](https://img.shields.io/badge/AppVersion-v0.39.0-informational?style=flat-square)

Terraform Pull Request Automation for GitLab - https://www.runatlantis.io

## Overview

Atlantis is a self-hosted application that listens for GitLab webhooks and runs `terraform plan` and `terraform apply` commands on your Terraform repositories. This enables GitOps workflows for infrastructure changes.

## How It Works

```
1. Developer opens MR with Terraform changes
2. GitLab sends webhook to Atlantis
3. Atlantis runs `terraform plan` automatically
4. Plan output posted as MR comment
5. Reviewer comments `atlantis apply` to approve
6. Atlantis runs `terraform apply` and updates MR
```

## Configuration

| Key | Description | Default |
|-----|-------------|---------|
| `atlantis.gitlab.user` | GitLab username for Atlantis | `atlantis` |
| `atlantis.gitlab.hostname` | GitLab hostname | `gitlab.ops.techsecom.io` |
| `atlantis.ingress.host` | Ingress hostname | `atlantis.ops.techsecom.io` |
| `atlantis.orgAllowlist` | Allowed repositories | `gitlab.ops.techsecom.io/*` |
| `atlantis.defaultTFVersion` | Default Terraform version | `1.10.0` |
| `eso.vaultPath` | Vault path for secrets | `/atlantis/` |

## Vault Secrets Required

Store the following secrets in Vault at `/atlantis/`:

| Key | Description |
|-----|-------------|
| `gitlab_token` | GitLab Personal Access Token with `api` scope |
| `gitlab_webhook_secret` | Random string for webhook validation |
| `aws_access_key_id` | MinIO/S3 access key for state backend |
| `aws_secret_access_key` | MinIO/S3 secret key for state backend |
| `vault_token` | Vault token for Terraform providers (kubeconfig access) |

Example Vault commands:
```bash
vault kv put secret/atlantis \
  gitlab_token="glpat-xxxxxxxxxxxx" \
  gitlab_webhook_secret="$(openssl rand -hex 32)" \
  aws_access_key_id="minio-access-key" \
  aws_secret_access_key="minio-secret-key" \
  vault_token="hvs.your-vault-token"
```

## GitLab Setup

1. **Create GitLab User**: Create a user named `atlantis` (or use existing)
2. **Generate Token**: Create a Personal Access Token with `api` scope
3. **Configure Webhook**: In each repository, add webhook:
   - URL: `https://atlantis.ops.techsecom.io/events`
   - Secret Token: Same as `gitlab_webhook_secret` in Vault
   - Triggers: Push events, Merge request events, Comments

## Features

- **Parallel Plans**: Multiple projects can plan simultaneously
- **Automerge**: Automatically merge MR after successful apply
- **Custom Workflows**: Repositories can override default workflow
- **Apply Requirements**: Requires approval and mergeable status

## Source Code

- <https://github.com/runatlantis/atlantis>
- <https://www.runatlantis.io/docs/>
