# Claude Code Orchestrator

A repository demonstrating the GitHub + Claude Code workflow for AI-powered development automation.

## Overview

This repository serves as a template for managing AI agents using GitHub Issues and Pull Requests with Claude Code.

## Quick Start

1. Install the Claude GitHub App from [github.com/apps/claude](https://github.com/apps/claude)
2. Add `ANTHROPIC_API_KEY` to your repository secrets
3. Copy the workflow file from [examples/claude.yml](https://github.com/anthropics/claude-code-action/blob/main/examples/claude.yml)

## Usage

### Trigger Claude in Issues

```
@claude implement feature based on issue #123
@claude fix the bug described in issue #456
```

### Trigger Claude in PRs

```
@claude review this PR
@claude how should I implement user authentication?
```

## Documentation

- [CLAUDE.md](CLAUDE.md) - Project standards and guidelines
- [Workflow Guide](github-cloud-code-manager-workflow.md) - Detailed workflow documentation

## License

MIT