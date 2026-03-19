# GitHub Spec Kit Setup on GitHub Codespaces

This document outlines the steps to set up GitHub Spec Kit for spec-driven development in a GitHub Codespaces environment.

## Prerequisites

- GitHub Codespaces environment running Ubuntu 24.04.3 LTS
- Git repository initialized
- Basic command-line tools available (curl, etc.)

## Installation Steps

### 1. Install uv (Python Package Manager)

uv is required for installing and managing the Specify CLI tool.

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

This installs uv to `~/.local/bin` and makes it available in your PATH.

### 2. Install Specify CLI

Install the GitHub Spec Kit CLI tool using uv.

```bash
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git
```

This installs the `specify` command globally and all its dependencies.

### 3. Initialize Project

Initialize the current project directory with GitHub Spec Kit, configured for GitHub Copilot.

```bash
specify init --here --ai copilot --force
```

**Note:** The `--force` flag is used to skip confirmation when the directory is not empty (e.g., already has .git and .gitignore).

## What Gets Installed

After successful initialization, the following structure is created:

- `.specify/` - Main configuration and scripts directory
  - `init-options.json` - Initialization settings
  - `memory/` - Project memory storage
  - `scripts/bash/` - Bash scripts for automation
  - `templates/` - Template files
- `.github/` - GitHub workflows and CI/CD templates
- `.vscode/` - VS Code settings and extensions

## Verification

After setup, verify the installation:

```bash
specify check
```

This command checks that all required tools are installed and the CLI is working.

## Usage

Once set up, use the following slash commands in your AI agent (GitHub Copilot) chat:

### Core Workflow:
1. `/speckit.constitution` - Establish project principles
2. `/speckit.specify` - Create feature specifications
3. `/speckit.plan` - Generate implementation plans
4. `/speckit.tasks` - Break down into actionable tasks
5. `/speckit.implement` - Execute implementation

### Enhancement Commands:
- `/speckit.clarify` - Resolve ambiguities
- `/speckit.analyze` - Check consistency
- `/speckit.checklist` - Generate quality checklists

## Troubleshooting

- If `uv` command is not found, ensure `~/.local/bin` is in your PATH
- If `specify` command fails, try reinstalling with `uv tool install specify-cli --force --from git+https://github.com/github/spec-kit.git`
- For existing projects, always use `--force` with `specify init --here`

## Notes

- This setup is optimized for GitHub Codespaces with Ubuntu
- The tool integrates with GitHub Copilot for AI-assisted development
- All scripts are POSIX-compliant (bash) for cross-platform compatibility</content>
<parameter name="filePath">/workspaces/AngularNodePostgreSQL/setup-on-codespace.md