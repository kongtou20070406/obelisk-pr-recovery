# Obelisk CLI

The local Obelisk runtime used by coding agents. It indexes Claude Code, Codex, GitHub Copilot,
Kimi Code, and Pi transcripts into `~/.obelisk/obelisk.sqlite` and exposes the
stable `build`, `search`, `query`, and `attune` process interface.

```bash
npm install --global @obelisk-apps/cli
obelisk --version
obelisk install
obelisk --query /tmp/query.mjs
```

`obelisk install` installs the separate docs-only agent skill from
`tommy0103/obelisk-skill`. The CLI itself remains daemon-free: each command
refreshes the local index when write ownership is available, then exits.

### Read-only index errors

Search and query read from SQLite, but their pre-query refresh also needs write
access to `~/.obelisk/obelisk.sqlite`, its directory (including SQLite sidecars),
and the writer-lease file. A sandbox that permits reading the index but denies
these writes can therefore block a query before its script runs.

The CLI checks index writability before source discovery and stops on a shared
read-only writer failure instead of retrying every transcript. It reports the
index path and preserves the original SQLite error. No stale-result fallback,
automatic permission change, or alternative index is selected. If a host
sandbox is responsible, request host-approved permissions and retry the same
command with the same retrieval scope; persistent failures still exit nonzero.
