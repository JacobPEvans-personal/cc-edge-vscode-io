# cc-edge-vscode-io

Cribl Edge Pack that collects VS Code application logs, Copilot extension
logs, user settings, and extension inventory from the local filesystem.
Includes pipeline enrichment to identify Copilot-specific log sources.

## Pack Components

| Component | Type | Enabled | Description |
|---|---|---|---|
| `vscode-logs` | File Monitor | Yes | VS Code application and extension logs (recursive) |
| `vscode-settings` | File Monitor | No | VS Code user settings snapshots |
| `vscode-extensions` | File Monitor | No | VS Code installed extensions inventory |
| Copilot Source Enrichment | Pipeline (main) | Yes | Adds `copilot_source` field based on log file path |

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

## Deployment

Deployment to Cribl Edge nodes is automated by the
`cribl_packs` role in
[ansible-proxmox-apps](https://github.com/JacobPEvans/ansible-proxmox-apps/tree/main/roles/cribl_packs).
Pack version is pinned in `roles/cribl_packs/defaults/main.yml`.

To roll out a new release:

1. Cut a tag in this repo (publishes the `.crbl` asset).
2. Bump `version:` for `cc-edge-vscode-io` in `roles/cribl_packs/defaults/main.yml`.
3. Run `ansible-playbook playbooks/site.yml --tags cribl_packs` from that repo.

The role downloads the matching `.crbl`, unpacks it into
`/opt/cribl/local/edge/packs/cc-edge-vscode-io/`, and restarts
`cribl-edge.service` only when the version actually changed.

## Release Notes

### v1.0.1

- Fix: disable `tailOnly` on JSON config inputs (`vscode-settings`, `vscode-extensions`) to prevent data loss on full-file rewrites

### v1.0.0

- Initial release
- 3 file monitor inputs (logs enabled, settings and extensions disabled)
- Copilot source enrichment pipeline
