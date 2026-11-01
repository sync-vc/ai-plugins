# Sync.VC AI plugins

Plugins for building on the Sync.VC Enterprise API.

| Plugin | What it does |
| --- | --- |
| `enterprise-api` | Teaches Claude to get a token, call the API and verify webhooks |

## Install

In Claude Code:

```
/plugin marketplace add sync-vc/ai-plugins
/plugin install enterprise-api@sync-vc
```

Or from your shell:

```bash
claude plugin marketplace add sync-vc/ai-plugins
claude plugin install enterprise-api@sync-vc
```

To update, run `/plugin marketplace update sync-vc`.
