# Tutorial 23: Amazon Bedrock Integration with Claude Code

## A Comprehensive Guide to Running Claude Code with AWS Bedrock

---

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Core Concepts](#core-concepts)
4. [Understanding Amazon Bedrock](#understanding-amazon-bedrock)
5. [IAM Setup](#iam-setup)
6. [Configuring Claude Code for Bedrock](#configuring-claude-code-for-bedrock)
7. [Inference Profiles](#inference-profiles)
8. [Real-World Examples](#real-world-examples)
9. [Model Reference](#model-reference)
10. [Cost Analysis](#cost-analysis)
11. [Troubleshooting Guide](#troubleshooting-guide)
12. [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)
13. [Best Practices](#best-practices)
14. [Summary](#summary)

---

## Overview

### What is Amazon Bedrock?

Amazon Bedrock is a fully managed service from Amazon Web Services (AWS) that provides
access to foundation models (FMs) from leading AI companies, including Anthropic's Claude
models. It enables organizations to build and scale generative AI applications without
managing the underlying infrastructure.

Think of Bedrock as a secure, enterprise-grade gateway to powerful AI models. Instead of
calling Anthropic's API directly, you route your requests through AWS infrastructure,
gaining access to AWS's security, compliance, and billing features.

### Why Use Claude Code with Bedrock?

There are several compelling reasons to use Claude Code with Amazon Bedrock:

**Enterprise Integration**
- Seamless integration with existing AWS infrastructure
- Consolidated billing through AWS accounts
- Integration with AWS Identity and Access Management (IAM)
- VPC endpoints for private network access

**Security and Compliance**
- Data never leaves your AWS environment
- SOC 2, HIPAA, and other compliance certifications
- AWS CloudTrail for audit logging
- AWS KMS for encryption key management

**Operational Benefits**
- High availability across multiple AWS regions
- Automatic scaling and load balancing
- AWS CloudWatch monitoring and alerting
- Service quotas and rate limiting controls

**Cost Management**
- Volume discounts for high-usage organizations
- AWS Savings Plans compatibility
- Detailed cost allocation with tags
- AWS Budgets for spending controls

### Enterprise Benefits Deep Dive

Organizations choosing Bedrock for Claude Code access enjoy numerous advantages:

```
+------------------------------------------------------------------+
|                    ENTERPRISE BENEFITS                            |
+------------------------------------------------------------------+
|                                                                   |
|  SECURITY           COMPLIANCE          OPERATIONS               |
|  --------           ----------          ----------               |
|  - IAM Roles        - SOC 2             - Multi-AZ               |
|  - VPC Endpoints    - HIPAA             - Auto-scaling           |
|  - KMS Encryption   - GDPR              - CloudWatch             |
|  - CloudTrail       - PCI DSS           - X-Ray tracing          |
|                                                                   |
|  COST MANAGEMENT    INTEGRATION         GOVERNANCE               |
|  ---------------    -----------         ----------               |
|  - Volume Pricing   - EventBridge       - Service Control        |
|  - Savings Plans    - Step Functions    - Resource Tags          |
|  - Cost Explorer    - Lambda            - Organizations          |
|  - Budgets          - SageMaker         - Config Rules           |
|                                                                   |
+------------------------------------------------------------------+
```

---

## Prerequisites

Before integrating Claude Code with Amazon Bedrock, ensure you have the following:

### AWS Account Requirements

1. **Active AWS Account**
   - A valid AWS account with billing enabled
   - Account must be in good standing (no payment issues)
   - Account email must be verified

2. **Bedrock Service Access**
   - Bedrock must be enabled in your AWS region
   - Model access must be requested and approved for Claude models
   - Note: Model access approval can take up to 48 hours

3. **Administrative Access (Initial Setup)**
   - IAM administrator access for creating policies and roles
   - Permission to create OIDC providers (if using federated auth)

### Requesting Claude Model Access on Bedrock

Before using Claude models, you must request access:

```
Step-by-step process:

1. Sign in to AWS Console
2. Navigate to Amazon Bedrock
3. Click "Model access" in the left sidebar
4. Click "Manage model access"
5. Find "Anthropic" in the providers list
6. Select the Claude models you need:
   - Claude 3.5 Sonnet
   - Claude 3 Opus
   - Claude 3 Sonnet
   - Claude 3 Haiku
7. Click "Request model access"
8. Accept the End User License Agreement (EULA)
9. Submit the request
10. Wait for approval (usually immediate, up to 48 hours)
```

### Claude Code Installation

Ensure Claude Code is properly installed:

```bash
# Verify Claude Code installation
claude --version

# Expected output (version may vary)
# Claude Code v1.x.x

# If not installed, install via npm
npm install -g @anthropic-ai/claude-code

# Or via Homebrew (macOS)
brew install claude-code
```

### AWS CLI Installation

The AWS CLI is helpful for authentication and testing:

```bash
# Install AWS CLI v2

# macOS
brew install awscli

# Windows (PowerShell as Administrator)
msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi

# Linux
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Verify installation
aws --version
# Expected: aws-cli/2.x.x Python/3.x.x ...
```

### Environment Checklist

Use this checklist to verify your environment is ready:

```
[ ] AWS account active and in good standing
[ ] Bedrock enabled in desired region(s)
[ ] Claude model access approved
[ ] Claude Code installed and working
[ ] AWS CLI installed (optional but recommended)
[ ] IAM permissions configured (covered in next section)
[ ] AWS credentials available
```

---

## Core Concepts

### Understanding Key Terminology

Before diving into configuration, let's understand the fundamental concepts:

#### Inference Profiles

An inference profile is a configuration that determines how your requests are routed
to Claude models on Bedrock. There are two types:

1. **System-defined inference profiles**: Automatically route requests across regions
2. **Application inference profiles**: Custom configurations you create

```
+-------------------------------------------------------------------+
|                    INFERENCE PROFILE CONCEPT                       |
+-------------------------------------------------------------------+
|                                                                    |
|   Your Request                                                     |
|       |                                                            |
|       v                                                            |
|   +------------------+                                             |
|   | Inference Profile|                                             |
|   +------------------+                                             |
|       |                                                            |
|       +------------+------------+                                  |
|       |            |            |                                  |
|       v            v            v                                  |
|   us-east-1    us-west-2    eu-west-1                             |
|   (Primary)    (Fallback)   (Fallback)                            |
|                                                                    |
+-------------------------------------------------------------------+
```

#### Regions

Amazon Bedrock with Claude is available in multiple AWS regions:

| Region Code    | Region Name           | Claude 3.5 | Claude 3 Opus | Claude 3 Sonnet |
|---------------|-----------------------|------------|---------------|-----------------|
| us-east-1     | US East (N. Virginia) | Yes        | Yes           | Yes             |
| us-west-2     | US West (Oregon)      | Yes        | Yes           | Yes             |
| eu-west-1     | Europe (Ireland)      | Yes        | No            | Yes             |
| eu-central-1  | Europe (Frankfurt)    | Yes        | No            | Yes             |
| ap-northeast-1| Asia Pacific (Tokyo)  | Yes        | No            | Yes             |
| ap-southeast-1| Asia Pacific (Singapore)| Yes      | No            | Yes             |

*Note: Availability changes over time. Check AWS documentation for current status.*

#### Authentication Methods

AWS supports multiple authentication methods:

```
+-------------------------------------------------------------------+
|                    AUTHENTICATION METHODS                          |
+-------------------------------------------------------------------+
|                                                                    |
|  1. Long-term Credentials                                         |
|     - Access Key ID + Secret Access Key                           |
|     - Stored in ~/.aws/credentials                                |
|     - Suitable for: Development, testing                          |
|                                                                    |
|  2. Short-term Credentials                                        |
|     - Access Key ID + Secret Access Key + Session Token           |
|     - Temporary (1-12 hours)                                      |
|     - Suitable for: Production, CI/CD                             |
|                                                                    |
|  3. IAM Roles                                                     |
|     - Attached to AWS resources (EC2, Lambda, etc.)               |
|     - No explicit credentials needed                              |
|     - Suitable for: AWS-hosted applications                       |
|                                                                    |
|  4. OIDC Federation                                               |
|     - External identity provider (GitHub, Okta, etc.)             |
|     - Web identity tokens                                         |
|     - Suitable for: CI/CD, external applications                  |
|                                                                    |
|  5. AWS SSO / IAM Identity Center                                 |
|     - Centralized identity management                             |
|     - Session-based authentication                                |
|     - Suitable for: Enterprise, multi-account                     |
|                                                                    |
+-------------------------------------------------------------------+
```

### The Request Flow

Understanding how requests flow from Claude Code to Bedrock:

```
+-------------------------------------------------------------------+
|                       REQUEST FLOW                                 |
+-------------------------------------------------------------------+
|                                                                    |
|   Claude Code CLI                                                  |
|        |                                                           |
|        | 1. User issues command                                   |
|        v                                                           |
|   +------------------+                                             |
|   | AWS Credentials  |  2. Load credentials from:                 |
|   | Resolution       |     - Environment variables                |
|   +------------------+     - AWS profile                          |
|        |                   - IAM role                             |
|        | 3. Sign request with AWS Signature V4                    |
|        v                                                           |
|   +------------------+                                             |
|   | AWS Bedrock      |  4. Route to appropriate region            |
|   | Endpoint         |                                             |
|   +------------------+                                             |
|        |                                                           |
|        | 5. Invoke Claude model                                   |
|        v                                                           |
|   +------------------+                                             |
|   | Claude Model     |  6. Generate response                      |
|   | (on Bedrock)     |                                             |
|   +------------------+                                             |
|        |                                                           |
|        | 7. Stream response back                                  |
|        v                                                           |
|   Claude Code CLI (displays response)                              |
|                                                                    |
+-------------------------------------------------------------------+
```

---

## Understanding Amazon Bedrock

### Bedrock Service Overview

Amazon Bedrock is more than just an API gateway. It provides a comprehensive platform
for building AI applications:

**Core Components:**

```
+-------------------------------------------------------------------+
|                    AMAZON BEDROCK ARCHITECTURE                     |
+-------------------------------------------------------------------+
|                                                                    |
|  +-------------+  +-------------+  +-------------+                 |
|  | Foundation  |  | Model       |  | Guardrails  |                 |
|  | Models      |  | Customization|  | & Safety   |                 |
|  +-------------+  +-------------+  +-------------+                 |
|        |               |                |                          |
|        v               v                v                          |
|  +--------------------------------------------------+             |
|  |              BEDROCK RUNTIME                      |             |
|  |  - InvokeModel                                    |             |
|  |  - InvokeModelWithResponseStream                  |             |
|  |  - Converse API                                   |             |
|  +--------------------------------------------------+             |
|        |                                                           |
|        v                                                           |
|  +--------------------------------------------------+             |
|  |              INFRASTRUCTURE                       |             |
|  |  - Multi-AZ deployment                            |             |
|  |  - Auto-scaling                                   |             |
|  |  - VPC endpoints                                  |             |
|  +--------------------------------------------------+             |
|                                                                    |
+-------------------------------------------------------------------+
```

### Claude Models on Bedrock

Anthropic provides several Claude models through Bedrock:

#### Claude 3.5 Sonnet

The most capable model for complex tasks:

```
Model: Claude 3.5 Sonnet
Bedrock ID: anthropic.claude-3-5-sonnet-20240620-v1:0
Max Tokens: 200,000 input / 4,096 output
Best For: Complex reasoning, coding, analysis
Cost: Higher tier
```

#### Claude 3 Opus

The most powerful Claude 3 model:

```
Model: Claude 3 Opus
Bedrock ID: anthropic.claude-3-opus-20240229-v1:0
Max Tokens: 200,000 input / 4,096 output
Best For: Highly complex tasks, research, nuanced content
Cost: Highest tier
Availability: Limited regions
```

#### Claude 3 Sonnet

Balanced performance and cost:

```
Model: Claude 3 Sonnet
Bedrock ID: anthropic.claude-3-sonnet-20240229-v1:0
Max Tokens: 200,000 input / 4,096 output
Best For: General purpose, coding assistance
Cost: Medium tier
```

#### Claude 3 Haiku

Fast and cost-effective:

```
Model: Claude 3 Haiku
Bedrock ID: anthropic.claude-3-haiku-20240307-v1:0
Max Tokens: 200,000 input / 4,096 output
Best For: Quick tasks, high volume, simple queries
Cost: Lowest tier
```

### Pricing Comparison: Direct API vs Bedrock

Understanding when to use each option:

#### Direct Anthropic API Pricing

```
+-------------------------------------------------------------------+
|              ANTHROPIC API PRICING (Direct)                        |
+-------------------------------------------------------------------+
|                                                                    |
|  Model               | Input (per 1M tokens) | Output (per 1M)    |
|  --------------------|----------------------|---------------------|
|  Claude 3.5 Sonnet   | $3.00                | $15.00              |
|  Claude 3 Opus       | $15.00               | $75.00              |
|  Claude 3 Sonnet     | $3.00                | $15.00              |
|  Claude 3 Haiku      | $0.25                | $1.25               |
|                                                                    |
|  * Pay-as-you-go pricing                                          |
|  * No minimum commitment                                          |
|  * No volume discounts (standard tier)                            |
|                                                                    |
+-------------------------------------------------------------------+
```

#### Amazon Bedrock Pricing

```
+-------------------------------------------------------------------+
|              AMAZON BEDROCK PRICING                                |
+-------------------------------------------------------------------+
|                                                                    |
|  Model               | Input (per 1M tokens) | Output (per 1M)    |
|  --------------------|----------------------|---------------------|
|  Claude 3.5 Sonnet   | $3.00                | $15.00              |
|  Claude 3 Opus       | $15.00               | $75.00              |
|  Claude 3 Sonnet     | $3.00                | $15.00              |
|  Claude 3 Haiku      | $0.25                | $1.25               |
|                                                                    |
|  ADDITIONAL OPTIONS:                                               |
|  * Provisioned Throughput: Reserved capacity, lower per-token     |
|  * Batch Inference: Up to 50% discount for non-time-sensitive     |
|  * Savings Plans: Commit to usage for discounts                   |
|                                                                    |
+-------------------------------------------------------------------+
```

#### Cost Comparison Matrix

```
+-------------------------------------------------------------------+
|              WHEN TO USE EACH OPTION                               |
+-------------------------------------------------------------------+
|                                                                    |
|  SCENARIO                      | RECOMMENDED    | REASON           |
|  ------------------------------|----------------|------------------|
|  Individual developer          | Direct API     | Simpler setup    |
|  Small startup (<$1K/mo)       | Direct API     | No AWS overhead  |
|  Enterprise ($10K+/mo)         | Bedrock        | Volume discounts |
|  Regulated industry            | Bedrock        | Compliance       |
|  AWS-heavy infrastructure      | Bedrock        | Integration      |
|  Multi-cloud strategy          | Direct API     | Flexibility      |
|  CI/CD pipelines in AWS        | Bedrock        | IAM integration  |
|  High-volume batch processing  | Bedrock        | Batch pricing    |
|                                                                    |
+-------------------------------------------------------------------+
```

### Enterprise Benefits in Detail

#### Security Features

```
+-------------------------------------------------------------------+
|              BEDROCK SECURITY FEATURES                             |
+-------------------------------------------------------------------+
|                                                                    |
|  ENCRYPTION:                                                       |
|  - TLS 1.2+ in transit                                            |
|  - AES-256 at rest                                                |
|  - Customer-managed KMS keys                                       |
|                                                                    |
|  NETWORK ISOLATION:                                                |
|  - VPC endpoints (PrivateLink)                                    |
|  - No public internet required                                    |
|  - Security groups for access control                             |
|                                                                    |
|  ACCESS CONTROL:                                                   |
|  - IAM policies with conditions                                   |
|  - Resource-based policies                                        |
|  - Attribute-based access control (ABAC)                          |
|                                                                    |
|  AUDIT & MONITORING:                                               |
|  - CloudTrail for API logging                                     |
|  - CloudWatch for metrics                                         |
|  - AWS Config for compliance                                      |
|                                                                    |
+-------------------------------------------------------------------+
```

#### Compliance Certifications

```
Amazon Bedrock Compliance Status:

[x] SOC 1, SOC 2, SOC 3
[x] ISO 27001, ISO 27017, ISO 27018
[x] PCI DSS Level 1
[x] HIPAA Eligible
[x] FedRAMP High (in AWS GovCloud)
[x] GDPR
[x] CCPA
[x] IRAP (Australia)
[x] K-ISMS (Korea)
[x] MTCS (Singapore)
```

---

## IAM Setup

### Required Permissions

To use Claude Code with Bedrock, you need specific IAM permissions:

#### Minimum Required Actions

```
bedrock:InvokeModel
  - Required for synchronous model invocation
  - Used for standard Claude Code operations

bedrock:InvokeModelWithResponseStream
  - Required for streaming responses
  - Essential for real-time output display
```

#### Recommended Additional Actions

```
bedrock:ListFoundationModels
  - List available models
  - Useful for verification

bedrock:GetFoundationModel
  - Get model details
  - Helpful for debugging

bedrock:ListInferenceProfiles
  - List available inference profiles
  - Required for cross-region inference

bedrock:GetInferenceProfile
  - Get profile configuration details
```

### Creating the IAM Policy

#### Basic Policy (Minimum Required)

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "BedrockClaudeInvoke",
            "Effect": "Allow",
            "Action": [
                "bedrock:InvokeModel",
                "bedrock:InvokeModelWithResponseStream"
            ],
            "Resource": [
                "arn:aws:bedrock:*::foundation-model/anthropic.claude-*"
            ]
        }
    ]
}
```

#### Standard Policy (Recommended)

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "BedrockClaudeInvoke",
            "Effect": "Allow",
            "Action": [
                "bedrock:InvokeModel",
                "bedrock:InvokeModelWithResponseStream"
            ],
            "Resource": [
                "arn:aws:bedrock:*::foundation-model/anthropic.claude-*"
            ]
        },
        {
            "Sid": "BedrockModelDiscovery",
            "Effect": "Allow",
            "Action": [
                "bedrock:ListFoundationModels",
                "bedrock:GetFoundationModel"
            ],
            "Resource": "*"
        },
        {
            "Sid": "BedrockInferenceProfiles",
            "Effect": "Allow",
            "Action": [
                "bedrock:ListInferenceProfiles",
                "bedrock:GetInferenceProfile"
            ],
            "Resource": "*"
        }
    ]
}
```

#### Restrictive Policy (Production)

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "BedrockClaudeInvokeRestricted",
            "Effect": "Allow",
            "Action": [
                "bedrock:InvokeModel",
                "bedrock:InvokeModelWithResponseStream"
            ],
            "Resource": [
                "arn:aws:bedrock:us-east-1::foundation-model/anthropic.claude-3-sonnet-*",
                "arn:aws:bedrock:us-east-1::foundation-model/anthropic.claude-3-haiku-*"
            ],
            "Condition": {
                "StringEquals": {
                    "aws:RequestedRegion": "us-east-1"
                },
                "IpAddress": {
                    "aws:SourceIp": [
                        "10.0.0.0/8",
                        "192.168.1.0/24"
                    ]
                }
            }
        }
    ]
}
```

### Creating the Policy via AWS Console

```
Step-by-step instructions:

1. Sign in to the AWS Management Console
2. Navigate to IAM (Identity and Access Management)
3. Click "Policies" in the left sidebar
4. Click "Create policy"
5. Select the "JSON" tab
6. Paste your policy document
7. Click "Next: Tags" (optional: add tags)
8. Click "Next: Review"
9. Enter a policy name: "ClaudeCodeBedrockAccess"
10. Enter a description: "Allows Claude Code to invoke Claude models on Bedrock"
11. Click "Create policy"
```

### Creating the Policy via AWS CLI

```bash
# Create a file with the policy document
cat > claude-bedrock-policy.json << 'EOF'
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "BedrockClaudeInvoke",
            "Effect": "Allow",
            "Action": [
                "bedrock:InvokeModel",
                "bedrock:InvokeModelWithResponseStream"
            ],
            "Resource": [
                "arn:aws:bedrock:*::foundation-model/anthropic.claude-*"
            ]
        },
        {
            "Sid": "BedrockModelDiscovery",
            "Effect": "Allow",
            "Action": [
                "bedrock:ListFoundationModels",
                "bedrock:GetFoundationModel"
            ],
            "Resource": "*"
        }
    ]
}
EOF

# Create the policy
aws iam create-policy \
    --policy-name ClaudeCodeBedrockAccess \
    --policy-document file://claude-bedrock-policy.json \
    --description "Allows Claude Code to invoke Claude models on Bedrock"

# Note the policy ARN from the output
# Example: arn:aws:iam::123456789012:policy/ClaudeCodeBedrockAccess
```

### Role Setup

#### Creating an IAM Role for Users

```bash
# Trust policy for IAM users
cat > trust-policy-user.json << 'EOF'
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::ACCOUNT_ID:root"
            },
            "Action": "sts:AssumeRole",
            "Condition": {
                "StringEquals": {
                    "sts:ExternalId": "claude-code-external-id"
                }
            }
        }
    ]
}
EOF

# Create the role
aws iam create-role \
    --role-name ClaudeCodeBedrockRole \
    --assume-role-policy-document file://trust-policy-user.json \
    --description "Role for Claude Code Bedrock access"

# Attach the policy
aws iam attach-role-policy \
    --role-name ClaudeCodeBedrockRole \
    --policy-arn arn:aws:iam::ACCOUNT_ID:policy/ClaudeCodeBedrockAccess
```

#### Creating an IAM Role for EC2/Lambda

```bash
# Trust policy for AWS services
cat > trust-policy-service.json << 'EOF'
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": [
                    "ec2.amazonaws.com",
                    "lambda.amazonaws.com"
                ]
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
EOF

# Create the role
aws iam create-role \
    --role-name ClaudeCodeBedrockServiceRole \
    --assume-role-policy-document file://trust-policy-service.json \
    --description "Service role for Claude Code on Bedrock"

# Attach the policy
aws iam attach-role-policy \
    --role-name ClaudeCodeBedrockServiceRole \
    --policy-arn arn:aws:iam::ACCOUNT_ID:policy/ClaudeCodeBedrockAccess
```

### OIDC Authentication

OIDC (OpenID Connect) authentication enables secure access from external identity
providers without storing long-term AWS credentials.

#### Setting Up GitHub Actions OIDC Provider

```bash
# Step 1: Create the OIDC provider
aws iam create-open-id-connect-provider \
    --url https://token.actions.githubusercontent.com \
    --client-id-list sts.amazonaws.com \
    --thumbprint-list 6938fd4d98bab03faadb97b34396831e3780aea1

# Step 2: Create trust policy for GitHub Actions
cat > github-oidc-trust.json << 'EOF'
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "arn:aws:iam::ACCOUNT_ID:oidc-provider/token.actions.githubusercontent.com"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringEquals": {
                    "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
                },
                "StringLike": {
                    "token.actions.githubusercontent.com:sub": "repo:YOUR_ORG/YOUR_REPO:*"
                }
            }
        }
    ]
}
EOF

# Step 3: Create the role
aws iam create-role \
    --role-name ClaudeCodeGitHubActionsRole \
    --assume-role-policy-document file://github-oidc-trust.json \
    --description "Role for GitHub Actions to use Claude Code with Bedrock"

# Step 4: Attach the Bedrock policy
aws iam attach-role-policy \
    --role-name ClaudeCodeGitHubActionsRole \
    --policy-arn arn:aws:iam::ACCOUNT_ID:policy/ClaudeCodeBedrockAccess
```

#### OIDC Trust Policy Template

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "arn:aws:iam::ACCOUNT_ID:oidc-provider/token.actions.githubusercontent.com"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringEquals": {
                    "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
                },
                "StringLike": {
                    "token.actions.githubusercontent.com:sub": "repo:YOUR_ORGANIZATION/*:*"
                }
            }
        }
    ]
}
```

#### Setting Up Okta OIDC Provider

```bash
# Step 1: Create the OIDC provider for Okta
aws iam create-open-id-connect-provider \
    --url https://YOUR_OKTA_DOMAIN.okta.com \
    --client-id-list YOUR_OKTA_CLIENT_ID \
    --thumbprint-list YOUR_OKTA_THUMBPRINT

# Step 2: Create trust policy
cat > okta-oidc-trust.json << 'EOF'
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "arn:aws:iam::ACCOUNT_ID:oidc-provider/YOUR_OKTA_DOMAIN.okta.com"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringEquals": {
                    "YOUR_OKTA_DOMAIN.okta.com:aud": "YOUR_OKTA_CLIENT_ID"
                }
            }
        }
    ]
}
EOF

# Step 3: Create and configure the role
aws iam create-role \
    --role-name ClaudeCodeOktaRole \
    --assume-role-policy-document file://okta-oidc-trust.json

aws iam attach-role-policy \
    --role-name ClaudeCodeOktaRole \
    --policy-arn arn:aws:iam::ACCOUNT_ID:policy/ClaudeCodeBedrockAccess
```

---

## Configuring Claude Code for Bedrock

### Environment Variables

Claude Code uses standard AWS environment variables for authentication:

#### Required Environment Variables

```bash
# Template: Environment setup for Bedrock
export AWS_REGION=us-east-1
export AWS_ACCESS_KEY_ID=your-access-key
export AWS_SECRET_ACCESS_KEY=your-secret-key

# Or use AWS profile
export AWS_PROFILE=claude-bedrock
```

#### All Supported Environment Variables

```bash
# Region Configuration
export AWS_REGION=us-east-1
export AWS_DEFAULT_REGION=us-east-1  # Fallback if AWS_REGION not set

# Long-term Credentials
export AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
export AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY

# Session Credentials (Temporary)
export AWS_SESSION_TOKEN=FwoGZXIvYXdzEBY...

# Profile Selection
export AWS_PROFILE=claude-bedrock

# Assume Role (Alternative)
export AWS_ROLE_ARN=arn:aws:iam::123456789012:role/ClaudeCodeRole
export AWS_WEB_IDENTITY_TOKEN_FILE=/var/run/secrets/token

# Endpoint Customization (Advanced)
export AWS_ENDPOINT_URL_BEDROCK_RUNTIME=https://bedrock-runtime.us-east-1.amazonaws.com
```

#### Setting Up Environment for Different Scenarios

**Development Environment (macOS/Linux):**

```bash
# Add to ~/.bashrc or ~/.zshrc
export AWS_REGION=us-east-1
export AWS_PROFILE=claude-bedrock-dev

# Reload shell configuration
source ~/.bashrc  # or source ~/.zshrc
```

**Development Environment (Windows PowerShell):**

```powershell
# Temporary (current session only)
$env:AWS_REGION = "us-east-1"
$env:AWS_PROFILE = "claude-bedrock-dev"

# Permanent (user environment)
[Environment]::SetEnvironmentVariable("AWS_REGION", "us-east-1", "User")
[Environment]::SetEnvironmentVariable("AWS_PROFILE", "claude-bedrock-dev", "User")
```

**CI/CD Environment (GitHub Actions):**

```yaml
env:
  AWS_REGION: us-east-1
  # Credentials provided by OIDC role assumption
```

### AWS Credentials File Setup

#### Template: AWS Credentials File

```yaml
# File: ~/.aws/credentials

[default]
aws_access_key_id = YOUR_DEFAULT_ACCESS_KEY
aws_secret_access_key = YOUR_DEFAULT_SECRET_KEY

[claude-bedrock]
aws_access_key_id = YOUR_ACCESS_KEY
aws_secret_access_key = YOUR_SECRET_KEY
region = us-east-1

[claude-bedrock-dev]
aws_access_key_id = YOUR_DEV_ACCESS_KEY
aws_secret_access_key = YOUR_DEV_SECRET_KEY
region = us-west-2

[claude-bedrock-prod]
aws_access_key_id = YOUR_PROD_ACCESS_KEY
aws_secret_access_key = YOUR_PROD_SECRET_KEY
region = us-east-1
```

#### Template: AWS Config File

```ini
# File: ~/.aws/config

[default]
region = us-east-1
output = json

[profile claude-bedrock]
region = us-east-1
output = json

[profile claude-bedrock-dev]
region = us-west-2
output = json

[profile claude-bedrock-role]
role_arn = arn:aws:iam::123456789012:role/ClaudeCodeRole
source_profile = claude-bedrock
region = us-east-1
```

### CLI Configuration

#### Using the --provider Flag

```bash
# Template: Using Claude Code with Bedrock
claude --provider bedrock --model anthropic.claude-3-sonnet-20240229-v1:0

# With inference profile
claude --provider bedrock --model us.anthropic.claude-3-sonnet-20240229-v1:0
```

#### Model Selection Examples

```bash
# Using Claude 3.5 Sonnet
claude --provider bedrock --model anthropic.claude-3-5-sonnet-20240620-v1:0

# Using Claude 3 Opus
claude --provider bedrock --model anthropic.claude-3-opus-20240229-v1:0

# Using Claude 3 Sonnet
claude --provider bedrock --model anthropic.claude-3-sonnet-20240229-v1:0

# Using Claude 3 Haiku (cost-effective)
claude --provider bedrock --model anthropic.claude-3-haiku-20240307-v1:0
```

#### Using Specific Regions

```bash
# Specify region via environment variable
AWS_REGION=us-west-2 claude --provider bedrock --model anthropic.claude-3-sonnet-20240229-v1:0

# Or using AWS profile with region configured
AWS_PROFILE=claude-bedrock-west claude --provider bedrock --model anthropic.claude-3-sonnet-20240229-v1:0
```

### Settings File Configuration

#### Template: settings.json for Bedrock

```json
{
    "provider": "bedrock",
    "model": "anthropic.claude-3-sonnet-20240229-v1:0",
    "bedrock": {
        "region": "us-east-1",
        "profile": "claude-bedrock",
        "inferenceProfile": "us.anthropic.claude-3-sonnet-20240229-v1:0"
    }
}
```

#### Extended Settings Configuration

```json
{
    "provider": "bedrock",
    "model": "anthropic.claude-3-5-sonnet-20240620-v1:0",
    "bedrock": {
        "region": "us-east-1",
        "profile": "claude-bedrock-prod",
        "inferenceProfile": "us.anthropic.claude-3-5-sonnet-20240620-v1:0",
        "maxRetries": 3,
        "timeout": 300000
    },
    "defaults": {
        "maxTokens": 4096,
        "temperature": 0.7
    }
}
```

#### Location of Settings File

```
Windows:  %APPDATA%\claude-code\settings.json
macOS:    ~/Library/Application Support/claude-code/settings.json
Linux:    ~/.config/claude-code/settings.json
```

---

## Inference Profiles

### What Are Inference Profiles?

Inference profiles are configurations that define how requests are routed to models
on Bedrock. They provide:

- **Cross-region inference**: Automatic failover across regions
- **Load balancing**: Distribute requests across endpoints
- **Custom routing**: Define your own routing strategies

### System-Defined Inference Profiles

AWS provides pre-configured inference profiles for cross-region inference:

```
+-------------------------------------------------------------------+
|              SYSTEM-DEFINED INFERENCE PROFILES                     |
+-------------------------------------------------------------------+
|                                                                    |
|  Profile Prefix     | Primary Regions                             |
|  -------------------|---------------------------------------------|
|  us.anthropic.*     | us-east-1, us-west-2                        |
|  eu.anthropic.*     | eu-west-1, eu-central-1                     |
|  apac.anthropic.*   | ap-northeast-1, ap-southeast-1              |
|                                                                    |
+-------------------------------------------------------------------+
```

### Using Inference Profiles with Claude Code

```bash
# Standard model invocation (single region)
claude --provider bedrock --model anthropic.claude-3-sonnet-20240229-v1:0

# Cross-region inference profile (US)
claude --provider bedrock --model us.anthropic.claude-3-sonnet-20240229-v1:0

# Cross-region inference profile (EU)
claude --provider bedrock --model eu.anthropic.claude-3-sonnet-20240229-v1:0
```

### Creating Custom Inference Profiles

Custom inference profiles allow you to define specific routing behaviors:

#### Via AWS Console

```
1. Navigate to Amazon Bedrock console
2. Click "Inference profiles" in the sidebar
3. Click "Create inference profile"
4. Configure:
   - Name: my-claude-profile
   - Base model: anthropic.claude-3-sonnet-20240229-v1:0
   - Regions: Select desired regions
   - Routing: Configure priority/weights
5. Click "Create"
```

#### Via AWS CLI

```bash
# Create a custom inference profile
aws bedrock create-inference-profile \
    --inference-profile-name my-claude-profile \
    --model-source '{"copyFrom": "arn:aws:bedrock:us-east-1::foundation-model/anthropic.claude-3-sonnet-20240229-v1:0"}' \
    --description "Custom inference profile for Claude Code"

# List inference profiles
aws bedrock list-inference-profiles

# Get profile details
aws bedrock get-inference-profile \
    --inference-profile-identifier my-claude-profile
```

### Cross-Region Inference Benefits

```
+-------------------------------------------------------------------+
|              CROSS-REGION INFERENCE BENEFITS                       |
+-------------------------------------------------------------------+
|                                                                    |
|  HIGH AVAILABILITY                                                 |
|  - Automatic failover if one region is unavailable                |
|  - No single point of failure                                     |
|  - Improved uptime SLA                                            |
|                                                                    |
|  PERFORMANCE                                                       |
|  - Route to nearest region automatically                          |
|  - Reduced latency for global users                               |
|  - Load balancing across regions                                  |
|                                                                    |
|  CAPACITY                                                          |
|  - Access capacity across multiple regions                        |
|  - Handle traffic spikes better                                   |
|  - Avoid regional quota limits                                    |
|                                                                    |
+-------------------------------------------------------------------+
```

### Cost Optimization with Inference Profiles

```
+-------------------------------------------------------------------+
|              INFERENCE PROFILE COST STRATEGIES                     |
+-------------------------------------------------------------------+
|                                                                    |
|  Strategy 1: Use Haiku for Simple Tasks                           |
|  ----------------------------------------                          |
|  Profile: default-haiku                                           |
|  Model: anthropic.claude-3-haiku-20240307-v1:0                    |
|  Use case: Simple queries, high volume                            |
|  Cost savings: Up to 90% vs Opus                                  |
|                                                                    |
|  Strategy 2: Regional Selection                                    |
|  ----------------------------------------                          |
|  Profile: us-only                                                 |
|  Regions: us-east-1, us-west-2                                    |
|  Use case: US-based operations                                    |
|  Cost savings: Avoid cross-region data transfer                   |
|                                                                    |
|  Strategy 3: Tiered Routing                                        |
|  ----------------------------------------                          |
|  Profile: tiered-routing                                          |
|  Primary: us-east-1 (lower cost region)                           |
|  Fallback: us-west-2                                              |
|  Cost savings: Route most traffic to lower-cost region            |
|                                                                    |
+-------------------------------------------------------------------+
```

---

## Real-World Examples

### Example 1: Enterprise Setup with Corporate AWS Account

A large enterprise wants to enable Claude Code for their development team using
their existing AWS infrastructure.

**Requirements:**
- Integrate with existing AWS SSO
- Use corporate VPC for network isolation
- Implement cost tracking by department
- Enforce model restrictions

**Implementation:**

```bash
# Step 1: Create department-specific IAM policies

# Engineering team policy
cat > engineering-policy.json << 'EOF'
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "EngineeringClaudeAccess",
            "Effect": "Allow",
            "Action": [
                "bedrock:InvokeModel",
                "bedrock:InvokeModelWithResponseStream"
            ],
            "Resource": [
                "arn:aws:bedrock:*::foundation-model/anthropic.claude-3-5-sonnet-*",
                "arn:aws:bedrock:*::foundation-model/anthropic.claude-3-sonnet-*",
                "arn:aws:bedrock:*::foundation-model/anthropic.claude-3-haiku-*"
            ],
            "Condition": {
                "ForAnyValue:StringEquals": {
                    "aws:PrincipalTag/department": "engineering"
                }
            }
        }
    ]
}
EOF

# Create policy
aws iam create-policy \
    --policy-name EngineeringClaudeAccess \
    --policy-document file://engineering-policy.json
```

```bash
# Step 2: Set up VPC endpoint for Bedrock
aws ec2 create-vpc-endpoint \
    --vpc-id vpc-0123456789abcdef0 \
    --service-name com.amazonaws.us-east-1.bedrock-runtime \
    --vpc-endpoint-type Interface \
    --subnet-ids subnet-0123456789abcdef0 subnet-0123456789abcdef1 \
    --security-group-ids sg-0123456789abcdef0 \
    --private-dns-enabled
```

```bash
# Step 3: Configure SSO permission set
# (This is typically done in AWS IAM Identity Center console)
# Create permission set: ClaudeCodeDeveloper
# Attach policy: EngineeringClaudeAccess
# Assign to: Engineering group
```

**Developer Usage:**

```bash
# Developer logs in via SSO
aws sso login --profile engineering-sso

# Use Claude Code with Bedrock
AWS_PROFILE=engineering-sso claude --provider bedrock \
    --model anthropic.claude-3-sonnet-20240229-v1:0
```

---

### Example 2: CI/CD Pipeline with Bedrock

Implementing Claude Code in a GitHub Actions CI/CD pipeline for automated code review.

**Workflow File:**

```yaml
# .github/workflows/claude-review.yml
name: Claude Code Review

on:
  pull_request:
    types: [opened, synchronize]

permissions:
  id-token: write
  contents: read
  pull-requests: write

jobs:
  code-review:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/ClaudeCodeGitHubActionsRole
          aws-region: us-east-1

      - name: Install Claude Code
        run: npm install -g @anthropic-ai/claude-code

      - name: Run Claude Code Review
        env:
          AWS_REGION: us-east-1
          CLAUDE_PROVIDER: bedrock
          CLAUDE_MODEL: anthropic.claude-3-sonnet-20240229-v1:0
        run: |
          # Get changed files
          CHANGED_FILES=$(git diff --name-only origin/main...HEAD)

          # Run Claude Code review
          claude --provider bedrock \
            --model anthropic.claude-3-sonnet-20240229-v1:0 \
            -p "Review these code changes for issues: $CHANGED_FILES"

      - name: Post review comment
        if: always()
        uses: actions/github-script@v7
        with:
          script: |
            // Post Claude's review as PR comment
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: process.env.CLAUDE_REVIEW_OUTPUT
            })
```

---

### Example 3: Multi-Region Deployment

Deploying Claude Code access across multiple regions for a global team.

**Terraform Configuration:**

```hcl
# main.tf

# Variables
variable "regions" {
  default = ["us-east-1", "eu-west-1", "ap-northeast-1"]
}

variable "account_id" {
  type = string
}

# IAM Policy (global)
resource "aws_iam_policy" "claude_bedrock" {
  name        = "ClaudeBedrockMultiRegion"
  description = "Multi-region access to Claude on Bedrock"

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Sid    = "BedrockInvokeMultiRegion"
        Effect = "Allow"
        Action = [
          "bedrock:InvokeModel",
          "bedrock:InvokeModelWithResponseStream"
        ]
        Resource = [
          for region in var.regions :
          "arn:aws:bedrock:${region}::foundation-model/anthropic.claude-*"
        ]
      }
    ]
  })
}

# IAM Role
resource "aws_iam_role" "claude_code" {
  name = "ClaudeCodeMultiRegionRole"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Principal = {
          AWS = "arn:aws:iam::${var.account_id}:root"
        }
        Action = "sts:AssumeRole"
      }
    ]
  })
}

resource "aws_iam_role_policy_attachment" "claude_bedrock" {
  role       = aws_iam_role.claude_code.name
  policy_arn = aws_iam_policy.claude_bedrock.arn
}

# Output
output "role_arn" {
  value = aws_iam_role.claude_code.arn
}
```

**Usage with Region Selection:**

```bash
# Use US region
AWS_REGION=us-east-1 claude --provider bedrock --model anthropic.claude-3-sonnet-20240229-v1:0

# Use EU region
AWS_REGION=eu-west-1 claude --provider bedrock --model anthropic.claude-3-sonnet-20240229-v1:0

# Use cross-region profile for automatic routing
claude --provider bedrock --model us.anthropic.claude-3-sonnet-20240229-v1:0
```

---

### Example 4: Cost-Optimized Inference Profile

Setting up a cost-optimized configuration using model tiering.

**Strategy:**

```
+-------------------------------------------------------------------+
|              COST-OPTIMIZED MODEL TIERING                          |
+-------------------------------------------------------------------+
|                                                                    |
|  Task Complexity    | Model               | Cost per 1M tokens    |
|  -------------------|---------------------|----------------------|
|  Simple queries     | Claude 3 Haiku      | $0.25 / $1.25        |
|  Standard tasks     | Claude 3 Sonnet     | $3.00 / $15.00       |
|  Complex analysis   | Claude 3.5 Sonnet   | $3.00 / $15.00       |
|  Research/Creative  | Claude 3 Opus       | $15.00 / $75.00      |
|                                                                    |
+-------------------------------------------------------------------+
```

**Implementation Script:**

```bash
#!/bin/bash
# smart-claude.sh - Cost-optimized Claude Code wrapper

# Analyze task complexity (simple heuristic)
analyze_complexity() {
    local prompt="$1"
    local word_count=$(echo "$prompt" | wc -w)

    # Simple tasks: short prompts, basic queries
    if [ $word_count -lt 20 ]; then
        echo "simple"
    # Standard tasks: medium prompts
    elif [ $word_count -lt 100 ]; then
        echo "standard"
    # Complex tasks: long prompts with detailed requirements
    else
        echo "complex"
    fi
}

# Select model based on complexity
select_model() {
    local complexity="$1"
    case $complexity in
        simple)
            echo "anthropic.claude-3-haiku-20240307-v1:0"
            ;;
        standard)
            echo "anthropic.claude-3-sonnet-20240229-v1:0"
            ;;
        complex)
            echo "anthropic.claude-3-5-sonnet-20240620-v1:0"
            ;;
    esac
}

# Main
PROMPT="$1"
COMPLEXITY=$(analyze_complexity "$PROMPT")
MODEL=$(select_model "$COMPLEXITY")

echo "Using model: $MODEL (complexity: $COMPLEXITY)"
claude --provider bedrock --model "$MODEL" -p "$PROMPT"
```

---

### Example 5: Federated Authentication Setup

Implementing OIDC federation for a company using Okta.

**Okta Configuration:**

```
1. In Okta Admin Console:
   - Create new App Integration
   - Type: Web
   - Sign-in method: OIDC
   - Grant type: Client credentials

2. Configure App:
   - Name: AWS Bedrock Access
   - Sign-in redirect: https://aws.amazon.com
   - Assignments: Assign to developers group

3. Note the following:
   - Client ID: 0oa1234567890abcdef
   - Issuer URL: https://company.okta.com
```

**AWS Configuration:**

```bash
# Create OIDC Provider
aws iam create-open-id-connect-provider \
    --url https://company.okta.com \
    --client-id-list 0oa1234567890abcdef \
    --thumbprint-list $(openssl s_client -connect company.okta.com:443 2>/dev/null | \
        openssl x509 -fingerprint -noout | cut -d'=' -f2 | tr -d ':' | tr '[:upper:]' '[:lower:]')

# Create trust policy
cat > okta-trust-policy.json << 'EOF'
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "arn:aws:iam::123456789012:oidc-provider/company.okta.com"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringEquals": {
                    "company.okta.com:aud": "0oa1234567890abcdef"
                }
            }
        }
    ]
}
EOF

# Create role
aws iam create-role \
    --role-name OktaClaudeCodeRole \
    --assume-role-policy-document file://okta-trust-policy.json

# Attach policy
aws iam attach-role-policy \
    --role-name OktaClaudeCodeRole \
    --policy-arn arn:aws:iam::123456789012:policy/ClaudeCodeBedrockAccess
```

**Developer Usage:**

```bash
# Get OIDC token from Okta (via Okta CLI or SDK)
OIDC_TOKEN=$(okta-get-token --client-id 0oa1234567890abcdef)

# Assume role with web identity
eval $(aws sts assume-role-with-web-identity \
    --role-arn arn:aws:iam::123456789012:role/OktaClaudeCodeRole \
    --role-session-name OktaSession \
    --web-identity-token "$OIDC_TOKEN" \
    --query 'Credentials.[AccessKeyId,SecretAccessKey,SessionToken]' \
    --output text | \
    awk '{printf "export AWS_ACCESS_KEY_ID=%s\nexport AWS_SECRET_ACCESS_KEY=%s\nexport AWS_SESSION_TOKEN=%s\n", $1, $2, $3}')

# Use Claude Code
claude --provider bedrock --model anthropic.claude-3-sonnet-20240229-v1:0
```

---

### Example 6: Team Access Management

Managing Claude Code access for different teams with varying permissions.

**Team Structure:**

```
+-------------------------------------------------------------------+
|              TEAM ACCESS MATRIX                                    |
+-------------------------------------------------------------------+
|                                                                    |
|  Team         | Models Allowed        | Regions    | Daily Limit  |
|  -------------|----------------------|------------|--------------|
|  Engineering  | All Claude 3.x       | All        | Unlimited    |
|  QA           | Haiku, Sonnet        | US only    | 10,000 req   |
|  Support      | Haiku only           | US only    | 1,000 req    |
|  Research     | All (including Opus) | All        | Unlimited    |
|                                                                    |
+-------------------------------------------------------------------+
```

**IAM Policies:**

```json
// Engineering Team Policy
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "EngineeringFullAccess",
            "Effect": "Allow",
            "Action": [
                "bedrock:InvokeModel",
                "bedrock:InvokeModelWithResponseStream"
            ],
            "Resource": [
                "arn:aws:bedrock:*::foundation-model/anthropic.claude-*"
            ]
        }
    ]
}
```

```json
// QA Team Policy
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "QALimitedAccess",
            "Effect": "Allow",
            "Action": [
                "bedrock:InvokeModel",
                "bedrock:InvokeModelWithResponseStream"
            ],
            "Resource": [
                "arn:aws:bedrock:us-*::foundation-model/anthropic.claude-3-haiku-*",
                "arn:aws:bedrock:us-*::foundation-model/anthropic.claude-3-sonnet-*"
            ]
        }
    ]
}
```

```json
// Support Team Policy
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "SupportMinimalAccess",
            "Effect": "Allow",
            "Action": [
                "bedrock:InvokeModel",
                "bedrock:InvokeModelWithResponseStream"
            ],
            "Resource": [
                "arn:aws:bedrock:us-east-1::foundation-model/anthropic.claude-3-haiku-*"
            ]
        }
    ]
}
```

---

### Example 7: Compliance Configuration

Setting up Claude Code for a regulated industry (healthcare/HIPAA).

**Requirements:**
- All data must stay in US regions
- Audit logging required
- Encryption with customer-managed keys
- VPC isolation

**Implementation:**

```bash
# Step 1: Create KMS key for encryption
aws kms create-key \
    --description "Claude Code Bedrock encryption key" \
    --key-usage ENCRYPT_DECRYPT \
    --origin AWS_KMS

# Note the key ID from output
KMS_KEY_ID=arn:aws:kms:us-east-1:123456789012:key/12345678-1234-1234-1234-123456789012

# Step 2: Create VPC endpoint
aws ec2 create-vpc-endpoint \
    --vpc-id vpc-hipaa-compliant \
    --service-name com.amazonaws.us-east-1.bedrock-runtime \
    --vpc-endpoint-type Interface \
    --subnet-ids subnet-private-1 subnet-private-2 \
    --security-group-ids sg-hipaa-compliant \
    --private-dns-enabled

# Step 3: Enable CloudTrail logging
aws cloudtrail create-trail \
    --name bedrock-audit-trail \
    --s3-bucket-name hipaa-audit-logs \
    --kms-key-id $KMS_KEY_ID \
    --enable-log-file-validation

aws cloudtrail put-event-selectors \
    --trail-name bedrock-audit-trail \
    --event-selectors '[{
        "ReadWriteType": "All",
        "IncludeManagementEvents": true,
        "DataResources": [{
            "Type": "AWS::BedrockRuntime::InvokeModel",
            "Values": ["arn:aws:bedrock:us-east-1:123456789012:*"]
        }]
    }]'
```

**Compliance-Focused IAM Policy:**

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "HIPAACompliantAccess",
            "Effect": "Allow",
            "Action": [
                "bedrock:InvokeModel",
                "bedrock:InvokeModelWithResponseStream"
            ],
            "Resource": [
                "arn:aws:bedrock:us-east-1::foundation-model/anthropic.claude-*"
            ],
            "Condition": {
                "StringEquals": {
                    "aws:RequestedRegion": "us-east-1"
                },
                "Bool": {
                    "aws:SecureTransport": "true"
                },
                "IpAddress": {
                    "aws:VpcSourceIp": "10.0.0.0/8"
                }
            }
        }
    ]
}
```

---

## Model Reference

### Complete Model ID Table

| Model | Bedrock Model ID | Max Input | Max Output |
|-------|------------------|-----------|------------|
| Claude 3.5 Sonnet | anthropic.claude-3-5-sonnet-20240620-v1:0 | 200K | 4,096 |
| Claude 3.5 Sonnet v2 | anthropic.claude-3-5-sonnet-20241022-v2:0 | 200K | 8,192 |
| Claude 3 Opus | anthropic.claude-3-opus-20240229-v1:0 | 200K | 4,096 |
| Claude 3 Sonnet | anthropic.claude-3-sonnet-20240229-v1:0 | 200K | 4,096 |
| Claude 3 Haiku | anthropic.claude-3-haiku-20240307-v1:0 | 200K | 4,096 |

### Cross-Region Inference Profile IDs

| Model | US Profile | EU Profile |
|-------|------------|------------|
| Claude 3.5 Sonnet | us.anthropic.claude-3-5-sonnet-20240620-v1:0 | eu.anthropic.claude-3-5-sonnet-20240620-v1:0 |
| Claude 3 Sonnet | us.anthropic.claude-3-sonnet-20240229-v1:0 | eu.anthropic.claude-3-sonnet-20240229-v1:0 |
| Claude 3 Haiku | us.anthropic.claude-3-haiku-20240307-v1:0 | eu.anthropic.claude-3-haiku-20240307-v1:0 |

### Model Capabilities Matrix

```
+-------------------------------------------------------------------+
|              MODEL CAPABILITIES                                    |
+-------------------------------------------------------------------+
|                                                                    |
|  Capability        | Haiku | Sonnet | Opus | 3.5 Sonnet           |
|  ------------------|-------|--------|------|----------------------|
|  Speed             | +++++ | +++    | ++   | ++++                 |
|  Cost Efficiency   | +++++ | +++    | +    | +++                  |
|  Complex Reasoning | ++    | +++    | +++++| ++++                 |
|  Code Generation   | ++    | ++++   | +++++| +++++                |
|  Creative Writing  | ++    | +++    | +++++| ++++                 |
|  Analysis          | ++    | ++++   | +++++| +++++                |
|  Vision            | Yes   | Yes    | Yes  | Yes                  |
|                                                                    |
+-------------------------------------------------------------------+
```

---

## Cost Analysis

### Detailed Pricing Comparison

#### On-Demand Pricing

| Model | Input (per 1M tokens) | Output (per 1M tokens) |
|-------|----------------------|------------------------|
| Claude 3.5 Sonnet | $3.00 | $15.00 |
| Claude 3 Opus | $15.00 | $75.00 |
| Claude 3 Sonnet | $3.00 | $15.00 |
| Claude 3 Haiku | $0.25 | $1.25 |

#### Cost Scenarios

```
+-------------------------------------------------------------------+
|              MONTHLY COST SCENARIOS                                |
+-------------------------------------------------------------------+
|                                                                    |
|  Scenario: Individual Developer                                    |
|  Usage: 1M input tokens, 200K output tokens/month                  |
|  Model: Claude 3 Sonnet                                            |
|  Cost: $3.00 + $3.00 = $6.00/month                                |
|                                                                    |
|  Scenario: Small Team (5 developers)                               |
|  Usage: 10M input tokens, 2M output tokens/month                   |
|  Model: Claude 3 Sonnet                                            |
|  Cost: $30.00 + $30.00 = $60.00/month                             |
|                                                                    |
|  Scenario: Enterprise (50 developers)                              |
|  Usage: 100M input tokens, 20M output tokens/month                 |
|  Model: Mix (70% Haiku, 30% Sonnet)                                |
|  Haiku: 70M * $0.25/M + 14M * $1.25/M = $17.50 + $17.50 = $35.00  |
|  Sonnet: 30M * $3.00/M + 6M * $15.00/M = $90.00 + $90.00 = $180.00|
|  Total: $215.00/month                                              |
|                                                                    |
|  Scenario: High-Volume Production                                  |
|  Usage: 1B input tokens, 200M output tokens/month                  |
|  Consider: Provisioned Throughput for significant savings          |
|                                                                    |
+-------------------------------------------------------------------+
```

### API vs Bedrock Decision Matrix

| Factor | API Direct | Bedrock | Winner For |
|--------|------------|---------|------------|
| Setup Complexity | Low | Medium | API (simplicity) |
| Enterprise Integration | Limited | Excellent | Bedrock (enterprise) |
| Volume Pricing | Standard | Discounts | Bedrock (high volume) |
| Compliance | Basic | Comprehensive | Bedrock (regulated) |
| Billing | Separate | Consolidated | Bedrock (enterprise) |
| Network Control | Limited | VPC/PrivateLink | Bedrock (security) |
| Audit Logging | Basic | CloudTrail | Bedrock (compliance) |

---

## Troubleshooting Guide

### Common Errors and Solutions

#### Error: "Access Denied"

**Symptoms:**
```
Error: User: arn:aws:iam::123456789012:user/developer is not authorized
to perform: bedrock:InvokeModel on resource: arn:aws:bedrock:us-east-1::foundation-model/anthropic.claude-3-sonnet-20240229-v1:0
```

**Solutions:**

1. **Check IAM policy attachment:**
```bash
# List policies attached to user
aws iam list-attached-user-policies --user-name developer

# List policies attached to role
aws iam list-attached-role-policies --role-name ClaudeCodeRole
```

2. **Verify policy permissions:**
```bash
# Simulate policy to check access
aws iam simulate-principal-policy \
    --policy-source-arn arn:aws:iam::123456789012:user/developer \
    --action-names bedrock:InvokeModel \
    --resource-arns arn:aws:bedrock:us-east-1::foundation-model/anthropic.claude-3-sonnet-20240229-v1:0
```

3. **Check for explicit denies:**
```bash
# Look for SCPs or permission boundaries
aws organizations list-policies --filter SERVICE_CONTROL_POLICY
```

---

#### Error: "Region Not Supported"

**Symptoms:**
```
Error: Bedrock is not available in region ap-south-1
```

**Solutions:**

1. **Verify region availability:**
```bash
# List available regions for Bedrock
aws ec2 describe-regions --query 'Regions[].RegionName' --output text | \
    xargs -I {} sh -c 'aws bedrock list-foundation-models --region {} 2>/dev/null && echo {}'
```

2. **Use supported region:**
```bash
# Set correct region
export AWS_REGION=us-east-1
claude --provider bedrock --model anthropic.claude-3-sonnet-20240229-v1:0
```

3. **Use cross-region inference profile:**
```bash
claude --provider bedrock --model us.anthropic.claude-3-sonnet-20240229-v1:0
```

---

#### Error: "Model Not Available"

**Symptoms:**
```
Error: The specified model is not available or access has not been granted
```

**Solutions:**

1. **Request model access:**
```
AWS Console > Bedrock > Model access > Manage model access >
Select Anthropic models > Request access
```

2. **Verify model access status:**
```bash
aws bedrock list-foundation-models \
    --query 'modelSummaries[?contains(modelId, `claude`)].{ID:modelId,Status:modelLifecycle.status}' \
    --output table
```

3. **Check for specific model version:**
```bash
# List all Claude models with their exact IDs
aws bedrock list-foundation-models \
    --query 'modelSummaries[?providerName==`Anthropic`].modelId' \
    --output text
```

---

#### Error: "Authentication Failure"

**Symptoms:**
```
Error: Unable to locate credentials. You can configure credentials by running "aws configure"
```

**Solutions:**

1. **Verify credentials are set:**
```bash
# Check current identity
aws sts get-caller-identity

# If this fails, credentials are not configured
```

2. **Configure credentials:**
```bash
# Option 1: Environment variables
export AWS_ACCESS_KEY_ID=your-key
export AWS_SECRET_ACCESS_KEY=your-secret

# Option 2: AWS CLI configuration
aws configure

# Option 3: Use named profile
aws configure --profile claude-bedrock
export AWS_PROFILE=claude-bedrock
```

3. **For temporary credentials, verify session token:**
```bash
# Session token may have expired
# Get new temporary credentials
aws sts get-session-token --duration-seconds 3600
```

---

#### Error: "Quota Exceeded"

**Symptoms:**
```
Error: You have exceeded your service quota for bedrock:InvokeModel
```

**Solutions:**

1. **Check current quotas:**
```bash
aws service-quotas get-service-quota \
    --service-code bedrock \
    --quota-code L-XXXXXXXX
```

2. **Request quota increase:**
```bash
aws service-quotas request-service-quota-increase \
    --service-code bedrock \
    --quota-code L-XXXXXXXX \
    --desired-value 1000
```

3. **Implement rate limiting in your application:**
```bash
# Use exponential backoff
# Most AWS SDKs handle this automatically
```

---

### Diagnostic Commands

```bash
# Complete diagnostic script
#!/bin/bash

echo "=== AWS Configuration Check ==="
echo "Current Identity:"
aws sts get-caller-identity

echo ""
echo "Region: ${AWS_REGION:-$(aws configure get region)}"

echo ""
echo "=== Bedrock Access Check ==="
aws bedrock list-foundation-models \
    --query 'modelSummaries[?contains(modelId, `claude`)].modelId' \
    --output table 2>&1

echo ""
echo "=== Test Invocation ==="
aws bedrock-runtime invoke-model \
    --model-id anthropic.claude-3-haiku-20240307-v1:0 \
    --body '{"anthropic_version":"bedrock-2023-05-31","max_tokens":10,"messages":[{"role":"user","content":"Hi"}]}' \
    --content-type application/json \
    --accept application/json \
    /dev/stdout 2>&1 | head -c 200

echo ""
echo "=== Diagnostic Complete ==="
```

---

## Quick Reference Cheat Sheet

### Environment Setup

```bash
# Quick setup for Bedrock
export AWS_REGION=us-east-1
export AWS_PROFILE=claude-bedrock

# Or with explicit credentials
export AWS_REGION=us-east-1
export AWS_ACCESS_KEY_ID=AKIA...
export AWS_SECRET_ACCESS_KEY=...
```

### Common Commands

```bash
# Basic usage
claude --provider bedrock --model anthropic.claude-3-sonnet-20240229-v1:0

# With cross-region profile
claude --provider bedrock --model us.anthropic.claude-3-sonnet-20240229-v1:0

# Specify region explicitly
AWS_REGION=eu-west-1 claude --provider bedrock --model anthropic.claude-3-sonnet-20240229-v1:0

# Cost-effective option
claude --provider bedrock --model anthropic.claude-3-haiku-20240307-v1:0

# Most capable option
claude --provider bedrock --model anthropic.claude-3-5-sonnet-20240620-v1:0
```

### Model Quick Reference

| Short Name | Full Model ID |
|------------|---------------|
| Haiku | anthropic.claude-3-haiku-20240307-v1:0 |
| Sonnet | anthropic.claude-3-sonnet-20240229-v1:0 |
| Opus | anthropic.claude-3-opus-20240229-v1:0 |
| Sonnet 3.5 | anthropic.claude-3-5-sonnet-20240620-v1:0 |

### IAM Quick Reference

```json
// Minimal policy
{
    "Version": "2012-10-17",
    "Statement": [{
        "Effect": "Allow",
        "Action": ["bedrock:InvokeModel", "bedrock:InvokeModelWithResponseStream"],
        "Resource": ["arn:aws:bedrock:*::foundation-model/anthropic.claude-*"]
    }]
}
```

### AWS CLI Verification

```bash
# Verify credentials
aws sts get-caller-identity

# List Claude models
aws bedrock list-foundation-models --query 'modelSummaries[?contains(modelId,`claude`)].modelId'

# Test invocation
aws bedrock-runtime invoke-model \
    --model-id anthropic.claude-3-haiku-20240307-v1:0 \
    --body '{"anthropic_version":"bedrock-2023-05-31","max_tokens":100,"messages":[{"role":"user","content":"Hello"}]}' \
    --content-type application/json \
    --accept application/json \
    output.json
```

---

## Best Practices

### Security Best Practices

#### 1. IAM Least Privilege

```
PRINCIPLE: Grant only the permissions necessary for the task

DO:
- Use specific resource ARNs
- Add conditions (region, IP, time)
- Separate policies by team/function
- Regular access reviews

DON'T:
- Use wildcard (*) for resources
- Share credentials
- Grant bedrock:* permissions
- Use long-term credentials in CI/CD
```

**Example - Least Privilege Policy:**

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "LeastPrivilegeClaudeAccess",
            "Effect": "Allow",
            "Action": [
                "bedrock:InvokeModelWithResponseStream"
            ],
            "Resource": [
                "arn:aws:bedrock:us-east-1::foundation-model/anthropic.claude-3-sonnet-20240229-v1:0"
            ],
            "Condition": {
                "IpAddress": {
                    "aws:SourceIp": "192.168.1.0/24"
                },
                "DateGreaterThan": {
                    "aws:CurrentTime": "2024-01-01T00:00:00Z"
                },
                "DateLessThan": {
                    "aws:CurrentTime": "2025-01-01T00:00:00Z"
                }
            }
        }
    ]
}
```

#### 2. Credential Management

```
BEST PRACTICES FOR CREDENTIALS:

Development:
- Use AWS profiles (not environment variables)
- Enable MFA for IAM users
- Use short-lived credentials when possible

CI/CD:
- Use OIDC federation (no stored secrets)
- Implement role assumption with session tags
- Audit credential usage via CloudTrail

Production:
- Use IAM roles for AWS resources
- Never embed credentials in code
- Rotate credentials regularly
```

### Reliability Best Practices

#### 1. Use Inference Profiles

```
BENEFITS OF INFERENCE PROFILES:

High Availability:
- Automatic failover across regions
- No single point of failure
- Improved uptime

Performance:
- Load balancing across endpoints
- Reduced latency variability
- Better handling of traffic spikes

Configuration:
claude --provider bedrock --model us.anthropic.claude-3-sonnet-20240229-v1:0
                                 ^^^ Cross-region profile prefix
```

#### 2. Implement Retry Logic

```bash
# Example retry wrapper script
#!/bin/bash

MAX_RETRIES=3
RETRY_DELAY=5

for i in $(seq 1 $MAX_RETRIES); do
    if claude --provider bedrock --model anthropic.claude-3-sonnet-20240229-v1:0 "$@"; then
        exit 0
    fi

    if [ $i -lt $MAX_RETRIES ]; then
        echo "Attempt $i failed, retrying in ${RETRY_DELAY}s..."
        sleep $RETRY_DELAY
        RETRY_DELAY=$((RETRY_DELAY * 2))  # Exponential backoff
    fi
done

echo "All $MAX_RETRIES attempts failed"
exit 1
```

### Cost Management Best Practices

#### 1. Monitor Costs with AWS Cost Explorer

```
SETUP COST MONITORING:

1. Enable Cost Explorer:
   AWS Console > Cost Management > Cost Explorer

2. Create cost allocation tags:
   - project: claude-code
   - environment: production
   - team: engineering

3. Set up cost anomaly detection:
   AWS Cost Management > Cost Anomaly Detection > Create monitor

4. Configure budget alerts:
   AWS Budgets > Create budget > Set monthly threshold
```

#### 2. Set Up Billing Alerts

```bash
# Create a billing alarm via CLI
aws cloudwatch put-metric-alarm \
    --alarm-name "Claude-Bedrock-Spending-Alert" \
    --alarm-description "Alert when Bedrock spending exceeds $100" \
    --metric-name EstimatedCharges \
    --namespace AWS/Billing \
    --statistic Maximum \
    --period 21600 \
    --threshold 100 \
    --comparison-operator GreaterThanThreshold \
    --dimensions Name=ServiceName,Value=AmazonBedrock \
    --evaluation-periods 1 \
    --alarm-actions arn:aws:sns:us-east-1:123456789012:billing-alerts
```

#### 3. Optimize Model Selection

```
MODEL SELECTION GUIDELINES:

Use Haiku ($0.25/$1.25 per 1M tokens) for:
- Simple classification tasks
- Basic Q&A
- High-volume, low-complexity operations
- Testing and development

Use Sonnet ($3/$15 per 1M tokens) for:
- General coding tasks
- Documentation generation
- Standard analysis work
- Most production workloads

Use Opus ($15/$75 per 1M tokens) for:
- Complex reasoning tasks
- Research and analysis
- Nuanced content creation
- When quality is paramount

Use Claude 3.5 Sonnet ($3/$15 per 1M tokens) for:
- Complex coding tasks
- Advanced analysis
- When you need best capability at Sonnet price
```

### Operational Best Practices

#### 1. Implement Logging and Monitoring

```bash
# Enable CloudWatch logging for Bedrock
# Create log group
aws logs create-log-group --log-group-name /aws/bedrock/claude-code

# Create metric filter for errors
aws logs put-metric-filter \
    --log-group-name /aws/bedrock/claude-code \
    --filter-name ErrorCount \
    --filter-pattern "ERROR" \
    --metric-transformations \
        metricName=BedrockErrors,metricNamespace=ClaudeCode,metricValue=1

# Create alarm for errors
aws cloudwatch put-metric-alarm \
    --alarm-name "Claude-Code-Error-Rate" \
    --metric-name BedrockErrors \
    --namespace ClaudeCode \
    --statistic Sum \
    --period 300 \
    --threshold 10 \
    --comparison-operator GreaterThanThreshold \
    --evaluation-periods 1 \
    --alarm-actions arn:aws:sns:us-east-1:123456789012:ops-alerts
```

#### 2. Implement Tagging Strategy

```
RECOMMENDED TAGS:

Required Tags:
- environment: dev/staging/prod
- project: project-name
- team: team-name
- cost-center: CC-12345

Optional Tags:
- owner: email@company.com
- created-by: automation/manual
- compliance: hipaa/pci/sox

Tag your IAM roles:
aws iam tag-role --role-name ClaudeCodeRole --tags \
    Key=environment,Value=production \
    Key=project,Value=claude-code \
    Key=team,Value=engineering
```

---

## Summary

### Key Takeaways

1. **Amazon Bedrock** provides enterprise-grade access to Claude models with AWS integration
2. **IAM permissions** require at minimum `bedrock:InvokeModel` and `bedrock:InvokeModelWithResponseStream`
3. **Authentication** can use credentials, profiles, IAM roles, or OIDC federation
4. **Inference profiles** enable cross-region reliability and load balancing
5. **Cost optimization** involves selecting appropriate models for each task complexity

### Quick Start Checklist

```
[ ] AWS account with Bedrock access
[ ] Claude model access approved
[ ] IAM policy created and attached
[ ] AWS credentials configured
[ ] Claude Code installed
[ ] Environment variables set
[ ] Test invocation successful
```

### Next Steps

After completing this tutorial, consider:

1. **Exploring advanced features:**
   - Provisioned Throughput for predictable performance
   - Guardrails for content filtering
   - Model evaluation for benchmarking

2. **Implementing production patterns:**
   - VPC endpoints for network isolation
   - CloudTrail for audit logging
   - AWS Config for compliance monitoring

3. **Optimizing for your use case:**
   - Custom inference profiles
   - Model tiering strategies
   - Cost allocation and budgeting

### Additional Resources

- AWS Bedrock Documentation: https://docs.aws.amazon.com/bedrock/
- Anthropic Claude Documentation: https://docs.anthropic.com/
- AWS IAM Best Practices: https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html
- Claude Code Documentation: https://docs.anthropic.com/claude-code/

---

*Tutorial Version: 1.0*
*Last Updated: 2024*
*Compatible with: Claude Code v1.x, AWS Bedrock (current)*
