# Tutorial 24: Google Vertex AI Integration with Claude Code

## Complete Guide to Enterprise Claude Deployment on Google Cloud Platform

---

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Core Concepts](#core-concepts)
4. [Understanding Vertex AI](#understanding-vertex-ai)
5. [GCP Project Setup](#gcp-project-setup)
6. [Authentication Methods](#authentication-methods)
7. [Configuring Claude Code for Vertex AI](#configuring-claude-code-for-vertex-ai)
8. [Real-World Examples](#real-world-examples)
9. [Available Models and Regions](#available-models-and-regions)
10. [Troubleshooting](#troubleshooting)
11. [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)
12. [Best Practices](#best-practices)

---

## Overview

### What is Google Vertex AI?

Google Vertex AI is Google Cloud Platform's unified machine learning platform that brings together all Google Cloud services for building ML models and deploying AI applications. Through a partnership between Anthropic and Google Cloud, Claude models are available directly on Vertex AI, allowing enterprises to access Claude's capabilities through their existing Google Cloud infrastructure.

### Why Use Vertex AI for Claude?

**Enterprise Integration**
- Seamless integration with existing GCP infrastructure
- Unified billing through Google Cloud
- Centralized access management via Google Cloud IAM
- Network security through VPC Service Controls

**Compliance and Security**
- Data residency controls with regional deployments
- SOC 2 Type II compliance
- HIPAA eligibility (with BAA)
- GDPR compliance support
- FedRAMP authorization (in progress)

**Operational Benefits**
- No separate API keys to manage
- Leverage existing GCP service accounts
- Integrated logging with Cloud Logging
- Monitoring through Cloud Monitoring
- Cost tracking via GCP billing

**Scalability**
- Enterprise-grade rate limits
- Automatic scaling
- Multi-region availability
- High availability SLAs

### When to Choose Vertex AI Over Direct API

| Consideration | Direct Anthropic API | Vertex AI |
|--------------|---------------------|-----------|
| Quick start | Faster setup | More setup required |
| Enterprise compliance | Manual configuration | Built-in compliance |
| Existing GCP investment | N/A | Leverage existing infra |
| Billing consolidation | Separate billing | Unified GCP billing |
| Data residency | Limited control | Regional control |
| Network security | Public endpoint | VPC support |
| Identity management | API keys | Google IAM |

---

## Prerequisites

### Required Accounts and Access

Before beginning this tutorial, ensure you have:

#### 1. Google Cloud Platform Account
```
- Active GCP account with billing enabled
- Organization access (for enterprise features)
- Project Owner or Editor role (for initial setup)
```

#### 2. Claude Code Installation
```bash
# Verify Claude Code is installed
claude --version

# If not installed, install via npm
npm install -g @anthropic-ai/claude-code
```

#### 3. Google Cloud SDK (gcloud CLI)
```bash
# Check if gcloud is installed
gcloud --version

# If not installed, download from:
# https://cloud.google.com/sdk/docs/install

# Initialize gcloud
gcloud init
```

#### 4. Required Permissions

For complete setup, you need these IAM permissions:
```
- roles/serviceusage.serviceUsageAdmin (enable APIs)
- roles/iam.serviceAccountAdmin (create service accounts)
- roles/iam.serviceAccountKeyAdmin (create keys)
- roles/resourcemanager.projectIamAdmin (grant permissions)
- roles/aiplatform.admin (manage Vertex AI)
```

### Verification Checklist

Run these commands to verify your setup:

```bash
# Verify gcloud authentication
gcloud auth list

# Check current project
gcloud config get-value project

# Verify billing is enabled
gcloud billing accounts list

# Check available regions
gcloud ai locations list
```

---

## Core Concepts

### Understanding Google Cloud Identity and Access Management (IAM)

IAM is Google Cloud's system for managing access to resources. Understanding IAM is essential for secure Vertex AI integration.

#### Principals (Who)
```
- User accounts (you@company.com)
- Service accounts (sa@project.iam.gserviceaccount.com)
- Groups (team@company.com)
- Workload Identity (external systems)
```

#### Roles (What permissions)
```
roles/aiplatform.user       - Use AI Platform resources
roles/aiplatform.admin      - Full AI Platform access
roles/aiplatform.viewer     - Read-only access
```

#### Resources (On what)
```
- Projects
- Specific Vertex AI resources
- Folders (organizational)
```

### Workload Identity Federation

Workload Identity Federation allows external identities (like GitHub Actions, AWS, Azure) to access GCP resources without service account keys.

**How it works:**
```
1. External system (GitHub) issues an OIDC token
2. Token is exchanged with GCP Security Token Service
3. STS returns short-lived GCP credentials
4. Credentials are used to access Vertex AI
```

**Benefits:**
- No long-lived credentials to manage
- Automatic credential rotation
- Audit trail of external access
- Reduced security risk

### Regions and Data Residency

Claude on Vertex AI is available in specific regions. Your choice affects:

**Latency**: Choose regions close to your users
**Data residency**: Keep data in specific geographic areas
**Availability**: Some models may have region-specific availability
**Pricing**: Costs may vary by region

**Available Regions:**
```
us-central1      - Iowa, USA
us-east4         - Northern Virginia, USA
europe-west1     - Belgium, Europe
asia-northeast1  - Tokyo, Japan
```

### Authentication Flow

Understanding the authentication flow helps troubleshoot issues:

```
┌─────────────────┐
│  Claude Code    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Check for creds │
│ in this order:  │
└────────┬────────┘
         │
    ┌────┴────┐
    ▼         ▼
┌───────┐ ┌──────────────┐
│ ADC   │ │ Service Acct │
│       │ │ Key File     │
└───┬───┘ └──────┬───────┘
    │            │
    └─────┬──────┘
          ▼
┌─────────────────┐
│  GCP IAM Auth   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Vertex AI API  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Claude Model   │
└─────────────────┘
```

---

## Understanding Vertex AI

### Vertex AI Service Overview

Vertex AI is Google Cloud's comprehensive ML platform that provides:

**Model Garden**
- Access to first-party Google models
- Third-party models including Claude
- Open-source model deployment
- Custom model hosting

**Development Tools**
- Jupyter notebooks
- Training pipelines
- Feature store
- Model registry

**Deployment Infrastructure**
- Online prediction endpoints
- Batch prediction
- Auto-scaling
- A/B testing

### Claude Models on Vertex AI

Anthropic's Claude models are available through Vertex AI's Model Garden as "Partner Models." This means:

**Direct Access**
- No separate Anthropic account needed
- Models run on Google infrastructure
- GCP handles the model serving

**API Compatibility**
- Same capabilities as direct API
- Identical model behavior
- Compatible request/response format

**Enterprise Features**
- VPC Service Controls support
- Customer-managed encryption keys (CMEK)
- Cloud Audit Logs integration

### Pricing Comparison

Vertex AI pricing for Claude models uses a pay-per-use model based on tokens:

| Model | Input (per 1M tokens) | Output (per 1M tokens) |
|-------|----------------------|------------------------|
| Claude 3.5 Sonnet | $3.00 | $15.00 |
| Claude 3 Opus | $15.00 | $75.00 |
| Claude 3 Sonnet | $3.00 | $15.00 |
| Claude 3 Haiku | $0.25 | $1.25 |

**Note:** Prices may vary. Check current pricing at:
https://cloud.google.com/vertex-ai/generative-ai/pricing

**Cost Optimization Tips:**
```
1. Use appropriate model for task complexity
2. Implement prompt caching where available
3. Set up budget alerts
4. Monitor usage with Cloud Monitoring
5. Use batch processing for non-urgent tasks
```

### Enterprise Benefits

#### Business Associate Agreement (BAA)

For healthcare organizations handling PHI (Protected Health Information):

```
1. GCP offers BAA coverage for Vertex AI
2. Claude on Vertex AI can be covered under BAA
3. Enables HIPAA-compliant deployments
4. Required for healthcare AI applications
```

**To obtain BAA:**
- Contact Google Cloud sales
- Review covered services
- Sign BAA amendment
- Configure compliant settings

#### Compliance Certifications

**SOC 2 Type II**
- Security controls audited
- Annual recertification
- Trust service criteria covered

**ISO 27001**
- Information security management
- International standard
- Continuous compliance

**GDPR**
- Data processing agreements
- Data residency options
- Right to erasure support

#### VPC Service Controls

Protect sensitive data with network-level security:

```
┌─────────────────────────────────────┐
│         VPC Service Perimeter       │
│  ┌─────────────────────────────┐    │
│  │     Your GCP Project        │    │
│  │  ┌───────┐    ┌──────────┐  │    │
│  │  │ App   │───▶│ Vertex AI│  │    │
│  │  └───────┘    └──────────┘  │    │
│  └─────────────────────────────┘    │
│                                     │
│  ✗ No data exfiltration             │
│  ✗ No unauthorized access           │
└─────────────────────────────────────┘
```

---

## GCP Project Setup

### Step 1: Create or Select a Project

#### Creating a New Project

```bash
# Create new project
gcloud projects create PROJECT_ID \
  --name="Claude Code Vertex AI" \
  --organization=ORGANIZATION_ID

# Example with specific project ID
gcloud projects create my-claude-project-2024 \
  --name="Claude Code Production" \
  --organization=123456789012

# Set as current project
gcloud config set project my-claude-project-2024
```

#### Using Existing Project

```bash
# List available projects
gcloud projects list

# Set current project
gcloud config set project existing-project-id

# Verify current project
gcloud config get-value project
```

### Step 2: Enable Billing

```bash
# List billing accounts
gcloud billing accounts list

# Link billing to project
gcloud billing projects link PROJECT_ID \
  --billing-account=BILLING_ACCOUNT_ID

# Example
gcloud billing projects link my-claude-project-2024 \
  --billing-account=0X0X0X-0X0X0X-0X0X0X
```

### Step 3: Enable Required APIs

#### Enable Vertex AI API

```bash
# Enable the Vertex AI API
gcloud services enable aiplatform.googleapis.com

# Verify API is enabled
gcloud services list --enabled | grep aiplatform
```

#### Enable Additional Required APIs

```bash
# Enable all required APIs at once
gcloud services enable \
  aiplatform.googleapis.com \
  iam.googleapis.com \
  iamcredentials.googleapis.com \
  cloudresourcemanager.googleapis.com \
  sts.googleapis.com

# Verify all APIs are enabled
gcloud services list --enabled
```

### Step 4: Enable Claude Model Access

Claude models on Vertex AI require explicit enablement:

```bash
# Via gcloud (if available)
gcloud ai models list --region=us-central1

# Or enable via Cloud Console:
# 1. Go to Vertex AI > Model Garden
# 2. Search for "Claude"
# 3. Click "Enable" for desired models
# 4. Accept terms of service
```

**Manual Enablement via Console:**

```
1. Navigate to: https://console.cloud.google.com/vertex-ai/model-garden
2. Search for "Anthropic" or "Claude"
3. Select the Claude model you want to use
4. Click "Enable"
5. Review and accept the terms
6. Wait for enablement to complete
```

### Step 5: Project Configuration

#### Set Default Region

```bash
# Set default region for Vertex AI
gcloud config set ai/region us-central1

# Verify configuration
gcloud config list
```

#### Configure Project Defaults

```bash
# Create configuration file for project
gcloud config configurations create claude-vertex

# Set all defaults
gcloud config set project my-claude-project-2024
gcloud config set ai/region us-central1
gcloud config set compute/region us-central1

# List configurations
gcloud config configurations list

# Switch between configurations
gcloud config configurations activate claude-vertex
```

---

## Service Account Setup

### Understanding Service Accounts

Service accounts are special Google accounts that represent applications rather than users. They are the recommended way to authenticate Claude Code in automated environments.

### Creating a Service Account

#### Step 1: Create the Service Account

```bash
# Create service account
gcloud iam service-accounts create claude-code-sa \
  --display-name="Claude Code Service Account" \
  --description="Service account for Claude Code Vertex AI access"

# Verify creation
gcloud iam service-accounts list
```

#### Step 2: Grant Required Roles

**Minimal Required Role:**

```bash
# Grant Vertex AI User role (minimal permissions)
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:claude-code-sa@PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/aiplatform.user"

# Example with actual project ID
gcloud projects add-iam-policy-binding my-claude-project-2024 \
  --member="serviceAccount:claude-code-sa@my-claude-project-2024.iam.gserviceaccount.com" \
  --role="roles/aiplatform.user"
```

**Additional Roles for Extended Functionality:**

```bash
# For logging access (recommended)
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:claude-code-sa@PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/logging.viewer"

# For monitoring access (recommended)
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:claude-code-sa@PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/monitoring.viewer"
```

#### Role Reference Table

| Role | Purpose | Required |
|------|---------|----------|
| roles/aiplatform.user | Invoke Claude models | Yes |
| roles/logging.viewer | View logs | Recommended |
| roles/monitoring.viewer | View metrics | Recommended |
| roles/aiplatform.admin | Full Vertex AI access | Optional |

### Key Management

#### Creating a Service Account Key

**Warning:** Service account keys are long-lived credentials. Use Workload Identity Federation when possible.

```bash
# Create key file
gcloud iam service-accounts keys create ~/claude-vertex-key.json \
  --iam-account=claude-code-sa@PROJECT_ID.iam.gserviceaccount.com

# Example
gcloud iam service-accounts keys create ~/claude-vertex-key.json \
  --iam-account=claude-code-sa@my-claude-project-2024.iam.gserviceaccount.com

# Set restrictive permissions on key file
chmod 600 ~/claude-vertex-key.json

# On Windows (PowerShell)
# icacls $HOME\claude-vertex-key.json /inheritance:r /grant:r "$($env:USERNAME):(R)"
```

#### Key Security Best Practices

```bash
# 1. List existing keys to audit
gcloud iam service-accounts keys list \
  --iam-account=claude-code-sa@PROJECT_ID.iam.gserviceaccount.com

# 2. Delete unused keys
gcloud iam service-accounts keys delete KEY_ID \
  --iam-account=claude-code-sa@PROJECT_ID.iam.gserviceaccount.com

# 3. Set key expiration (organization policy)
# Requires organization admin access
```

#### Key Rotation Schedule

Implement regular key rotation:

```bash
#!/bin/bash
# key-rotation.sh - Rotate service account keys

SA_EMAIL="claude-code-sa@PROJECT_ID.iam.gserviceaccount.com"
KEY_PATH="$HOME/claude-vertex-key.json"
BACKUP_PATH="$HOME/claude-vertex-key.backup.json"

# Backup old key
cp "$KEY_PATH" "$BACKUP_PATH"

# Create new key
gcloud iam service-accounts keys create "$KEY_PATH.new" \
  --iam-account="$SA_EMAIL"

# Test new key
export GOOGLE_APPLICATION_CREDENTIALS="$KEY_PATH.new"
# ... run test command ...

# If test passes, replace old key
mv "$KEY_PATH.new" "$KEY_PATH"

# Delete old key from GCP (get key ID first)
OLD_KEY_ID=$(gcloud iam service-accounts keys list \
  --iam-account="$SA_EMAIL" \
  --format="value(name)" | head -1)

gcloud iam service-accounts keys delete "$OLD_KEY_ID" \
  --iam-account="$SA_EMAIL" --quiet

echo "Key rotation complete"
```

---

## Authentication Methods

### Method 1: Application Default Credentials (ADC)

ADC is the recommended authentication method for local development. It uses your personal Google Cloud credentials.

#### Interactive Login

```bash
# Standard login (for gcloud commands)
gcloud auth login

# This opens a browser for authentication
# Follow the prompts to authenticate
```

#### Application Default Credentials Login

```bash
# Set up ADC (for application authentication)
gcloud auth application-default login

# This creates credentials at:
# Linux/macOS: ~/.config/gcloud/application_default_credentials.json
# Windows: %APPDATA%\gcloud\application_default_credentials.json
```

#### Setting Project for ADC

```bash
# Set project for ADC
gcloud auth application-default set-quota-project PROJECT_ID

# Example
gcloud auth application-default set-quota-project my-claude-project-2024
```

#### When to Use Each Login Type

| Command | Use Case |
|---------|----------|
| `gcloud auth login` | Running gcloud CLI commands |
| `gcloud auth application-default login` | Applications using Google client libraries |

#### Verifying ADC Setup

```bash
# Check current ADC configuration
gcloud auth application-default print-access-token

# If this returns a token, ADC is configured correctly
```

### Method 2: Workload Identity Federation

Workload Identity Federation is the most secure method for automated environments. It eliminates the need for service account keys.

#### Setting Up Workload Identity Pool

```bash
# Create workload identity pool
gcloud iam workload-identity-pools create claude-pool \
  --location="global" \
  --display-name="Claude Code Pool" \
  --description="Workload Identity Pool for Claude Code"

# Verify creation
gcloud iam workload-identity-pools describe claude-pool \
  --location="global"
```

#### Creating OIDC Provider for GitHub Actions

```bash
# Create OIDC provider for GitHub
gcloud iam workload-identity-pools providers create-oidc github-provider \
  --location="global" \
  --workload-identity-pool="claude-pool" \
  --display-name="GitHub Actions Provider" \
  --description="OIDC provider for GitHub Actions" \
  --attribute-mapping="google.subject=assertion.sub,attribute.actor=assertion.actor,attribute.repository=assertion.repository,attribute.repository_owner=assertion.repository_owner" \
  --issuer-uri="https://token.actions.githubusercontent.com"
```

#### Granting Service Account Impersonation

```bash
# Get project number
PROJECT_NUM=$(gcloud projects describe PROJECT_ID --format="value(projectNumber)")

# Grant impersonation for specific repository
gcloud iam service-accounts add-iam-policy-binding \
  claude-code-sa@PROJECT_ID.iam.gserviceaccount.com \
  --role="roles/iam.workloadIdentityUser" \
  --member="principalSet://iam.googleapis.com/projects/${PROJECT_NUM}/locations/global/workloadIdentityPools/claude-pool/attribute.repository/OWNER/REPO"

# Example for repository myorg/myrepo
gcloud iam service-accounts add-iam-policy-binding \
  claude-code-sa@my-claude-project-2024.iam.gserviceaccount.com \
  --role="roles/iam.workloadIdentityUser" \
  --member="principalSet://iam.googleapis.com/projects/123456789012/locations/global/workloadIdentityPools/claude-pool/attribute.repository/myorg/myrepo"
```

#### Complete Workload Identity Setup Script

```bash
#!/bin/bash
# setup-workload-identity.sh

# Configuration
PROJECT_ID="your-project-id"
POOL_NAME="claude-pool"
PROVIDER_NAME="github-provider"
SA_NAME="claude-code-sa"
GITHUB_ORG="your-org"
GITHUB_REPO="your-repo"

# Get project number
PROJECT_NUM=$(gcloud projects describe $PROJECT_ID --format="value(projectNumber)")

# Create pool
echo "Creating workload identity pool..."
gcloud iam workload-identity-pools create $POOL_NAME \
  --location="global" \
  --display-name="Claude Code Pool"

# Create provider
echo "Creating OIDC provider..."
gcloud iam workload-identity-pools providers create-oidc $PROVIDER_NAME \
  --location="global" \
  --workload-identity-pool="$POOL_NAME" \
  --display-name="GitHub Provider" \
  --attribute-mapping="google.subject=assertion.sub,attribute.actor=assertion.actor,attribute.repository=assertion.repository" \
  --issuer-uri="https://token.actions.githubusercontent.com"

# Grant impersonation
echo "Granting service account impersonation..."
gcloud iam service-accounts add-iam-policy-binding \
  ${SA_NAME}@${PROJECT_ID}.iam.gserviceaccount.com \
  --role="roles/iam.workloadIdentityUser" \
  --member="principalSet://iam.googleapis.com/projects/${PROJECT_NUM}/locations/global/workloadIdentityPools/${POOL_NAME}/attribute.repository/${GITHUB_ORG}/${GITHUB_REPO}"

# Output configuration for GitHub Actions
echo ""
echo "GitHub Actions Configuration:"
echo "=============================="
echo "workload_identity_provider: projects/${PROJECT_NUM}/locations/global/workloadIdentityPools/${POOL_NAME}/providers/${PROVIDER_NAME}"
echo "service_account: ${SA_NAME}@${PROJECT_ID}.iam.gserviceaccount.com"
```

### Method 3: Service Account Key File

Use service account key files when Workload Identity is not available.

#### Setting Up Key-Based Authentication

```bash
# Create key file (if not already done)
gcloud iam service-accounts keys create ~/claude-vertex-key.json \
  --iam-account=claude-code-sa@PROJECT_ID.iam.gserviceaccount.com

# Set environment variable
export GOOGLE_APPLICATION_CREDENTIALS=~/claude-vertex-key.json

# Verify authentication works
gcloud auth list
```

#### Secure Key Storage

**Local Development:**

```bash
# Store in user directory with restricted permissions
chmod 600 ~/claude-vertex-key.json

# Add to .bashrc or .zshrc
echo 'export GOOGLE_APPLICATION_CREDENTIALS=~/claude-vertex-key.json' >> ~/.bashrc
```

**CI/CD Environments:**

```bash
# Store as secret and write to file
echo "$GCP_SA_KEY" > /tmp/gcp-key.json
export GOOGLE_APPLICATION_CREDENTIALS=/tmp/gcp-key.json

# Clean up after use
trap "rm -f /tmp/gcp-key.json" EXIT
```

#### Security Considerations for Key Files

**DO:**
- Store keys in secret managers
- Use restrictive file permissions
- Rotate keys regularly (90 days recommended)
- Monitor key usage
- Delete keys when no longer needed

**DON'T:**
- Commit keys to version control
- Share keys between environments
- Use the same key for multiple applications
- Store keys in plain text in CI/CD configs

---

## Configuring Claude Code for Vertex AI

### Environment Variables

#### Required Environment Variables

```bash
# Project ID (required)
export GOOGLE_CLOUD_PROJECT=your-project-id

# Region (required)
export GOOGLE_CLOUD_REGION=us-central1

# Credentials path (if using service account key)
export GOOGLE_APPLICATION_CREDENTIALS=~/claude-vertex-key.json
```

#### Complete Environment Setup

```bash
# Add to ~/.bashrc or ~/.zshrc for persistence

# GCP Configuration
export GOOGLE_CLOUD_PROJECT=my-claude-project-2024
export GOOGLE_CLOUD_REGION=us-central1
export GOOGLE_APPLICATION_CREDENTIALS=~/claude-vertex-key.json

# Optional: Set default provider for Claude Code
export CLAUDE_CODE_PROVIDER=vertexai
export CLAUDE_CODE_MODEL=claude-3-5-sonnet@20240620
```

#### Windows Environment Variables

```powershell
# PowerShell (session)
$env:GOOGLE_CLOUD_PROJECT = "your-project-id"
$env:GOOGLE_CLOUD_REGION = "us-central1"
$env:GOOGLE_APPLICATION_CREDENTIALS = "$HOME\claude-vertex-key.json"

# Permanent (requires admin)
[System.Environment]::SetEnvironmentVariable("GOOGLE_CLOUD_PROJECT", "your-project-id", "User")
[System.Environment]::SetEnvironmentVariable("GOOGLE_CLOUD_REGION", "us-central1", "User")
[System.Environment]::SetEnvironmentVariable("GOOGLE_APPLICATION_CREDENTIALS", "$HOME\claude-vertex-key.json", "User")
```

### CLI Configuration

#### Using the --provider Flag

```bash
# Basic usage with Vertex AI
claude --provider vertexai "Explain this code"

# With specific model
claude --provider vertexai --model claude-3-5-sonnet@20240620 "Analyze this function"

# With region specification
claude --provider vertexai --model claude-3-opus@20240229 --region us-east4 "Review this PR"
```

#### Available CLI Flags for Vertex AI

| Flag | Description | Example |
|------|-------------|---------|
| `--provider` | Set provider to vertexai | `--provider vertexai` |
| `--model` | Specify Claude model | `--model claude-3-5-sonnet@20240620` |
| `--region` | GCP region | `--region us-central1` |
| `--project` | GCP project ID | `--project my-project` |

### Settings File Configuration

#### settings.json for Vertex AI

Create or edit the Claude Code settings file:

**Location:**
- Linux/macOS: `~/.config/claude-code/settings.json`
- Windows: `%APPDATA%\claude-code\settings.json`

```json
{
  "provider": "vertexai",
  "model": "claude-3-5-sonnet@20240620",
  "vertexai": {
    "project": "your-project-id",
    "region": "us-central1"
  }
}
```

#### Advanced Settings Configuration

```json
{
  "provider": "vertexai",
  "model": "claude-3-5-sonnet@20240620",
  "vertexai": {
    "project": "your-project-id",
    "region": "us-central1",
    "credentials": "~/claude-vertex-key.json"
  },
  "defaults": {
    "maxTokens": 4096,
    "temperature": 0.7
  },
  "logging": {
    "level": "info",
    "file": "~/.config/claude-code/claude.log"
  }
}
```

#### Project-Specific Configuration

Create `.claude-code.json` in your project root:

```json
{
  "provider": "vertexai",
  "model": "claude-3-sonnet@20240229",
  "vertexai": {
    "project": "team-project-id",
    "region": "europe-west1"
  },
  "context": {
    "includeGitHistory": true,
    "includeFileTree": true
  }
}
```

### Verifying Configuration

```bash
# Test Vertex AI connection
claude --provider vertexai --print "Hello, are you connected via Vertex AI?"

# Check current configuration
claude config show

# Verbose mode for debugging
claude --provider vertexai --verbose "Test message"
```

---

## Real-World Examples

### Example 1: Enterprise Setup with Corporate GCP

**Scenario:** Large enterprise with existing GCP organization, multiple teams, and compliance requirements.

#### Organization Structure

```
Organization: acme-corp.com
├── Folder: Engineering
│   ├── Project: eng-dev (development)
│   ├── Project: eng-staging (staging)
│   └── Project: eng-prod (production)
└── Folder: Data Science
    └── Project: ds-ml (ML workloads)
```

#### Setup Steps

```bash
# 1. Create dedicated project under Engineering folder
gcloud projects create eng-claude-prod \
  --folder=ENGINEERING_FOLDER_ID \
  --name="Claude Production"

# 2. Enable APIs with organization policy compliance
gcloud services enable aiplatform.googleapis.com \
  --project=eng-claude-prod

# 3. Create service account with naming convention
gcloud iam service-accounts create svc-claude-prod \
  --display-name="Claude Production Service Account" \
  --project=eng-claude-prod

# 4. Apply organization's IAM policies
gcloud projects add-iam-policy-binding eng-claude-prod \
  --member="serviceAccount:svc-claude-prod@eng-claude-prod.iam.gserviceaccount.com" \
  --role="roles/aiplatform.user"

# 5. Enable audit logging
gcloud projects get-iam-policy eng-claude-prod > policy.yaml
# Add audit logging configuration
# Apply updated policy
gcloud projects set-iam-policy eng-claude-prod policy.yaml
```

#### Team Access Configuration

```bash
# Create Google Group for team
# (Done via Google Admin Console)

# Grant group access to Claude
gcloud projects add-iam-policy-binding eng-claude-prod \
  --member="group:claude-users@acme-corp.com" \
  --role="roles/aiplatform.user"

# Grant individual developer access
gcloud projects add-iam-policy-binding eng-claude-prod \
  --member="user:developer@acme-corp.com" \
  --role="roles/aiplatform.user"
```

### Example 2: CI/CD Pipeline with Vertex AI

**Scenario:** Automated code review and analysis in CI/CD pipeline.

#### GitHub Actions Workflow

```yaml
# .github/workflows/claude-review.yml
name: Claude Code Review

on:
  pull_request:
    branches: [main, develop]

jobs:
  code-review:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
      id-token: write

    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Authenticate to Google Cloud
        uses: google-github-actions/auth@v2
        with:
          workload_identity_provider: 'projects/123456789012/locations/global/workloadIdentityPools/claude-pool/providers/github-provider'
          service_account: 'claude-code-sa@my-project.iam.gserviceaccount.com'

      - name: Set up Cloud SDK
        uses: google-github-actions/setup-gcloud@v2

      - name: Install Claude Code
        run: npm install -g @anthropic-ai/claude-code

      - name: Get changed files
        id: changed-files
        run: |
          echo "files=$(git diff --name-only origin/main...HEAD | tr '\n' ' ')" >> $GITHUB_OUTPUT

      - name: Run Claude Code Review
        env:
          GOOGLE_CLOUD_PROJECT: my-project
          GOOGLE_CLOUD_REGION: us-central1
        run: |
          claude --provider vertexai \
            --model claude-3-sonnet@20240229 \
            --print \
            "Review these changed files for bugs, security issues, and code quality: ${{ steps.changed-files.outputs.files }}" \
            > review.md

      - name: Post review comment
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const review = fs.readFileSync('review.md', 'utf8');
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `## Claude Code Review\n\n${review}`
            });
```

#### GitLab CI Configuration

```yaml
# .gitlab-ci.yml
stages:
  - review

claude-review:
  stage: review
  image: google/cloud-sdk:latest
  id_tokens:
    GITLAB_OIDC_TOKEN:
      aud: https://iam.googleapis.com/projects/PROJECT_NUM/locations/global/workloadIdentityPools/gitlab-pool/providers/gitlab-provider
  script:
    # Authenticate using Workload Identity
    - |
      gcloud iam workload-identity-pools create-cred-config \
        projects/PROJECT_NUM/locations/global/workloadIdentityPools/gitlab-pool/providers/gitlab-provider \
        --service-account=claude-code-sa@PROJECT_ID.iam.gserviceaccount.com \
        --output-file=/tmp/creds.json \
        --credential-source-file=/tmp/gitlab-token
    - echo "$GITLAB_OIDC_TOKEN" > /tmp/gitlab-token
    - export GOOGLE_APPLICATION_CREDENTIALS=/tmp/creds.json

    # Install and run Claude Code
    - npm install -g @anthropic-ai/claude-code
    - |
      claude --provider vertexai \
        --model claude-3-sonnet@20240229 \
        --print \
        "Analyze the changes in this merge request"
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
```

### Example 3: Multi-Region Deployment

**Scenario:** Global organization requiring data residency compliance.

#### Region-Specific Configuration

```bash
# US Configuration
export CLAUDE_US_PROJECT=my-project-us
export CLAUDE_US_REGION=us-central1

# EU Configuration
export CLAUDE_EU_PROJECT=my-project-eu
export CLAUDE_EU_REGION=europe-west1

# APAC Configuration
export CLAUDE_APAC_PROJECT=my-project-apac
export CLAUDE_APAC_REGION=asia-northeast1
```

#### Region Selection Script

```bash
#!/bin/bash
# select-region.sh - Select Claude region based on user location

get_region_config() {
    case "$1" in
        us|US|united-states)
            echo "export GOOGLE_CLOUD_PROJECT=my-project-us"
            echo "export GOOGLE_CLOUD_REGION=us-central1"
            ;;
        eu|EU|europe)
            echo "export GOOGLE_CLOUD_PROJECT=my-project-eu"
            echo "export GOOGLE_CLOUD_REGION=europe-west1"
            ;;
        apac|APAC|asia)
            echo "export GOOGLE_CLOUD_PROJECT=my-project-apac"
            echo "export GOOGLE_CLOUD_REGION=asia-northeast1"
            ;;
        *)
            echo "Unknown region: $1"
            echo "Available: us, eu, apac"
            exit 1
            ;;
    esac
}

# Usage: source <(./select-region.sh us)
get_region_config "$1"
```

#### Multi-Region Terraform Configuration

```hcl
# terraform/main.tf

variable "regions" {
  default = {
    us = {
      project = "claude-us-prod"
      region  = "us-central1"
    }
    eu = {
      project = "claude-eu-prod"
      region  = "europe-west1"
    }
    apac = {
      project = "claude-apac-prod"
      region  = "asia-northeast1"
    }
  }
}

resource "google_project" "claude" {
  for_each = var.regions

  name       = "Claude ${upper(each.key)}"
  project_id = each.value.project
  org_id     = var.organization_id
}

resource "google_project_service" "vertexai" {
  for_each = var.regions

  project = google_project.claude[each.key].project_id
  service = "aiplatform.googleapis.com"
}

resource "google_service_account" "claude" {
  for_each = var.regions

  project      = google_project.claude[each.key].project_id
  account_id   = "claude-code-sa"
  display_name = "Claude Code Service Account"
}

resource "google_project_iam_member" "claude_user" {
  for_each = var.regions

  project = google_project.claude[each.key].project_id
  role    = "roles/aiplatform.user"
  member  = "serviceAccount:${google_service_account.claude[each.key].email}"
}
```

### Example 4: Workload Identity for GitHub Actions (Complete Setup)

**Scenario:** Secure authentication from GitHub Actions without service account keys.

#### Complete Setup Script

```bash
#!/bin/bash
# setup-github-workload-identity.sh

set -e

# Configuration - Update these values
PROJECT_ID="your-project-id"
GITHUB_ORG="your-github-org"
GITHUB_REPO="your-repo-name"
SA_NAME="claude-github-sa"
POOL_NAME="github-pool"
PROVIDER_NAME="github-provider"

echo "Setting up Workload Identity Federation for GitHub Actions"
echo "==========================================================="

# Get project number
PROJECT_NUM=$(gcloud projects describe $PROJECT_ID --format="value(projectNumber)")
echo "Project Number: $PROJECT_NUM"

# Create service account
echo "Creating service account..."
gcloud iam service-accounts create $SA_NAME \
  --display-name="Claude GitHub Actions SA" \
  --project=$PROJECT_ID || true

SA_EMAIL="${SA_NAME}@${PROJECT_ID}.iam.gserviceaccount.com"
echo "Service Account: $SA_EMAIL"

# Grant Vertex AI permissions
echo "Granting Vertex AI permissions..."
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:$SA_EMAIL" \
  --role="roles/aiplatform.user"

# Create workload identity pool
echo "Creating workload identity pool..."
gcloud iam workload-identity-pools create $POOL_NAME \
  --location="global" \
  --display-name="GitHub Actions Pool" \
  --project=$PROJECT_ID || true

# Create OIDC provider
echo "Creating OIDC provider..."
gcloud iam workload-identity-pools providers create-oidc $PROVIDER_NAME \
  --location="global" \
  --workload-identity-pool=$POOL_NAME \
  --display-name="GitHub Provider" \
  --attribute-mapping="google.subject=assertion.sub,attribute.actor=assertion.actor,attribute.repository=assertion.repository,attribute.repository_owner=assertion.repository_owner" \
  --issuer-uri="https://token.actions.githubusercontent.com" \
  --project=$PROJECT_ID || true

# Allow impersonation
echo "Setting up service account impersonation..."
gcloud iam service-accounts add-iam-policy-binding $SA_EMAIL \
  --role="roles/iam.workloadIdentityUser" \
  --member="principalSet://iam.googleapis.com/projects/${PROJECT_NUM}/locations/global/workloadIdentityPools/${POOL_NAME}/attribute.repository/${GITHUB_ORG}/${GITHUB_REPO}" \
  --project=$PROJECT_ID

# Output configuration
echo ""
echo "=========================================="
echo "Setup Complete! Add these to your GitHub Actions workflow:"
echo "=========================================="
echo ""
echo "workload_identity_provider: projects/${PROJECT_NUM}/locations/global/workloadIdentityPools/${POOL_NAME}/providers/${PROVIDER_NAME}"
echo "service_account: $SA_EMAIL"
echo ""
echo "Example workflow step:"
echo "----------------------"
cat << 'EOF'
- uses: google-github-actions/auth@v2
  with:
    workload_identity_provider: 'projects/PROJECT_NUM/locations/global/workloadIdentityPools/github-pool/providers/github-provider'
    service_account: 'SA_EMAIL'
EOF
echo ""
echo "Replace PROJECT_NUM with: $PROJECT_NUM"
echo "Replace SA_EMAIL with: $SA_EMAIL"
```

#### GitHub Actions Workflow with Full Features

```yaml
# .github/workflows/claude-vertex.yml
name: Claude with Vertex AI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:
    inputs:
      task:
        description: 'Task for Claude'
        required: true
        default: 'Analyze the codebase'

env:
  PROJECT_ID: my-project-id
  REGION: us-central1
  MODEL: claude-3-5-sonnet@20240620

jobs:
  claude-task:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
      id-token: write

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Authenticate to Google Cloud
        id: auth
        uses: google-github-actions/auth@v2
        with:
          workload_identity_provider: 'projects/123456789012/locations/global/workloadIdentityPools/github-pool/providers/github-provider'
          service_account: 'claude-github-sa@my-project-id.iam.gserviceaccount.com'
          token_format: 'access_token'

      - name: Set up Cloud SDK
        uses: google-github-actions/setup-gcloud@v2
        with:
          project_id: ${{ env.PROJECT_ID }}

      - name: Install Claude Code
        run: npm install -g @anthropic-ai/claude-code

      - name: Run Claude Task
        env:
          GOOGLE_CLOUD_PROJECT: ${{ env.PROJECT_ID }}
          GOOGLE_CLOUD_REGION: ${{ env.REGION }}
        run: |
          TASK="${{ github.event.inputs.task || 'Analyze recent changes' }}"
          claude --provider vertexai \
            --model ${{ env.MODEL }} \
            --print \
            "$TASK" > claude-output.md

      - name: Upload Claude Output
        uses: actions/upload-artifact@v4
        with:
          name: claude-analysis
          path: claude-output.md

      - name: Comment on PR
        if: github.event_name == 'pull_request'
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const output = fs.readFileSync('claude-output.md', 'utf8');
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `## Claude Analysis\n\n${output}`
            });
```

### Example 5: Team Access with IAM

**Scenario:** Multiple teams with different access levels.

#### IAM Structure

```
Organization
├── Group: claude-admins@company.com
│   └── Role: roles/aiplatform.admin
├── Group: claude-developers@company.com
│   └── Role: roles/aiplatform.user
└── Group: claude-viewers@company.com
    └── Role: roles/aiplatform.viewer
```

#### Setup Script

```bash
#!/bin/bash
# setup-team-access.sh

PROJECT_ID="your-project-id"

# Admin access - full control
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="group:claude-admins@company.com" \
  --role="roles/aiplatform.admin"

# Developer access - use models
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="group:claude-developers@company.com" \
  --role="roles/aiplatform.user"

# Viewer access - read-only
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="group:claude-viewers@company.com" \
  --role="roles/aiplatform.viewer"

# Custom role for specific permissions
gcloud iam roles create claudeCodeUser \
  --project=$PROJECT_ID \
  --title="Claude Code User" \
  --description="Custom role for Claude Code users" \
  --permissions="aiplatform.endpoints.predict"

echo "Team access configured successfully"
```

#### Custom IAM Role Definition

```yaml
# claude-code-role.yaml
title: "Claude Code User"
description: "Minimal permissions for Claude Code"
stage: "GA"
includedPermissions:
  - aiplatform.endpoints.predict
  - aiplatform.models.list
  - aiplatform.models.get
  - logging.logEntries.list
```

```bash
# Create custom role from YAML
gcloud iam roles create claudeCodeUser \
  --project=$PROJECT_ID \
  --file=claude-code-role.yaml
```

### Example 6: Compliance Setup (HIPAA, SOC2)

**Scenario:** Healthcare organization requiring HIPAA compliance.

#### HIPAA-Compliant Configuration

```bash
#!/bin/bash
# hipaa-compliant-setup.sh

PROJECT_ID="healthcare-claude-prod"
REGION="us-central1"

# 1. Create project in compliant folder
gcloud projects create $PROJECT_ID \
  --folder=HIPAA_FOLDER_ID \
  --name="Claude HIPAA Production"

# 2. Enable required APIs
gcloud services enable \
  aiplatform.googleapis.com \
  cloudkms.googleapis.com \
  logging.googleapis.com \
  monitoring.googleapis.com \
  --project=$PROJECT_ID

# 3. Create CMEK key ring and key
gcloud kms keyrings create claude-keyring \
  --location=$REGION \
  --project=$PROJECT_ID

gcloud kms keys create claude-key \
  --location=$REGION \
  --keyring=claude-keyring \
  --purpose=encryption \
  --project=$PROJECT_ID

# 4. Enable audit logging
cat > audit-config.yaml << EOF
auditConfigs:
- service: aiplatform.googleapis.com
  auditLogConfigs:
  - logType: ADMIN_READ
  - logType: DATA_READ
  - logType: DATA_WRITE
EOF

gcloud projects set-iam-policy $PROJECT_ID audit-config.yaml

# 5. Create service account with minimal permissions
gcloud iam service-accounts create claude-hipaa-sa \
  --display-name="Claude HIPAA Service Account" \
  --project=$PROJECT_ID

gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:claude-hipaa-sa@${PROJECT_ID}.iam.gserviceaccount.com" \
  --role="roles/aiplatform.user"

# 6. Configure VPC Service Controls (requires organization admin)
echo "VPC Service Controls configuration requires organization admin access"
echo "Contact your security team to add this project to a service perimeter"
```

#### Data Residency Configuration

```bash
# Ensure data stays in US
gcloud resource-manager org-policies set-policy \
  --project=$PROJECT_ID \
  policy.yaml

# policy.yaml
# constraint: constraints/gcp.resourceLocations
# listPolicy:
#   allowedValues:
#     - us-central1
#     - us-east4
```

### Example 7: Cost Monitoring Configuration

**Scenario:** Set up comprehensive cost monitoring and alerts.

#### Budget Alert Setup

```bash
#!/bin/bash
# setup-cost-monitoring.sh

PROJECT_ID="your-project-id"
BILLING_ACCOUNT_ID="your-billing-account"

# Create budget using gcloud
gcloud billing budgets create \
  --billing-account=$BILLING_ACCOUNT_ID \
  --display-name="Claude Vertex AI Monthly Budget" \
  --budget-amount=1000USD \
  --threshold-rule=percent=50 \
  --threshold-rule=percent=90 \
  --threshold-rule=percent=100 \
  --notifications-rule-pubsub-topic="projects/$PROJECT_ID/topics/billing-alerts"

# Create Pub/Sub topic for alerts
gcloud pubsub topics create billing-alerts --project=$PROJECT_ID

# Create subscription for email alerts
gcloud pubsub subscriptions create billing-alerts-email \
  --topic=billing-alerts \
  --push-endpoint="https://your-alert-handler.com/webhook" \
  --project=$PROJECT_ID
```

#### Cloud Monitoring Dashboard

```json
{
  "displayName": "Claude Vertex AI Usage",
  "dashboardFilters": [],
  "mosaicLayout": {
    "columns": 12,
    "tiles": [
      {
        "width": 6,
        "height": 4,
        "widget": {
          "title": "API Request Count",
          "xyChart": {
            "dataSets": [{
              "timeSeriesQuery": {
                "timeSeriesFilter": {
                  "filter": "metric.type=\"aiplatform.googleapis.com/prediction/online/prediction_count\" resource.type=\"aiplatform.googleapis.com/Endpoint\"",
                  "aggregation": {
                    "alignmentPeriod": "60s",
                    "perSeriesAligner": "ALIGN_RATE"
                  }
                }
              }
            }]
          }
        }
      },
      {
        "xPos": 6,
        "width": 6,
        "height": 4,
        "widget": {
          "title": "Latency (p50, p95, p99)",
          "xyChart": {
            "dataSets": [{
              "timeSeriesQuery": {
                "timeSeriesFilter": {
                  "filter": "metric.type=\"aiplatform.googleapis.com/prediction/online/prediction_latencies\" resource.type=\"aiplatform.googleapis.com/Endpoint\"",
                  "aggregation": {
                    "alignmentPeriod": "60s",
                    "perSeriesAligner": "ALIGN_PERCENTILE_50"
                  }
                }
              }
            }]
          }
        }
      }
    ]
  }
}
```

#### Cost Analysis Query

```sql
-- BigQuery cost analysis for Claude usage
SELECT
  DATE(usage_start_time) as date,
  service.description as service,
  sku.description as sku,
  SUM(cost) as total_cost,
  SUM(usage.amount) as total_usage,
  usage.unit as unit
FROM `project.dataset.gcp_billing_export_v1_XXXXXX`
WHERE
  service.description = "Vertex AI"
  AND usage_start_time >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 30 DAY)
GROUP BY
  date, service, sku, unit
ORDER BY
  date DESC, total_cost DESC
```

---

## Available Models and Regions

### Available Claude Models on Vertex AI

| Model | Vertex AI Model ID | Context Window | Best For |
|-------|-------------------|----------------|----------|
| Claude 3.5 Sonnet | claude-3-5-sonnet@20240620 | 200K tokens | Balanced performance/cost |
| Claude 3 Opus | claude-3-opus@20240229 | 200K tokens | Complex reasoning tasks |
| Claude 3 Sonnet | claude-3-sonnet@20240229 | 200K tokens | General purpose |
| Claude 3 Haiku | claude-3-haiku@20240307 | 200K tokens | Fast, cost-effective |

### Model Selection Guide

```
┌─────────────────────────────────────────────────────────────┐
│                    Model Selection Guide                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Need highest quality reasoning?                            │
│  └── YES → Claude 3 Opus                                    │
│                                                             │
│  Need balance of quality and speed?                         │
│  └── YES → Claude 3.5 Sonnet (recommended default)         │
│                                                             │
│  Need fast responses for simple tasks?                      │
│  └── YES → Claude 3 Haiku                                  │
│                                                             │
│  Budget constrained with moderate complexity?               │
│  └── YES → Claude 3 Sonnet                                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Available Regions

| Region | Location | Typical Use Case |
|--------|----------|------------------|
| us-central1 | Iowa, USA | General US workloads |
| us-east4 | Northern Virginia, USA | East coast, government |
| europe-west1 | Belgium, Europe | EU data residency |
| asia-northeast1 | Tokyo, Japan | APAC workloads |

### Region Selection Considerations

```bash
# Check region availability
gcloud ai locations list

# Test latency to different regions
for region in us-central1 us-east4 europe-west1 asia-northeast1; do
  echo "Testing $region..."
  time curl -s "https://${region}-aiplatform.googleapis.com/health" > /dev/null
done
```

---

## Troubleshooting

### Error: "Permission Denied"

**Symptoms:**
```
Error: PERMISSION_DENIED: Request had insufficient authentication scopes
```

**Solutions:**

```bash
# 1. Verify service account has correct role
gcloud projects get-iam-policy PROJECT_ID \
  --flatten="bindings[].members" \
  --filter="bindings.members:serviceAccount:YOUR_SA" \
  --format="table(bindings.role)"

# 2. Grant missing role
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:YOUR_SA@PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/aiplatform.user"

# 3. Verify ADC has quota project set
gcloud auth application-default set-quota-project PROJECT_ID

# 4. Re-authenticate
gcloud auth application-default login
```

### Error: "API Not Enabled"

**Symptoms:**
```
Error: Vertex AI API has not been used in project PROJECT_ID before
```

**Solutions:**

```bash
# Enable the Vertex AI API
gcloud services enable aiplatform.googleapis.com --project=PROJECT_ID

# Verify API is enabled
gcloud services list --enabled --project=PROJECT_ID | grep aiplatform

# If Claude models aren't accessible, enable via Console:
# Vertex AI > Model Garden > Claude > Enable
```

### Error: "Region Not Available"

**Symptoms:**
```
Error: Location REGION is not available for this resource
```

**Solutions:**

```bash
# List available regions
gcloud ai locations list

# Use an available region
export GOOGLE_CLOUD_REGION=us-central1

# Check model availability in region
gcloud ai models list --region=us-central1
```

### Error: "Authentication Failed"

**Symptoms:**
```
Error: Could not automatically determine credentials
```

**Solutions:**

```bash
# Solution 1: Check GOOGLE_APPLICATION_CREDENTIALS
echo $GOOGLE_APPLICATION_CREDENTIALS
ls -la $GOOGLE_APPLICATION_CREDENTIALS

# Solution 2: Re-authenticate with ADC
gcloud auth application-default login

# Solution 3: Verify key file is valid
cat $GOOGLE_APPLICATION_CREDENTIALS | jq .type
# Should output: "service_account"

# Solution 4: Create new key if corrupted
gcloud iam service-accounts keys create ~/new-key.json \
  --iam-account=YOUR_SA@PROJECT_ID.iam.gserviceaccount.com
export GOOGLE_APPLICATION_CREDENTIALS=~/new-key.json
```

### Error: "Quota Exceeded"

**Symptoms:**
```
Error: RESOURCE_EXHAUSTED: Quota exceeded for aiplatform.googleapis.com
```

**Solutions:**

```bash
# 1. Check current quota usage
gcloud compute project-info describe --project=PROJECT_ID

# 2. Request quota increase via Console:
# IAM & Admin > Quotas > Filter for "Vertex AI"

# 3. Implement retry with exponential backoff
# (In your application code)

# 4. Use different region with available quota
export GOOGLE_CLOUD_REGION=us-east4
```

### Error: "Model Not Found"

**Symptoms:**
```
Error: Model claude-3-5-sonnet@20240620 not found
```

**Solutions:**

```bash
# 1. Verify model name is correct
# Check the model ID matches exactly

# 2. Ensure Claude models are enabled
# Visit: Vertex AI > Model Garden > Claude

# 3. Check model availability in your region
gcloud ai models list --region=us-central1

# 4. Try using full model path
claude --provider vertexai \
  --model "publishers/anthropic/models/claude-3-5-sonnet@20240620"
```

### Debugging Connection Issues

```bash
# Enable verbose logging
claude --provider vertexai --verbose "test"

# Check network connectivity
curl -v https://us-central1-aiplatform.googleapis.com/health

# Verify credentials
gcloud auth application-default print-access-token

# Test API directly
ACCESS_TOKEN=$(gcloud auth application-default print-access-token)
curl -X POST \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  "https://us-central1-aiplatform.googleapis.com/v1/projects/PROJECT_ID/locations/us-central1/publishers/anthropic/models/claude-3-sonnet@20240229:predict" \
  -d '{"instances":[{"messages":[{"role":"user","content":"Hello"}]}]}'
```

### Common Issues Checklist

```
□ API enabled?
  gcloud services list --enabled | grep aiplatform

□ Service account has correct role?
  gcloud projects get-iam-policy PROJECT_ID | grep -A2 aiplatform.user

□ Credentials configured?
  echo $GOOGLE_APPLICATION_CREDENTIALS

□ Project configured?
  echo $GOOGLE_CLOUD_PROJECT

□ Region configured?
  echo $GOOGLE_CLOUD_REGION

□ Claude models enabled in Model Garden?
  (Check console)

□ Network connectivity?
  curl https://us-central1-aiplatform.googleapis.com/health
```

---

## Quick Reference Cheat Sheet

### Essential Commands

```bash
# ============================================
# AUTHENTICATION
# ============================================

# Login interactively
gcloud auth login

# Set up ADC for applications
gcloud auth application-default login

# Set quota project for ADC
gcloud auth application-default set-quota-project PROJECT_ID

# Use service account key
export GOOGLE_APPLICATION_CREDENTIALS=~/path/to/key.json

# ============================================
# PROJECT SETUP
# ============================================

# Create new project
gcloud projects create PROJECT_ID --name="Project Name"

# Set current project
gcloud config set project PROJECT_ID

# Enable Vertex AI API
gcloud services enable aiplatform.googleapis.com

# Link billing
gcloud billing projects link PROJECT_ID --billing-account=BILLING_ID

# ============================================
# SERVICE ACCOUNT
# ============================================

# Create service account
gcloud iam service-accounts create SA_NAME \
  --display-name="Display Name"

# Grant Vertex AI access
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:SA_NAME@PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/aiplatform.user"

# Create key file
gcloud iam service-accounts keys create ~/key.json \
  --iam-account=SA_NAME@PROJECT_ID.iam.gserviceaccount.com

# ============================================
# CLAUDE CODE USAGE
# ============================================

# Basic Vertex AI usage
claude --provider vertexai "Your prompt"

# With specific model
claude --provider vertexai --model claude-3-5-sonnet@20240620 "Prompt"

# With region
claude --provider vertexai --region us-central1 "Prompt"

# Non-interactive mode
claude --provider vertexai --print "Prompt" > output.txt

# ============================================
# ENVIRONMENT VARIABLES
# ============================================

export GOOGLE_CLOUD_PROJECT=your-project-id
export GOOGLE_CLOUD_REGION=us-central1
export GOOGLE_APPLICATION_CREDENTIALS=~/claude-vertex-key.json

# ============================================
# WORKLOAD IDENTITY
# ============================================

# Create pool
gcloud iam workload-identity-pools create POOL_NAME \
  --location="global"

# Create OIDC provider
gcloud iam workload-identity-pools providers create-oidc PROVIDER_NAME \
  --location="global" \
  --workload-identity-pool="POOL_NAME" \
  --issuer-uri="https://token.actions.githubusercontent.com" \
  --attribute-mapping="google.subject=assertion.sub"

# Grant impersonation
gcloud iam service-accounts add-iam-policy-binding SA_EMAIL \
  --role="roles/iam.workloadIdentityUser" \
  --member="principalSet://iam.googleapis.com/projects/NUM/locations/global/workloadIdentityPools/POOL/attribute.repository/ORG/REPO"
```

### Model Quick Reference

```
┌──────────────────────────────────────────────────────────────┐
│ Model                        │ ID                            │
├──────────────────────────────┼───────────────────────────────┤
│ Claude 3.5 Sonnet            │ claude-3-5-sonnet@20240620    │
│ Claude 3 Opus                │ claude-3-opus@20240229        │
│ Claude 3 Sonnet              │ claude-3-sonnet@20240229      │
│ Claude 3 Haiku               │ claude-3-haiku@20240307       │
└──────────────────────────────┴───────────────────────────────┘
```

### Region Quick Reference

```
┌─────────────────┬────────────────────────┐
│ Region          │ Location               │
├─────────────────┼────────────────────────┤
│ us-central1     │ Iowa, USA              │
│ us-east4        │ N. Virginia, USA       │
│ europe-west1    │ Belgium                │
│ asia-northeast1 │ Tokyo, Japan           │
└─────────────────┴────────────────────────┘
```

### settings.json Template

```json
{
  "provider": "vertexai",
  "model": "claude-3-5-sonnet@20240620",
  "vertexai": {
    "project": "your-project-id",
    "region": "us-central1"
  }
}
```

### GitHub Actions Template

```yaml
- uses: google-github-actions/auth@v2
  with:
    workload_identity_provider: 'projects/PROJECT_NUM/locations/global/workloadIdentityPools/POOL/providers/PROVIDER'
    service_account: 'SA@PROJECT.iam.gserviceaccount.com'

- name: Run Claude
  env:
    GOOGLE_CLOUD_PROJECT: project-id
    GOOGLE_CLOUD_REGION: us-central1
  run: claude --provider vertexai "prompt"
```

---

## Best Practices

### Security Best Practices

#### 1. Use Workload Identity Over Service Account Keys

```bash
# PREFERRED: Workload Identity (no keys)
# Set up federation and use short-lived tokens

# AVOID: Long-lived service account keys
# Only use when Workload Identity is not possible
```

#### 2. Implement Least Privilege

```bash
# Create custom role with minimal permissions
gcloud iam roles create minimalClaudeUser \
  --project=PROJECT_ID \
  --title="Minimal Claude User" \
  --permissions="aiplatform.endpoints.predict"

# Use custom role instead of broad roles
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:SA_EMAIL" \
  --role="projects/PROJECT_ID/roles/minimalClaudeUser"
```

#### 3. Regular Key Rotation

```bash
# If using keys, rotate every 90 days
# Script for automated rotation
#!/bin/bash
SA_EMAIL="sa@project.iam.gserviceaccount.com"

# Create new key
gcloud iam service-accounts keys create new-key.json \
  --iam-account=$SA_EMAIL

# Test new key
# ... validation ...

# Delete old key
OLD_KEY=$(gcloud iam service-accounts keys list \
  --iam-account=$SA_EMAIL --format="value(name)" | tail -1)
gcloud iam service-accounts keys delete $OLD_KEY \
  --iam-account=$SA_EMAIL --quiet
```

### Operational Best Practices

#### 1. Monitor with Cloud Monitoring

```bash
# Create alerting policy for errors
gcloud monitoring policies create \
  --display-name="Claude API Errors" \
  --condition-filter='resource.type="aiplatform.googleapis.com/Endpoint" AND metric.type="aiplatform.googleapis.com/prediction/online/error_count"' \
  --condition-threshold-value=10 \
  --notification-channels=CHANNEL_ID
```

#### 2. Set Up Budget Alerts

```bash
# Create budget with alerts at 50%, 90%, 100%
gcloud billing budgets create \
  --billing-account=BILLING_ACCOUNT \
  --display-name="Claude Monthly Budget" \
  --budget-amount=1000USD \
  --threshold-rule=percent=50 \
  --threshold-rule=percent=90 \
  --threshold-rule=percent=100
```

#### 3. Implement Retry Logic

```python
# Example retry logic for API calls
import time
from google.api_core import retry

@retry.Retry(
    initial=1.0,
    maximum=60.0,
    multiplier=2.0,
    predicate=retry.if_transient_error
)
def call_claude_with_retry(prompt):
    # Your Claude API call here
    pass
```

#### 4. Use Appropriate Models

```
Task Complexity → Model Selection

Simple (classification, extraction)  → Claude 3 Haiku
Medium (summarization, Q&A)         → Claude 3 Sonnet / 3.5 Sonnet
Complex (analysis, reasoning)        → Claude 3 Opus
```

### Development Best Practices

#### 1. Environment-Specific Configuration

```bash
# Development
export GOOGLE_CLOUD_PROJECT=my-project-dev
export GOOGLE_CLOUD_REGION=us-central1

# Staging
export GOOGLE_CLOUD_PROJECT=my-project-staging
export GOOGLE_CLOUD_REGION=us-central1

# Production
export GOOGLE_CLOUD_PROJECT=my-project-prod
export GOOGLE_CLOUD_REGION=us-central1
```

#### 2. Use Project-Specific Settings

```json
// .claude-code.json in project root
{
  "provider": "vertexai",
  "model": "claude-3-sonnet@20240229",
  "vertexai": {
    "project": "${GOOGLE_CLOUD_PROJECT}",
    "region": "${GOOGLE_CLOUD_REGION}"
  }
}
```

#### 3. Implement Logging

```bash
# Enable Cloud Audit Logs
gcloud logging write claude-code-log \
  "Claude Code invocation" \
  --severity=INFO \
  --payload-type=json \
  --payload='{"action":"predict","model":"claude-3-sonnet"}'

# Query logs
gcloud logging read \
  'resource.type="aiplatform.googleapis.com/Endpoint"' \
  --limit=10
```

### Compliance Best Practices

#### 1. Enable Audit Logging

```yaml
# audit-config.yaml
auditConfigs:
- service: aiplatform.googleapis.com
  auditLogConfigs:
  - logType: ADMIN_READ
  - logType: DATA_READ
  - logType: DATA_WRITE
```

#### 2. Implement Data Residency

```bash
# Restrict resources to specific regions
gcloud resource-manager org-policies set-policy \
  --project=PROJECT_ID \
  --policy-file=location-policy.yaml

# location-policy.yaml
# constraint: constraints/gcp.resourceLocations
# listPolicy:
#   allowedValues:
#     - us-central1
#     - us-east4
```

#### 3. Use VPC Service Controls

```bash
# Create service perimeter (requires org admin)
gcloud access-context-manager perimeters create claude-perimeter \
  --title="Claude Service Perimeter" \
  --resources="projects/PROJECT_NUM" \
  --restricted-services="aiplatform.googleapis.com" \
  --policy=POLICY_ID
```

---

## Summary

This tutorial covered comprehensive Google Vertex AI integration with Claude Code:

1. **Setup**: Project creation, API enablement, service account configuration
2. **Authentication**: ADC, Workload Identity Federation, service account keys
3. **Configuration**: Environment variables, CLI flags, settings files
4. **Enterprise Features**: Compliance, multi-region, team access
5. **Best Practices**: Security, operations, development, compliance

### Next Steps

1. **Start Simple**: Use ADC for local development
2. **Add CI/CD**: Implement Workload Identity for GitHub Actions
3. **Scale Up**: Configure multi-region deployment
4. **Secure**: Implement VPC Service Controls for production
5. **Monitor**: Set up Cloud Monitoring and budget alerts

### Additional Resources

- [Vertex AI Documentation](https://cloud.google.com/vertex-ai/docs)
- [Claude on Vertex AI](https://cloud.google.com/vertex-ai/docs/generative-ai/partner-models/claude)
- [Workload Identity Federation](https://cloud.google.com/iam/docs/workload-identity-federation)
- [IAM Best Practices](https://cloud.google.com/iam/docs/using-iam-securely)
- [VPC Service Controls](https://cloud.google.com/vpc-service-controls/docs)

---

*Tutorial Version: 1.0*
*Last Updated: 2024*
*Compatibility: Claude Code v1.x, Google Cloud SDK 450+*
