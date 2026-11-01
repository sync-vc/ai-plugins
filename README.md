# Sync.VC Claude Code plugins

A [Claude Code plugin marketplace](https://code.claude.com/docs/en/plugin-marketplaces) for building on the Sync.VC API.

| Plugin | What it does |
| --- | --- |
| `enterprise-api` | Build against the Sync.VC Enterprise API |

## Install

In a Claude Code session:

```
/plugin marketplace add sync-vc/claude-plugins
/plugin install enterprise-api@sync-vc
```

Or from your shell:

```bash
claude plugin marketplace add sync-vc/claude-plugins
claude plugin install enterprise-api@sync-vc
```

To pick up later changes, run `/plugin marketplace update sync-vc`, or turn on auto-update for the marketplace under **Marketplaces** in `/plugin`.
