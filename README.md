# cc-edge-vscode-io

## Overview

Cribl Edge Pack that collects VS Code application logs, Copilot extension logs, user settings, and
extension inventory from the local filesystem. Includes pipeline enrichment to identify
Copilot-specific log sources.

## Pack Components

| Component | Type | Enabled | Description |
|---|---|---|---|
| `vscode-logs` | File Monitor | Yes | VS Code application and extension logs (recursive) |
| `vscode-settings` | File Monitor | No | VS Code user settings snapshots |
| `vscode-extensions` | File Monitor | No | VS Code installed extensions inventory |
| Copilot Source Enrichment | Pipeline (main) | Yes | Adds `copilot_source` field based on log file path |

## Data Sources

- **VS Code logs** — `$VSCODE_HOME/logs/` application and extension logs, scanned recursively (enabled)
- **VS Code settings** — `$VSCODE_HOME/User/*settings.json` snapshots (disabled by default)
- **VS Code extensions** — `$VSCODE_EXT_HOME/.vscode/*extensions.json` inventory (disabled by default)

## Data Contract

Events leave this pack tagged with a `datatype` metadata field; Cribl Stream maps datatypes to
Splunk sourcetypes/indexes per the table below. Knowledge objects for the sourcetypes ship in
[VisiCore_TA_AI_Observability](https://github.com/JacobPEvans/VisiCore_TA_AI_Observability) (v0.2.0+).

| Input | Datatype | Splunk sourcetype | Index | TA support |
|---|---|---|---|---|
| `vscode-logs` | `vscode-logs` | `vscode:logs` | `vscode` | ✓ (0.2.0+) |
| `vscode-settings` | `vscode-settings` | `vscode:settings` | `vscode` | ✓ (0.2.0+) |
| `vscode-extensions` | `vscode-extensions` | `vscode:extensions` | `vscode` | ✓ (0.2.0+) |

## Setup

### Environment Variables

Set these environment variables before starting Cribl Edge:

| Variable | Purpose | macOS | Linux | Windows |
|---|---|---|---|---|
| `VSCODE_HOME` | VS Code data directory | `~/Library/Application Support/Code` | `~/.config/Code` | `%APPDATA%\Code` |
| `VSCODE_EXT_HOME` | User home directory | `/Users/<user>` | `/home/<user>` | `C:\Users\<user>` |

### Pipeline Enrichment

The `main` pipeline adds a `copilot_source` field to events based on the source file path:

- `copilot-chat` — logs from `GitHub.copilot-chat` extension
- `copilot` — logs from `GitHub.copilot` extension
- `null` — all other VS Code logs

## Troubleshooting

- Verify `VSCODE_HOME` points to the correct VS Code data directory
- The `vscode-logs` input recursively scans `$VSCODE_HOME/logs/` — increase the polling interval from 30s if scan volume is too high
- Settings and extensions inputs are disabled by default — enable in Cribl Edge UI as needed

## Release Notes

### v1.0.2

- Docs: add Overview, Data Sources, and Data Contract sections
- Chore: normalize `package.json` metadata (trim author whitespace, pretty-print)

### v1.0.1

- Fix: disable `tailOnly` on JSON config inputs (`vscode-settings`, `vscode-extensions`) to prevent data loss on full-file rewrites

### v1.0.0

- Initial release
- 3 file monitor inputs (logs enabled, settings and extensions disabled)
- Copilot source enrichment pipeline
