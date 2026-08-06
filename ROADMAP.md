# Roadmap

## 0.1

- Static validation for `mcpServers` config files.
- Local stdio startup probe with `initialize` and capability list checks.
- Secret, command, shell wrapper, cwd, Docker volume, and package pinning checks.
- JSON output for CI and editor integrations.

## 0.2

- SARIF output for GitHub code scanning.
- Composite GitHub Action wrapper.
- PyPI Trusted Publishing workflow.
- First PyPI release (`v0.2.0`).
- `--output` support for structured reports.

## Next

- `--fix` suggestions for safe mechanical edits.
- More client config discovery for Claude Code, Windsurf, JetBrains IDEs, and VS Code profiles.
- Better support for Streamable HTTP and OAuth metadata validation.
- Publish the existing Action in GitHub Actions Marketplace.

## Later

- Known MCP server recipe catalog with expected packages and config examples.
- Optional allowlist policy file for teams.
- Markdown report generation for support tickets and issue templates.
