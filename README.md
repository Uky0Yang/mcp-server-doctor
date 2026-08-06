# mcp-server-doctor

Diagnose the **MCP client-to-server boundary**: configuration discovery, process launch, and the stdio JSON-RPC handshake that must succeed before tools can load.

`mcp-server-doctor` is a dependency-free Python CLI for people who configure local Model Context Protocol servers in Claude, Cursor, Copilot, VS Code, or another agent client and then see missing commands, hidden startup failures, or empty tool lists.

It statically checks client configuration and can launch a configured local stdio process to verify `initialize`, `notifications/initialized`, and advertised capability listing such as `tools/list`.

> This is not a generic MCP server test client. It focuses on failures that happen **before tool invocation**: locating client config, resolving the executable and working directory, starting the process, preserving JSON-RPC stdout framing, and completing the stdio handshake.

## Why This Exists

MCP adoption is moving quickly across AI coding tools, IDEs, and desktop assistants. The painful part is still local setup:

- Config files live in different places across clients.
- `npx`, `uvx`, Docker, Python, and absolute-path commands fail differently.
- A server can show as configured but still expose no tools.
- Tokens often get pasted into config files during troubleshooting.
- Server logs accidentally written to stdout can break JSON-RPC.

This tool gives developers and maintainers a fast preflight check for the exact boundary where an MCP client tries to start and discover a local server.

| In scope | Out of scope |
| --- | --- |
| Client config discovery and validation | Testing whether individual MCP tools are semantically correct |
| Command, package, environment, cwd, and shell-wrapper checks | Acting as an interactive MCP tool runner |
| Local process startup and stdio JSON-RPC framing | Dynamic testing of remote HTTP or OAuth transports |
| Initialize and advertised capability listing | Security approval for calling a discovered tool |

## Install

Run without installing:

```bash
uvx mcp-server-doctor check .
```

Or install the isolated CLI:

```bash
pipx install mcp-server-doctor
```

For local development:

```bash
python -m pip install -e .
```

Standard pip installation:

```bash
python -m pip install mcp-server-doctor
```

## Quick Start

Check the current repository for MCP config files:

```bash
mcp-server-doctor .
```

Check a specific config file:

```bash
mcp-server-doctor check ~/.cursor/mcp.json
```

Run static checks plus stdio startup probes:

```bash
mcp-server-doctor doctor ~/.cursor/mcp.json
```

Probe one server:

```bash
mcp-server-doctor doctor ~/.cursor/mcp.json --server filesystem
```

Return JSON for CI or editor integrations:

```bash
mcp-server-doctor doctor . --format json
```

Write SARIF for GitHub code scanning:

```bash
mcp-server-doctor check . --format sarif --output mcp-server-doctor.sarif
```

Short alias:

```bash
mcp-doctor doctor .
```

## What It Checks

Static checks:

- Valid JSON
- Required `mcpServers` object
- Local stdio server command, args, env, and cwd types
- Missing executable on `PATH`
- Missing absolute command path
- Missing or invalid cwd
- Literal token-like values in `env` or `args`
- Secret-looking env values such as API keys and passwords
- Shell wrapper usage such as `bash -c` or `powershell -Command`
- Unpinned `npx`, `uvx`, or `bunx` package names
- Broad Docker volume mounts
- Remote MCP server configs, reported as static-check only

Dynamic stdio probe:

- Starts the configured command
- Sends `initialize`
- Sends `notifications/initialized`
- Lists tools when available
- Lists resources and prompts when advertised
- Detects non-JSON stdout before a protocol response
- Captures stderr tail on failures without exposing config secrets

## Example

Given:

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/tmp"]
    }
  }
}
```

Run:

```bash
mcp-server-doctor check mcp.json
```

Output:

```text
mcp-server-doctor checked 1 config file(s), 1 server(s)
errors=0 warnings=1 info=0

Configs:
  - mcp.json
    - filesystem (stdio)

Findings:
  [warning] mcp.json#filesystem unpinned-package: npx package is not version-pinned: @modelcontextprotocol/server-filesystem
    hint: Pin package versions for reproducible MCP startup.
```

## Supported Config Locations

When no path is provided, the tool scans the current directory and common user-level locations when present:

- `mcp.json`
- `.mcp.json`
- `.cursor/mcp.json`
- `.vscode/mcp.json`
- `.github/copilot/mcp.json`
- Claude Desktop config on Windows, macOS, and Linux

Large generated folders such as `.git`, `node_modules`, `dist`, `build`, and virtual environments are skipped.

## CI

Use the packaged GitHub Action:

```yaml
name: MCP config check

on:
  pull_request:
  push:
    branches:
      - main

permissions:
  contents: read
  security-events: write

jobs:
  mcp-doctor:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7.0.0
      - uses: Uky0Yang/mcp-server-doctor@v0.2.0
        with:
          path: .
          command: check
          upload-results: "true"
          warnings-as-errors: "true"
```

Or install the CLI directly:

```yaml
name: MCP config check

on:
  pull_request:
  push:
    branches:
      - main

jobs:
  mcp-doctor:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7.0.0
      - uses: actions/setup-python@v6.3.0
        with:
          python-version: "3.12"
      - run: python -m pip install mcp-server-doctor
      - run: mcp-server-doctor check . --warnings-as-errors
```

Use `check` in CI unless your workflow intentionally installs and starts every configured MCP server.

## Publishing releases

`mcp-server-doctor` is published on PyPI through Trusted Publishing. Maintainers can follow the version, tag, and verification checklist in [docs/publishing.md](docs/publishing.md).

## Exit Codes

- `0`: no errors
- `1`: static check failed, or warnings failed under `--warnings-as-errors`
- `2`: static checks passed, but at least one stdio probe failed

## Design Principles

- No model calls
- No network calls by the doctor itself
- No runtime dependencies
- No secret values printed
- Clear findings that point to a fix
- Dynamic probing limited to local stdio commands from the provided config

## Current Limitations

- Streamable HTTP and OAuth flows are not dynamically probed yet.
- The stdio probe uses newline-delimited JSON-RPC, which is what common MCP SDK stdio transports use.
- The tool does not rewrite configs yet.
- It cannot know whether an MCP tool is safe to call; it only verifies discovery and startup.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). Useful contributions are real-world diagnostics from broken MCP setups, additional client config locations, and tests for server startup edge cases.

## License

MIT
