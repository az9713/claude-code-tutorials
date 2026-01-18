# Tutorial 16: Claude Code GitLab CI/CD Integration

> **Learn how to integrate Claude Code into your GitLab development workflow**

## Overview

Claude Code integrates with GitLab CI/CD to automate code changes, create merge requests from issues, and respond to comments with `@claude` mentions. This tutorial covers setup, configuration, and best practices for enterprise use.

### What You'll Learn

- Setting up Claude Code in GitLab CI/CD pipelines
- Creating merge requests from issues with Claude
- Configuring AWS Bedrock and Google Vertex AI providers
- Implementing OIDC-based authentication for enterprise security
- Best practices for security, performance, and cost optimization

### Prerequisites

- GitLab account with repository access
- Claude API key (or Bedrock/Vertex AI access)
- Basic knowledge of GitLab CI/CD (`.gitlab-ci.yml`)

---

## Part 1: Why Use Claude Code with GitLab?

### Key Benefits

| Benefit | Description |
|---------|-------------|
| **Instant MR creation** | Describe what you need, Claude proposes complete MR |
| **Automated implementation** | Turn issues into working code with a single mention |
| **Project-aware** | Claude follows your `CLAUDE.md` guidelines |
| **Simple setup** | One job in `.gitlab-ci.yml` + API variable |
| **Enterprise-ready** | Choose Claude API, AWS Bedrock, or Google Vertex AI |
| **Secure by default** | Runs in your runners with branch protection |

### How It Works

```
┌─────────────────────────────────────────────────────────────────────┐
│ 1. Event-Driven Orchestration                                        │
│    GitLab listens for triggers (e.g., @claude in comments)          │
├─────────────────────────────────────────────────────────────────────┤
│ 2. Provider Abstraction                                              │
│    Use Claude API (SaaS), AWS Bedrock, or Google Vertex AI          │
├─────────────────────────────────────────────────────────────────────┤
│ 3. Sandboxed Execution                                               │
│    Each interaction runs in isolated container                       │
├─────────────────────────────────────────────────────────────────────┤
│ 4. Code Review Flow                                                  │
│    All changes flow through MRs with approvals                       │
└─────────────────────────────────────────────────────────────────────┘
```

### What Claude Can Do

- Create and update MRs from issue descriptions or comments
- Analyze performance regressions and propose optimizations
- Implement features directly in a branch
- Fix bugs and regressions identified by tests
- Respond to follow-up comments for iterations

---

## Part 2: Quick Setup

### Step 1: Add API Key as CI/CD Variable

1. Go to **Settings** → **CI/CD** → **Variables**
2. Add `ANTHROPIC_API_KEY` (masked, protected as needed)

### Step 2: Add Claude Job to `.gitlab-ci.yml`

```yaml
stages:
  - ai

claude:
  stage: ai
  image: node:24-alpine3.21
  rules:
    - if: '$CI_PIPELINE_SOURCE == "web"'
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
  variables:
    GIT_STRATEGY: fetch
  before_script:
    - apk update
    - apk add --no-cache git curl bash
    - curl -fsSL https://claude.ai/install.sh | bash
  script:
    # Optional: start GitLab MCP server if available
    - /bin/gitlab-mcp-server || true
    # Use AI_FLOW_* variables for context
    - echo "$AI_FLOW_INPUT for $AI_FLOW_CONTEXT on $AI_FLOW_EVENT"
    - >
      claude
      -p "${AI_FLOW_INPUT:-'Review this MR and implement the requested changes'}"
      --permission-mode acceptEdits
      --allowedTools "Bash(*) Read(*) Edit(*) Write(*) mcp__gitlab"
      --debug
```

### Step 3: Test Your Setup

Run the job manually from **CI/CD** → **Pipelines**, or trigger from an MR.

---

## Part 3: Manual Setup (Production Recommended)

For enterprise environments with more control:

### 1. Configure Provider Access

| Provider | Configuration |
|----------|---------------|
| **Claude API** | Store `ANTHROPIC_API_KEY` as masked CI/CD variable |
| **AWS Bedrock** | Configure GitLab → AWS OIDC, create IAM role for Bedrock |
| **Google Vertex AI** | Configure Workload Identity Federation for GitLab → GCP |

### 2. Add Project Credentials

For GitLab API operations:
- Use `CI_JOB_TOKEN` by default, or
- Create a Project Access Token with `api` scope
- Store as `GITLAB_ACCESS_TOKEN` (masked) if using PAT

### 3. Enable Mention-Driven Triggers (Optional)

1. Add project webhook for "Comments (notes)" events
2. Have listener call pipeline trigger API with variables when comment contains `@claude`

---

## Part 4: AWS Bedrock Integration (OIDC)

### Prerequisites

- AWS account with Amazon Bedrock access to Claude models
- GitLab configured as OIDC identity provider in AWS IAM
- IAM role with Bedrock permissions and trust policy

### Required CI/CD Variables

| Variable | Description |
|----------|-------------|
| `AWS_ROLE_TO_ASSUME` | ARN of the IAM role for Bedrock access |
| `AWS_REGION` | Bedrock region (e.g., `us-west-2`) |

### Configuration Example

```yaml
claude-bedrock:
  stage: ai
  image: node:24-alpine3.21
  rules:
    - if: '$CI_PIPELINE_SOURCE == "web"'
  before_script:
    - apk add --no-cache bash curl jq git python3 py3-pip
    - pip install --no-cache-dir awscli
    - curl -fsSL https://claude.ai/install.sh | bash
    # Exchange GitLab OIDC token for AWS credentials
    - export AWS_WEB_IDENTITY_TOKEN_FILE="${CI_JOB_JWT_FILE:-/tmp/oidc_token}"
    - if [ -n "${CI_JOB_JWT_V2}" ]; then printf "%s" "$CI_JOB_JWT_V2" > "$AWS_WEB_IDENTITY_TOKEN_FILE"; fi
    - >
      aws sts assume-role-with-web-identity
      --role-arn "$AWS_ROLE_TO_ASSUME"
      --role-session-name "gitlab-claude-$(date +%s)"
      --web-identity-token "file://$AWS_WEB_IDENTITY_TOKEN_FILE"
      --duration-seconds 3600 > /tmp/aws_creds.json
    - export AWS_ACCESS_KEY_ID="$(jq -r .Credentials.AccessKeyId /tmp/aws_creds.json)"
    - export AWS_SECRET_ACCESS_KEY="$(jq -r .Credentials.SecretAccessKey /tmp/aws_creds.json)"
    - export AWS_SESSION_TOKEN="$(jq -r .Credentials.SessionToken /tmp/aws_creds.json)"
  script:
    - /bin/gitlab-mcp-server || true
    - >
      claude
      -p "${AI_FLOW_INPUT:-'Implement the requested changes and open an MR'}"
      --permission-mode acceptEdits
      --allowedTools "Bash(*) Read(*) Edit(*) Write(*) mcp__gitlab"
      --debug
  variables:
    AWS_REGION: "us-west-2"
```

> **Note:** Model IDs for Bedrock include region-specific prefixes and version suffixes (e.g., `us.anthropic.claude-sonnet-4-5-20250929-v1:0`).

---

## Part 5: Google Vertex AI Integration (Workload Identity Federation)

### Prerequisites

- Google Cloud project with Vertex AI API enabled
- Workload Identity Federation configured to trust GitLab OIDC
- Dedicated service account with Vertex AI roles

### Required CI/CD Variables

| Variable | Description |
|----------|-------------|
| `GCP_WORKLOAD_IDENTITY_PROVIDER` | Full provider resource name |
| `GCP_SERVICE_ACCOUNT` | Service account email |
| `CLOUD_ML_REGION` | Vertex region (e.g., `us-east5`) |

### Configuration Example

```yaml
claude-vertex:
  stage: ai
  image: gcr.io/google.com/cloudsdktool/google-cloud-cli:slim
  rules:
    - if: '$CI_PIPELINE_SOURCE == "web"'
  before_script:
    - apt-get update && apt-get install -y git && apt-get clean
    - curl -fsSL https://claude.ai/install.sh | bash
    # Authenticate to Google Cloud via WIF (no downloaded keys)
    - >
      gcloud auth login --cred-file=<(cat <<EOF
      {
        "type": "external_account",
        "audience": "${GCP_WORKLOAD_IDENTITY_PROVIDER}",
        "subject_token_type": "urn:ietf:params:oauth:token-type:jwt",
        "service_account_impersonation_url": "https://iamcredentials.googleapis.com/v1/projects/-/serviceAccounts/${GCP_SERVICE_ACCOUNT}:generateAccessToken",
        "token_url": "https://sts.googleapis.com/v1/token"
      }
      EOF
      )
    - gcloud config set project "$(gcloud projects list --format='value(projectId)' --filter="name:${CI_PROJECT_NAMESPACE}" | head -n1)" || true
  script:
    - /bin/gitlab-mcp-server || true
    - >
      CLOUD_ML_REGION="${CLOUD_ML_REGION:-us-east5}"
      claude
      -p "${AI_FLOW_INPUT:-'Review and update code as requested'}"
      --permission-mode acceptEdits
      --allowedTools "Bash(*) Read(*) Edit(*) Write(*) mcp__gitlab"
      --debug
  variables:
    CLOUD_ML_REGION: "us-east5"
```

> **Note:** With Workload Identity Federation, no service account keys need to be stored.

---

## Part 6: Example Use Cases

### Turn Issues into MRs

In an issue comment:
```
@claude implement this feature based on the issue description
```

Claude analyzes the issue and codebase, writes changes, opens an MR.

### Get Implementation Help

In an MR discussion:
```
@claude suggest a concrete approach to cache the results of this API call
```

Claude proposes changes with appropriate caching and updates the MR.

### Fix Bugs Quickly

In an issue or MR comment:
```
@claude fix the TypeError in the user dashboard component
```

Claude locates the bug, implements a fix, and updates the branch.

### Code Review

```
@claude review this MR for security vulnerabilities and performance issues
```

### Refactoring

```
@claude refactor the UserService class to use dependency injection and add unit tests
```

### Documentation

```
@claude generate API documentation for all endpoints in src/api/controllers/
```

---

## Part 7: Best Practices

### CLAUDE.md Configuration

Create a `CLAUDE.md` file at repository root:

```markdown
# Project Guidelines

## Coding Standards
- Use TypeScript strict mode
- Follow ESLint configuration
- Write tests for all new functions

## Security Requirements
- Never log sensitive data
- Use parameterized queries
- Validate all user input

## Testing
- Run `npm test` before committing
- Maintain >80% code coverage
```

### Security Considerations

| Practice | Description |
|----------|-------------|
| **Never commit API keys** | Always use CI/CD variables |
| **Use OIDC where possible** | No long-lived keys needed |
| **Limit job permissions** | Restrict network egress |
| **Review Claude's MRs** | Treat like any other contributor |
| **Protect variables** | Mark sensitive variables as protected |

### Optimizing Performance

1. **Keep CLAUDE.md focused** - Concise guidelines process faster
2. **Provide clear descriptions** - Reduce back-and-forth iterations
3. **Configure timeouts** - Avoid runaway runs with sensible limits
4. **Cache packages** - Speed up runner execution

### Cost Management

| Cost Type | Optimization |
|-----------|--------------|
| **GitLab Runner time** | Use efficient images, cache dependencies |
| **API tokens** | Use specific commands, limit turns |
| **Parallel runs** | Set concurrency limits |

```yaml
# Limit iterations
claude -p "..." --max-turns 10

# Set timeout
timeout: 30 minutes
```

---

## Part 8: Security and Governance

### Isolation Guarantees

- Each job runs in isolated container with restricted network access
- Claude's changes flow through MRs so reviewers see every diff
- Branch protection and approval rules apply to AI-generated code
- Claude Code uses workspace-scoped permissions

### Recommended Security Setup

```yaml
# Use job-specific tokens with minimal permissions
claude-job:
  variables:
    GIT_STRATEGY: clone  # Fresh clone each time
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
      when: manual  # Require manual approval
```

---

## Part 9: Troubleshooting

### Claude Not Responding to @claude Commands

| Issue | Solution |
|-------|----------|
| Pipeline not triggered | Verify trigger configuration |
| Missing variables | Check CI/CD variable settings |
| Wrong mention format | Use `@claude` not `/claude` |

### Job Can't Write Comments or Open MRs

| Issue | Solution |
|-------|----------|
| Insufficient permissions | Use PAT with `api` scope |
| MCP tool not enabled | Add `mcp__gitlab` to `--allowedTools` |
| Missing context | Pass `AI_FLOW_*` variables |

### Authentication Errors

| Provider | Solution |
|----------|----------|
| **Claude API** | Verify `ANTHROPIC_API_KEY` is valid |
| **Bedrock** | Check OIDC/WIF config, role permissions, region |
| **Vertex AI** | Verify WIF setup, service account permissions |

---

## Part 10: Advanced Configuration

### Common Parameters

| Parameter | Description |
|-----------|-------------|
| `-p` / `--prompt` | Provide instructions inline |
| `--prompt-file` | Load instructions from file |
| `--max-turns` | Limit iterations |
| `--timeout-minutes` | Limit execution time |
| `--permission-mode` | Set permission level |

### Environment Variables

| Variable | Description |
|----------|-------------|
| `ANTHROPIC_API_KEY` | Claude API key |
| `AWS_REGION` | Bedrock region |
| `CLOUD_ML_REGION` | Vertex AI region |
| `AI_FLOW_INPUT` | User's request |
| `AI_FLOW_CONTEXT` | Issue/MR context |
| `AI_FLOW_EVENT` | Trigger event type |

### Custom Prompts per Job Type

```yaml
stages:
  - ai

review-job:
  extends: .claude-base
  script:
    - claude -p "Review this MR for bugs and code quality issues"

implement-job:
  extends: .claude-base
  script:
    - claude -p "Implement the feature described in this issue"

refactor-job:
  extends: .claude-base
  script:
    - claude -p "Refactor for better maintainability and add tests"

.claude-base:
  stage: ai
  image: node:24-alpine3.21
  before_script:
    - apk add --no-cache git curl bash
    - curl -fsSL https://claude.ai/install.sh | bash
  variables:
    GIT_STRATEGY: fetch
```

---

## Quick Reference

### Setup Checklist

- [ ] Add `ANTHROPIC_API_KEY` as CI/CD variable (or configure OIDC)
- [ ] Create `.gitlab-ci.yml` with Claude job
- [ ] Create `CLAUDE.md` with project guidelines
- [ ] Test with manual pipeline trigger
- [ ] Configure mention triggers (optional)

### Provider Comparison

| Feature | Claude API | AWS Bedrock | Google Vertex AI |
|---------|------------|-------------|------------------|
| Setup complexity | Low | Medium | Medium |
| Authentication | API key | OIDC | WIF |
| Data residency | N/A | Regional | Regional |
| Enterprise billing | Direct | AWS account | GCP account |

### Common Commands

```yaml
# Basic usage
claude -p "Your prompt here"

# With permission mode
claude -p "..." --permission-mode acceptEdits

# With tool restrictions
claude -p "..." --allowedTools "Bash(*) Read(*) Edit(*) Write(*)"

# With debugging
claude -p "..." --debug

# With iteration limit
claude -p "..." --max-turns 10
```

---

## Sources

- [Claude Code GitLab CI/CD - Official Documentation](https://code.claude.com/docs/en/gitlab-ci-cd.md)
- [Claude Code CLI Reference](https://code.claude.com/docs/en/cli-reference.md)
- [Headless Mode](https://code.claude.com/docs/en/headless.md)

---

## Next Steps

1. **Start with Quick Setup** - Get basic integration working
2. **Add CLAUDE.md** - Define your project guidelines
3. **Configure Enterprise Provider** - Set up Bedrock or Vertex AI if needed
4. **Create Custom Jobs** - Define jobs for different task types
5. **Monitor and Optimize** - Track costs and performance
