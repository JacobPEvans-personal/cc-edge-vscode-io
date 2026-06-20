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

Each event leaves this pack tagged with a `datatype` metadata field. Downstream a Cribl Stream
worker maps that `datatype` to a Splunk sourcetype and index per the table below. This is the
stable contract the pack guarantees — any consumer that provides matching sourcetype/index
definitions can ingest the output without changes here.

| Input | Datatype | Splunk sourcetype | Index |
|---|---|---|---|
| `vscode-logs` | `vscode-logs` | `vscode:logs` | `vscode` |
| `vscode-settings` | `vscode-settings` | `vscode:settings` | `vscode` |
| `vscode-extensions` | `vscode-extensions` | `vscode:extensions` | `vscode` |

Consumers are responsible for defining the `vscode:*` sourcetypes and the `vscode` index in their
Splunk environment; this pack emits the `datatype` tag and leaves index/sourcetype resolution to
the receiving Stream worker.

## Installation

Install this pack into a Cribl Edge node, then set the environment variables below before starting
Cribl Edge so the file-monitor inputs can resolve their source paths:

| Variable | Purpose | macOS | Linux | Windows |
|---|---|---|---|---|
| `VSCODE_HOME` | VS Code data directory | `~/Library/Application Support/Code` | `~/.config/Code` | `%APPDATA%\Code` |
| `VSCODE_EXT_HOME` | User home directory | `/Users/<user>` | `/home/<user>` | `C:\Users\<user>` |

For example, on macOS:

```bash
export VSCODE_HOME="$HOME/Library/Application Support/Code"
export VSCODE_EXT_HOME="$HOME"
```

## Usage

The `vscode-logs` input is enabled by default and begins collecting on start. The
`vscode-settings` and `vscode-extensions` inputs ship disabled — enable them in the Cribl Edge UI
when you want settings snapshots or extension inventory.

The `main` pipeline enriches every event with a `copilot_source` field derived from the source
file path:

- `copilot-chat` — logs from the `GitHub.copilot-chat` extension
- `copilot` — logs from the `GitHub.copilot` extension
- `null` — all other VS Code logs

Events are emitted tagged with the `datatype` metadata field described in
[Data Contract](#data-contract) for downstream routing.

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

---

> Part of a [larger ecosystem of ~40 repos](https://docs.jacobpevans.com) — see how it all fits together.
