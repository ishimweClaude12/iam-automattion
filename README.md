# IAM Automation (CloudFormation + Git Sync)

Automates creation of IAM users, groups, and permissions with AWS CloudFormation,
deployed through CloudFormation Git Sync.

## Repo contents

| File              | Purpose                                            |
| ----------------- | -------------------------------------------------- |
| `iam.yaml`        | CloudFormation template that defines all resources |
| `deployment.yaml` | Git sync deployment file (points at the template)  |
| `README.md`       | This file                                          |

## What the template creates

- A one-time console password, auto-generated and stored in AWS Secrets Manager
  (`iam-one-time-password`). Nothing is hardcoded.
- `s3-group` with permission to list S3 buckets.
- `ec2-group` with permission to list and create EC2 instances.
- Three IAM users with console access, forced to reset their password at first login:
  - `ec2-user1` in `ec2-group`
  - `ec2-user2` in `ec2-group`, with an explicit deny on creating EC2 instances
  - `s3-user` in `s3-group`

## Security best practices applied

- Temporary password stored in Secrets Manager, never written into the template
  or exposed as a stack output.
- Least-privilege group policies scoped to only the actions required.
- Explicit deny on `ec2:RunInstances` for `ec2-user2` to demonstrate that deny
  overrides allow.
- `PasswordResetRequired: true` so users set their own password on first sign-in.
- Deployed via Git sync with pull-request review before changes reach the account.

## Deploying with Git Sync

1. Push this repo to GitHub.
2. In the CloudFormation console: Create stack, With new resources (standard),
   Template is ready, Sync from a Git repository.
3. Link the GitHub repository via a CodeConnections connection, select the branch,
   and set the deployment file path to `deployment.yaml`.
4. Provide the two IAM roles (Git sync role and stack operations role). The
   operations role needs permission to create IAM and Secrets Manager resources.
5. Because the template creates named IAM resources, the stack requires the
   `CAPABILITY_NAMED_IAM` capability.
6. Merge the pull request that Git sync opens. CloudFormation deploys the stack
   and then watches the repo for future commits.

## Retrieving the temporary password

The password is in Secrets Manager under `iam-one-time-password`.
Console: Secrets Manager, select the secret, Retrieve secret value.
CLI:

```bash
aws secretsmanager get-secret-value \
  --secret-id iam-one-time-password \
  --query SecretString --output text
```
