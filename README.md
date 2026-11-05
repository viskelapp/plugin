<p>
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/logo-dark.svg">
    <img src="assets/logo.svg" alt="Viskel" height="64">
  </picture>
</p>

# Viskel plugin for Claude

Design on the [Viskel](https://app.viskel.com) canvas from Claude. The plugin connects Claude to the Viskel MCP server and adds skills for the common jobs: building and editing boards, working through review comments, organising sections for handoff, writing collections, and importing claude.ai Design canvases.

- The app: [veebisepad/viskel](https://github.com/veebisepad/viskel)
- This plugin: [viskelapp/plugin](https://github.com/viskelapp/plugin)

## Requirements

An existing account on https://app.viskel.com. Signing in through Claude never creates an account; ask a workspace owner for an invitation.

## Install

### Claude Code

```
/plugin marketplace add viskelapp/plugin
/plugin install viskel@viskelapp
```

Then run `/mcp`, pick the `viskel` server of the plugin and sign in.

### claude.ai and Claude Desktop

1. Customize > Plugins > Add > Add marketplace: paste `https://github.com/viskelapp/plugin`, then install Viskel.
2. Customize > Plugins > Viskel > Connectors tab: Add (if it shows Not added), then Connect, and sign in. On Team/Enterprise, an Owner must add it.

## Signing in and out

The plugin connects to `https://app.viskel.com/mcp` with OAuth: Claude registers itself, you sign in to Viskel in the browser and approve it. No token to copy. An access token lasts an hour and Claude refreshes it for up to 30 days of inactivity.

To sign out, disconnect the app at https://app.viskel.com/settings/connected-apps, or clear the authentication in `/mcp` (Claude Code) or Customize > Connectors (claude.ai and Claude Desktop).

## Skills

| Skill | Use it to |
| --- | --- |
| `/viskel:design` | Create a design or change boards: plan, build, validate, snapshot, share the links. |
| `/viskel:comments` | Work through review comments: fix, reply, resolve, ask for decisions. |
| `/viskel:sections` | Group boards into sections, label them, link Jira issues and hand off to development. |
| `/viskel:collections` | Write a collection of self-contained HTML artifact pages. |
| `/viskel:import-claude-design` | Bring a claude.ai Design canvas into Viskel as a new design. |

Claude picks the right skill from what you ask; the slash commands start one directly.

## Moving from a hand-added server

If you added Viskel yourself with `claude mcp add`, remove it so the tools do not show up twice:

```
claude mcp remove viskel
```

The tool names change prefix: `mcp__viskel__*` becomes `mcp__plugin_viskel_viskel__*`. Update any `permissions.allow` entries in your settings to match.

## Privacy

The plugin is configuration only: it runs no code on your machine. Claude talks to the Viskel server at app.viskel.com over HTTPS, as you, with the permissions your account has. What Claude reads and writes there is stored in Viskel like any other change you make, with Claude named as the client in the history. The plugin collects nothing else. Disconnect at any time at https://app.viskel.com/settings/connected-apps.

## Updating

Claude Code: run `/plugin marketplace update viskelapp`, then in `/plugin` > Installed > Viskel choose Update now (or run `claude plugin update viskel@viskelapp` in a shell). Auto-update is off by default for third-party marketplaces; turn it on for `viskelapp` in the Marketplaces tab of `/plugin`. On claude.ai, update it from Customize > Plugins.

## Troubleshooting

- **Tools missing or "unauthorized"**: sign in again in `/mcp` or Customize > Connectors.
- **Sign-in refused**: you need an existing Viskel account; OAuth does not create one.
- **"Too many requests" when connecting**: the server allows ten client registrations per hour from one address. Wait an hour and connect again.
- **Two sets of Viskel tools**: remove the hand-added server (see above).

## Development

```
claude plugin validate --strict .
claude plugin marketplace add /path/to/plugin
claude plugin install viskel@viskelapp
```

Skills hold workflow only. The server's own instructions, each design's `CLAUDE.md` and each page's `PAGE.md` hold the rules, so skills don't repeat them. Tool names in skills are bare (`read_file`). `skills/design/references/board-format.md` records the viskel commit it was derived from; refresh it when the board format changes.

## License

MIT, see [LICENSE](LICENSE).
