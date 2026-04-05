# Manager of AI Agents: GitHub + Cloud Code Workflow

## Overview

This guide enables you to act as a **manager of AI agents** rather than a web coder. You review all PRs, assign tasks via GitHub, and use your preferred model for development workflows.

```
You (Manager) ←→ GitHub ←→ AI Agents (Claude Code)
                    ↓
              PR Reviews & Task Assignment
```

---

## 1. Claude Code GitHub Integration

### Quick Setup

```bash
# In Claude Code CLI, run:
/install-github-app
```

This command guides you through setting up the GitHub App and required secrets.

> **Note:** You must be a repository admin to install the GitHub app and add secrets. This quickstart method is only available for direct Claude API users.

### Manual Setup (Alternative)

1. **Install the Claude GitHub App**: [https://github.com/apps/claude](https://github.com/apps/claude)
   - Required permissions: Contents (Read/Write), Issues (Read/Write), Pull requests (Read/Write)
2. **Add ANTHROPIC_API_KEY** to repository secrets
3. **Copy the workflow file** from [examples/claude.yml](https://github.com/anthropics/claude-code-action/blob/main/examples/claude.yml)

---

## 2. Workflow Files

### Basic Workflow (Responds to @mentions)

```yaml
# .github/workflows/claude.yml
name: Claude Code
on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]
jobs:
  claude:
    runs-on: ubuntu-latest
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          # Responds to @claude mentions in comments
```

### PR Review Workflow

```yaml
# .github/workflows/review.yml
name: Code Review
on:
  pull_request:
    types: [opened, synchronize]
jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: "Review this pull request for code quality, correctness, and security. Analyze the diff, then post your findings as review comments."
          claude_args: "--max-turns 5"
```

### Daily Task Automation

```yaml
name: Daily Report
on:
  schedule:
    - cron: "0 9 * * *"  # 9am daily
jobs:
  report:
    runs-on: ubuntu-latest
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: "Generate a summary of yesterday's commits and open issues"
          claude_args: "--model opus"
```

### @claude Commands in Issues/PRs

```
@claude implement this feature based on the issue description
@claude how should I implement user authentication for this endpoint?
@claude fix the TypeError in the user dashboard component
```

---

## 3. Upgrading from Beta to v1.0

<Warning>
Claude Code GitHub Actions v1.0 introduces breaking changes that require updating your workflow files.
</Warning>

### Breaking Changes Reference

| Old Beta Input        | New v1.0 Input                        |
| --------------------- | ------------------------------------- |
| `mode`                | *(Removed - auto-detected)*           |
| `direct_prompt`       | `prompt`                              |
| `override_prompt`     | `prompt` with GitHub variables        |
| `custom_instructions` | `claude_args: --append-system-prompt` |
| `max_turns`           | `claude_args: --max-turns`            |
| `model`               | `claude_args: --model`                |
| `allowed_tools`       | `claude_args: --allowedTools`          |
| `disallowed_tools`    | `claude_args: --disallowedTools`       |
| `claude_env`          | `settings` JSON format                |

### Before and After Example

**Beta version:**

```yaml
- uses: anthropics/claude-code-action@beta
  with:
    mode: "tag"
    direct_prompt: "Review this PR for security issues"
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    custom_instructions: "Follow our coding standards"
    max_turns: "10"
    model: "claude-sonnet-4-6"
```

**GA version (v1.0):**

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    prompt: "Review this PR for security issues"
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    claude_args: |
      --append-system-prompt "Follow our coding standards"
      --max-turns 10
      --model claude-sonnet-4-6
```

---

## 4. AWS Bedrock Integration

### Prerequisites

1. AWS Bedrock access enabled with Claude model permissions
2. GitHub configured as an OIDC identity provider in AWS
3. IAM role with Bedrock permissions that trusts GitHub Actions

### Required Secrets

| Secret Name          | Description                                       |
| -------------------- | ------------------------------------------------- |
| `AWS_ROLE_TO_ASSUME` | ARN of the IAM role for Bedrock access            |
| `APP_ID`             | Your GitHub App ID (from app settings)            |
| `APP_PRIVATE_KEY`    | The private key you generated for your GitHub App |

### AWS Bedrock Workflow

```yaml
name: Claude PR Action

permissions:
  contents: write
  pull-requests: write
  issues: write
  id-token: write

on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]
  issues:
    types: [opened, assigned]

jobs:
  claude-pr:
    if: |
      (github.event_name == 'issue_comment' && contains(github.event.comment.body, '@claude')) ||
      (github.event_name == 'pull_request_review_comment' && contains(github.event.comment.body, '@claude')) ||
      (github.event_name == 'issues' && contains(github.event.issue.body, '@claude'))
    runs-on: ubuntu-latest
    env:
      AWS_REGION: us-west-2
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Generate GitHub App token
        id: app-token
        uses: actions/create-github-app-token@v2
        with:
          app-id: ${{ secrets.APP_ID }}
          private-key: ${{ secrets.APP_PRIVATE_KEY }}

      - name: Configure AWS Credentials (OIDC)
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_TO_ASSUME }}
          aws-region: us-west-2

      - uses: anthropics/claude-code-action@v1
        with:
          github_token: ${{ steps.app-token.outputs.token }}
          use_bedrock: "true"
          claude_args: '--model us.anthropic.claude-sonnet-4-6 --max-turns 10'
```

---

## 5. Google Vertex AI Integration

### Prerequisites

1. Vertex AI API enabled in your GCP project
2. Workload Identity Federation configured for GitHub
3. Service account with Vertex AI permissions

### Required Secrets

| Secret Name                      | Description                                       |
| -------------------------------- | ------------------------------------------------- |
| `GCP_WORKLOAD_IDENTITY_PROVIDER` | Workload identity provider resource name          |
| `GCP_SERVICE_ACCOUNT`            | Service account email with Vertex AI access       |
| `APP_ID`                          | Your GitHub App ID (from app settings)           |
| `APP_PRIVATE_KEY`                 | The private key you generated for your GitHub App |

### Google Vertex AI Workflow

```yaml
name: Claude PR Action

permissions:
  contents: write
  pull-requests: write
  issues: write
  id-token: write

on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]
  issues:
    types: [opened, assigned]

jobs:
  claude-pr:
    if: |
      (github.event_name == 'issue_comment' && contains(github.event.comment.body, '@claude')) ||
      (github.event_name == 'pull_request_review_comment' && contains(github.event.comment.body, '@claude')) ||
      (github.event_name == 'issues' && contains(github.event.issue.body, '@claude'))
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Generate GitHub App token
        id: app-token
        uses: actions/create-github-app-token@v2
        with:
          app-id: ${{ secrets.APP_ID }}
          private-key: ${{ secrets.APP_PRIVATE_KEY }}

      - name: Authenticate to Google Cloud
        id: auth
        uses: google-github-actions/auth@v2
        with:
          workload_identity_provider: ${{ secrets.GCP_WORKLOAD_IDENTITY_PROVIDER }}
          service_account: ${{ secrets.GCP_SERVICE_ACCOUNT }}

      - uses: anthropics/claude-code-action@v1
        with:
          github_token: ${{ steps.app-token.outputs.token }}
          trigger_phrase: "@claude"
          use_vertex: "true"
          claude_args: '--model claude-sonnet-4-5@20250929 --max-turns 10'
        env:
          ANTHROPIC_VERTEX_PROJECT_ID: ${{ steps.auth.outputs.project_id }}
          CLOUD_ML_REGION: us-east5
          VERTEX_REGION_CLAUDE_4_5_SONNET: us-east5
```

---

## 6. Action Parameters Reference

| Parameter            | Description                                                          | Required |
| -------------------- | -------------------------------------------------------------------- | -------- |
| `prompt`             | Instructions for Claude (plain text or a skill name)                 | No*      |
| `claude_args`        | CLI arguments passed to Claude Code                                  | No       |
| `anthropic_api_key`  | Claude API key                                                       | Yes**    |
| `github_token`       | GitHub token for API access                                          | No       |
| `trigger_phrase`     | Custom trigger phrase (default: "@claude")                            | No       |
| `use_bedrock`        | Use AWS Bedrock instead of Claude API                                 | No       |
| `use_vertex`         | Use Google Vertex AI instead of Claude API                            | No       |

*Prompt is optional - when omitted for issue/PR comments, Claude responds to trigger phrase
**Required for direct Claude API, not for Bedrock/Vertex

### Common CLI Arguments

```yaml
claude_args: "--max-turns 5 --model claude-sonnet-4-6 --mcp-config /path/to/config.json"
```

- `--max-turns`: Maximum conversation turns (default: 10)
- `--model`: Model to use (e.g., `claude-sonnet-4-6`, `claude-opus-4-6`)
- `--mcp-config`: Path to MCP configuration
- `--allowedTools`: Comma-separated list of allowed tools
- `--debug`: Enable debug output

---

## 7. Your Manager Workflow (Summary)

### Level 1: PR Review as Manager

1. Agent creates a PR (via Claude Code or Cloud Agent)
2. You get notified in GitHub
3. You comment: `@claude review this PR`
4. Claude provides line-by-line review
5. You approve/reject/request changes

### Level 2: Task Assignment

1. Create a GitHub Issue with requirements
2. Assign to agent (or comment `@claude implement`)
3. Agent works autonomously, creates PR when ready
4. You review the PR

---

## 8. Best Practices

### CLAUDE.md Configuration

Create a `CLAUDE.md` file in your repository root to define code style guidelines, review criteria, project-specific rules, and preferred patterns.

```markdown
# Code Review Standards
- All PRs need tests
- No direct commits to main
- Security review required for auth changes

# Agent Behavior
- Always create PRs, never commit directly
- Respond to @claude mentions within 1 hour
- Include test coverage for new features
```

### Security Considerations

<Warning>Never commit API keys directly to your repository.</Warning>

- Use GitHub Secrets for API keys
- Reference secrets: `anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}`
- Limit action permissions to only what's necessary
- Review Claude's suggestions before merging

### Cost Optimization

- Use specific `@claude` commands to reduce unnecessary API calls
- Configure appropriate `--max-turns` in `claude_args`
- Set workflow-level timeouts to avoid runaway jobs
- Use GitHub's concurrency controls to limit parallel runs

---

## 9. Troubleshooting

### Claude not responding to @claude commands

1. Verify the GitHub App is installed correctly
2. Check that workflows are enabled
3. Ensure API key is set in repository secrets
4. Confirm the comment contains `@claude` (not `/claude`)

### CI not running on Claude's commits

1. Ensure you're using the GitHub App or custom app (not Actions user)
2. Check workflow triggers include the necessary events
3. Verify app permissions include CI triggers

### Authentication errors

1. Confirm API key is valid and has sufficient permissions
2. For Bedrock/Vertex, check credentials configuration
3. Ensure secrets are named correctly in workflows

---

## 10. Key Commands

```bash
# 1. Install GitHub app
/claude
/install-github-app

# 2. In a PR comment, trigger review:
@claude review this PR for security issues

# 3. In an issue comment, trigger implementation:
@claude implement feature based on issue #123

# 4. Ask for help:
@claude how should I implement user authentication?
```

---

## 11. Key Tools Summary

| Tool                        | Purpose                                      |
| --------------------------- | -------------------------------------------- |
| **Claude Code CLI**         | Agent that implements features, reviews code  |
| **Claude GitHub App**       | Enables @mentions, auto-reviews              |
| **VS Code Cloud Agents**   | Unified view of all agent sessions           |
| **GitHub Actions**          | Automate workflows                           |
| **CLAUDE.md**               | Define project standards (agents follow this) |
| **AWS Bedrock**             | Enterprise Claude integration via AWS         |
| **Google Vertex AI**        | Enterprise Claude integration via GCP        |

---

## 12. Additional Resources

- [Claude Code Action Repository](https://github.com/anthropics/claude-code-action)
- [Examples Directory](https://github.com/anthropics/claude-code-action/tree/main/examples)
- [Security Documentation](https://github.com/anthropics/claude-code-action/blob/main/docs/security.md)
- [AWS Bedrock Setup](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_create_oidc.html)
- [Google Cloud Workload Identity Federation](https://cloud.google.com/iam/docs/workload-identity-federation)
