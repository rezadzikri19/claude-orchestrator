# Claude Code Project Standards

## Overview

This repository uses Claude Code with GitHub Actions for AI-powered development automation. MiniMax M2.7 is configured as the default model.

## Workflow

- All agent changes must be submitted via Pull Requests
- No direct commits to main branch
- @claude mentions in PRs/issues trigger AI review and implementation

## Code Standards

- Write clear, maintainable code with appropriate comments
- Include tests for new functionality
- Follow existing code patterns in the repository

## Commands

```bash
# Trigger Claude in a PR or issue comment:
@claude review this PR
@claude implement feature based on issue #123
@claude fix the bug described in issue #456
```

## Security

- Never commit secrets or API keys
- Use GitHub Secrets for all sensitive values
- Review AI-generated code before merging
