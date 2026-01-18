# Tutorial 42: Enterprise Deployment

> **Deploy Claude Code for enterprise teams**

## Overview

Guidelines for deploying Claude Code in enterprise environments.

---

## Part 1: Deployment Options

### Anthropic API (SaaS)

- Direct connection to Anthropic
- Simplest setup
- Standard pricing

### AWS Bedrock

- Data stays in AWS
- Regional compliance
- OIDC authentication

### Google Vertex AI

- Data stays in GCP
- Workload Identity
- Enterprise controls

---

## Part 2: Authentication

### Central Management

```json
// Managed settings (enterprise only)
{
  "apiProvider": "bedrock",
  "awsRegion": "us-west-2"
}
```

### OIDC Setup (Recommended)

No long-lived keys:
- AWS: IAM role assumption
- GCP: Workload Identity Federation

---

## Part 3: Access Controls

### Marketplace Restrictions

```json
{
  "strictKnownMarketplaces": [
    {
      "source": "github",
      "repo": "company/approved-plugins"
    }
  ]
}
```

### Plugin Controls

```json
{
  "enabledPlugins": {
    "security-tools@company": true
  }
}
```

---

## Part 4: Cost Management

### Rate Limits

| Team Size | TPM/User |
|-----------|----------|
| 1-5 | 200k-300k |
| 20-50 | 50k-75k |
| 100-500 | 15k-20k |

### Workspace Limits

Set spending limits in Claude Console.

### Tracking

- Claude Console for usage
- LiteLLM for Bedrock/Vertex

---

## Part 5: Security

### Permission Defaults

```json
{
  "permissions": {
    "defaultMode": "default",
    "deny": [
      "Bash(rm -rf *)",
      "Read(~/.ssh/*)"
    ]
  }
}
```

### Audit Requirements

- Enable logging
- Use enterprise providers
- Review all generated code

---

## Part 6: Rollout Strategy

### Pilot Phase

1. Select 5-10 users
2. Establish baselines
3. Gather feedback

### Expansion

1. Department by department
2. Monitor costs
3. Adjust rate limits

### Full Deployment

1. Organization-wide
2. Training materials
3. Support channels

---

## Quick Reference

### Enterprise Checklist

- [ ] Choose provider (API/Bedrock/Vertex)
- [ ] Configure authentication
- [ ] Set rate limits
- [ ] Restrict marketplaces
- [ ] Define permissions
- [ ] Plan rollout
- [ ] Enable monitoring

### Provider Comparison

| Feature | API | Bedrock | Vertex |
|---------|-----|---------|--------|
| Setup | Simple | Medium | Medium |
| Data residency | N/A | AWS | GCP |
| Billing | Anthropic | AWS | GCP |

---

## Sources

- [Amazon Bedrock](https://code.claude.com/docs/en/amazon-bedrock.md)
- [Google Vertex AI](https://code.claude.com/docs/en/google-vertex-ai.md)
- [Costs](https://code.claude.com/docs/en/costs.md)
