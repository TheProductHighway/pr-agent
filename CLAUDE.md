# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview
PR-Agent is an AI-powered code review tool that automates pull request analysis and feedback. It supports multiple Git providers (GitHub, GitLab, Bitbucket, Azure DevOps) and AI models (OpenAI, Claude, local models).

## Common Development Commands

### Setup and Installation
```bash
# Install development dependencies
pip install -r requirements-dev.txt

# Install package in development mode
pip install -e .
```

### Testing and Code Quality
```bash
# Run all tests
pytest

# Run tests with coverage
pytest --cov=pr_agent

# Run linting
ruff check .

# Run security scanning
bandit -r pr_agent/
```

### Running Locally
```bash
# CLI usage (after installing)
pr-agent --pr-url=<URL> review

# Start webhook server
python pr_agent/servers/github_app.py

# Run with Docker
docker build . -t pr-agent
docker run -e OPENAI_KEY=<key> pr-agent --pr-url=<URL> review
```

## Architecture Overview

### Core Components

1. **Git Providers** (`pr_agent/git_providers/`): Abstract interface for different Git platforms
   - Base class: `GitProvider` defines common interface
   - Implementations: `GithubProvider`, `GitlabProvider`, `BitbucketProvider`, etc.
   - Each provider handles platform-specific API interactions

2. **AI Handlers** (`pr_agent/algo/ai_handlers/`): Abstract interface for AI models
   - Base class: `BaseAiHandler` defines model interaction patterns
   - Main implementation: `LiteLLMAIHandler` supports multiple models
   - Handles token management, retries, and model-specific quirks

3. **Tools** (`pr_agent/tools/`): Individual PR analysis features
   - Each tool inherits from a base class and implements specific functionality
   - Examples: `PRReviewer`, `PRCodeSuggestions`, `PRDescription`
   - Tools are invoked via commands (e.g., `/review`, `/improve`)

4. **Configuration** (`pr_agent/settings/`):
   - Main config: `configuration.toml` (DO NOT modify directly)
   - Custom configs can be added via `.pr_agent.toml` in repo root
   - Configuration hierarchy: defaults → repo config → user args

### Key Design Patterns

- **Provider Pattern**: Abstracts Git platform differences
- **Handler Pattern**: Abstracts AI model differences  
- **Command Pattern**: Each tool responds to specific commands
- **Token Management**: Smart compression and patch handling for large PRs

### Important Files to Know

- `pr_agent/agent/pr_agent.py`: Main entry point and command dispatcher
- `pr_agent/settings/configuration.toml`: Default configuration and prompts
- `pr_agent/algo/utils.py`: Core utilities for token counting and patch processing
- `pr_agent/servers/`: Various deployment modes (GitHub App, GitLab webhook, etc.)

### Adding New Features

1. **New Tool**: Create class in `pr_agent/tools/`, inherit from base, register in `pr_agent.py`
2. **New Provider**: Implement `GitProvider` interface in `pr_agent/git_providers/`
3. **New Prompts**: Add to relevant section in `configuration.toml`
4. **Configuration**: Add new settings to `configuration.toml` with defaults

### Token and Context Management

The codebase uses sophisticated token management to handle large PRs:
- Automatic patch compression when exceeding token limits
- Smart file filtering based on relevance
- Incremental review for very large PRs
- Context enrichment with repo structure and dependencies

### Testing Approach

- Unit tests focus on individual components
- Integration tests verify provider interactions
- Mock AI responses for deterministic testing
- Use fixtures for common test data