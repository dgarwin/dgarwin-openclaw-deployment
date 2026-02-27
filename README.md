# OpenClaw Deployment

This repository contains the AWS CloudFormation template for deploying OpenClaw infrastructure.

## ⚠️ Security Notice

**This repository should NEVER be accessible to the OpenClaw agent.** The CloudFormation template defines IAM roles and permissions. If OpenClaw could modify this template, it could escalate its own privileges.

## Repository Structure

```
dgarwin-openclaw-deployment/
└── clawdbot-bedrock-agentcore.yaml  # CloudFormation template
```

## How It Works

This is part of a three-repository setup:

1. **dgarwin-openclaw-deployment** (this repo)
   - Contains CloudFormation template
   - Defines infrastructure and IAM roles
   - **Access**: Admins only, NOT accessible to OpenClaw

2. **dgarwin-openclaw-scripts**
   - Contains operational scripts (userdata.sh, sync-memory.sh)
   - Cloned by CloudFormation template during EC2 bootstrap
   - **Access**: Read-only by OpenClaw

3. **dgarwin-openclaw**
   - Contains workspace configuration (.md files, openclaw.json)
   - Holds private/personal information
   - Cloned by userdata.sh during setup
   - **Access**: Read-write by OpenClaw (for memory sync)

## Deployment Flow

1. CloudFormation template provisions infrastructure
2. EC2 UserData clones `dgarwin-openclaw-scripts`
3. Runs `userdata.sh` from scripts repo
4. `userdata.sh` clones `dgarwin-openclaw` for configuration
5. OpenClaw starts with configuration from config repo

## Deployment

```bash
aws cloudformation create-stack \
  --stack-name openclaw \
  --template-body file://clawdbot-bedrock-agentcore.yaml \
  --capabilities CAPABILITY_IAM \
  --region us-east-2
```

## Security Best Practices

- ✅ Keep this repo private
- ✅ Restrict GitHub access to admins only
- ✅ Review IAM role changes carefully
- ✅ Never grant OpenClaw write access to this repo
- ✅ Use GitHub branch protection for main branch

### Known Security Limitation

⚠️ **OpenClaw can currently modify its own `openclaw.json` configuration** since it has write access to the `dgarwin-openclaw` repository. This means the agent can change its own permissions settings (e.g., `tools.exec.security`).

**Current defense layers**:
1. IAM permissions (defined in this template) limit what the EC2 instance can do in AWS
2. GitHub repo permissions prevent access to this deployment repo
3. Agent's built-in safety design discourages self-modification

**TODO**: Add OS-level restrictions via CloudFormation:
- Implement systemd service hardening in the UserData
- Use AppArmor/SELinux profiles
- Mount configuration as read-only
- Container-based isolation

See `dgarwin-openclaw-scripts/README.md` for detailed TODO list.
