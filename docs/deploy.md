# Deploying

Every merge to `main` that touches `infra/` or `lambdas/` deploys to the dev AWS account (eu-west-1). The owner approves each run in the GitHub environment `dev`. The workflow is `.github/workflows/deploy.yml`; agents do not edit it.

```
merge to main -> build: Gradle + cdk synth (no AWS access) -> cdk.out
              -> deploy waits -> owner approves -> OIDC (environment dev) -> role github-ai-playground-deploy
              -> CDK bootstrap roles -> CloudFormation, capped by cdk-deploy-boundary
```

- **Code runs only in the build job.** The app and its dependencies run there, without AWS access. The deploy job only runs the CDK CLI on the synthesized `cdk.out`.
- **Only the latest commit deploys.** If `main` moved on while the run waited, the run stops and the newer run deploys.
- **The run logs are public**, like the repository.

The AWS side lives in the `aws-foundation` repository (`accounts/dev/github-deploy.tf`). The same account runs the devbox, so everything CDK deploys, including the roles Lambda functions run as, is capped by the permissions boundary `cdk-deploy-boundary`.

## Rules for the CDK app (`infra/`)

### Names and the boundary

- **Stack names start with `AiPlayground`**, e.g. `AiPlayground-LinkShortener`. Only those stacks are deployed.
  - Set `stackName` explicitly: inside a `Stage` the name becomes `<Stage>-<Stack>`.
- **IAM names start with `AiPlayground`.** The boundary only lets CDK manage IAM roles and policies named `AiPlayground*`.
  - CloudFormation names generated roles and policies after the stack, so leave `roleName` and `managedPolicyName` unset, or start them with `AiPlayground`.
  - No `path` on roles or policies.
  - No `PhysicalName.GENERATE_IF_NEEDED`.
- **Roles you pass to a service must be `AiPlayground*`.** This covers Lambda, Scheduler, Step Functions and any other service role.
- **Every role carries the boundary.** Set this in `infra/cdk.json`, and add an assertion test that every `AWS::IAM::Role` in the template has a `PermissionsBoundary`:

  ```json
  "context": {
    "@aws-cdk/core:permissionsBoundary": { "name": "cdk-deploy-boundary" }
  }
  ```

### The CDK toolchain

- **No unpinned code.** The `app` command in `cdk.json` runs local binaries only, e.g. `npx --no-install ts-node --prefer-ts-exts bin/app.ts`.
- **`aws-cdk` (the CLI) is a devDependency** in `infra/package.json`.
- **No lookups in CI.** `fromLookup` and similar run once on a laptop, and `cdk.context.json` is committed.

### What the app may contain

- **Serverless only.** No:
  - VPC, EC2 or a Lambda function in a VPC;
  - Route 53 hosted zones;
  - SSM documents or associations;
  - S3 access points;
  - account-wide settings (e.g. `logs:PutAccountPolicy`).
- **Parameters** (if any) live under `/AiPlayground/`.
- **No Docker assets.** Lambda code is a zip built by Gradle (`lambdas/`).
- **No secrets in stack outputs**, because the deploy log is public.

### What the app must not touch

- The tag `Project=devbox` or `Project=aws-foundation`.
- Buckets starting with `devbox-` or `tfstate-`.
- The `/devbox` parameter path.
- Schedules or alarms named `devbox-*`.

### Account IDs

Stacks are environment-agnostic, or use `process.env.CDK_DEFAULT_ACCOUNT`. Never write an account ID in code.

### What the boundary enforces

A deploy that breaks an IAM, network, devbox or account-settings rule fails with `AccessDenied`. These rules are only enforced by review:
- stack names, for stacks without IAM resources;
- no account IDs in code;
- no secrets in stack outputs.

## Locally

`npx cdk synth` needs no AWS credentials. Do not run `cdk deploy` from the devbox: it has no AWS credentials, by design.
