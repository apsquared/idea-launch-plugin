# Idea Launch — Agent Plugin

[Idea Launch](https://www.idea-launch.io) runs a small, managed ad test for a
SaaS or app idea, or for a product you already shipped, and returns a market
read: click-through rate, signups, a validation score and a recommended next
move. This plugin lets you prepare that test from Claude Code or Codex and read
the result back in the conversation. It connects the Idea Launch MCP server
(OAuth sign-in, no API key) and adds two skills: one for writing a brief worth
testing, one for reading the result honestly.

**Your agent prepares and reads. You start the paid test.** No tool can start,
boost or pay for a test; your agent hands you a link, and you review and start
the test in the Idea Launch web app. A small test measures how one audience
responds to one message and offer. It is evidence about that positioning, not
proof that a business will succeed.

Step-by-step setup for every client, with troubleshooting:
<https://www.idea-launch.io/mcp>

## Claude Code

In a Claude Code session:

```
/plugin marketplace add apsquared/idea-launch-plugin
/plugin install idea-launch@apsquared
```

Choose a scope when asked. If Claude Code says `Run /reload-plugins to apply`,
run `/reload-plugins`.

Then sign in: run `/mcp`, select `plugin:idea-launch:idea-launch`, and choose
**Authenticate**. Your browser opens the Idea Launch sign-in. Until you do,
`/mcp` and `claude mcp list` show the server as **Needs authentication** — the
expected state right after install. The skills load either way; the tools need
the sign-in.

On a machine without a browser:
`claude mcp login plugin:idea-launch:idea-launch --no-browser`, then paste the
redirect URL back when asked.

To sign out, use **Clear authentication** in `/mcp`. To remove the plugin,
`/plugin uninstall idea-launch@apsquared`.

## Codex

In your shell:

```
codex plugin marketplace add apsquared/idea-launch-plugin
codex plugin add idea-launch@apsquared
codex mcp login idea-launch
```

Codex does not start the sign-in on its own: until you run
`codex mcp login idea-launch`, `codex mcp list` shows the server as
**Not logged in**. Start a new Codex session afterwards; installed plugins load
in new sessions. The Codex IDE extension and the ChatGPT desktop app share the
same configuration (restart them after installing). Without a browser, add
`--no-browser` to the login command.

To sign out, `codex mcp logout idea-launch`. To remove the plugin,
`codex plugin remove idea-launch@apsquared`.

Codex reads this repository's `.claude-plugin/marketplace.json` as a
compatible marketplace, so both clients install from the same catalog.

## Check the connection

Ask your agent:

> Check my Idea Launch connection: how many Ad Runs do I have available, and
> which ideas are in my workspace? Don't create or change anything.

It calls `idealaunch_get_account` and `idealaunch_list_ideas` and reports your
available Ad Runs and your ideas (or that you have none yet). Nothing is
created, changed or charged.

Then try:

> I'm building [what it does, and who it's for]. Use Idea Launch to prepare a
> demand test: draft the brief with me, write the ad, and tell me what's still
> missing before I can start it in the app.

## Other clients

- **Claude chat** (claude.ai, the desktop app's chat, mobile — not Claude
  Code): add `https://www.idea-launch.io/api/mcp` as a custom connector under
  Customize > Connectors, then click Connect. You get the tools, not the skills.
- **Any Agent Plugins client** (VS Code, GitHub Copilot, Cursor and others):
  install this repository through the client's plugin support.
- **Any MCP client**: connect `https://www.idea-launch.io/api/mcp` over
  streamable HTTP and sign in with OAuth. How the sign-in starts depends on the
  client.
- **Skills only**, for an agent that reads `SKILL.md` files:
  `npx skills add apsquared/idea-launch-plugin`. This does not connect the
  server, which the skills need.

## The package

A portable [Agent Plugins](https://agent-plugins.org) v1.0.0 package with the
Claude Code layout alongside. Both formats read the same `skills/` directory,
unmodified.

```
agent-plugin/
├── plugin.json                          # Agent Plugins manifest (+ OpenAI listing metadata)
├── mcp.json                             # Agent Plugins MCP server (streamable-http), read by Codex
├── .claude-plugin/
│   ├── plugin.json                      # Claude Code manifest
│   └── marketplace.json                 # Marketplace entry (source ./), read by Claude Code and Codex
├── .mcp.json                            # Claude Code MCP server (http), Claude endpoint
├── assets/icon.png                      # Listing icon, 512×512
├── LICENSE                              # MIT
└── skills/
    ├── reading-a-market-read/SKILL.md   # how to interpret a result honestly
    └── setting-up-an-idea-test/SKILL.md # how to get a test worth paying for
```

The two MCP files describe the same server and carry no credentials. They
differ in the name each format gives the transport (Agent Plugins calls it
`streamable-http`, Claude Code calls it `http`) and in the URL: Claude Code
connects to `https://www.idea-launch.io/api/mcp/claude`, which has every tool
except AI image generation, because Anthropic's directory doesn't accept
connectors that generate images with AI. In Claude, add the hero image in the
Idea Launch web app, where you can upload one or generate one. Everywhere else
(`mcp.json`) the plugin connects to the full `https://www.idea-launch.io/api/mcp`.
Versions in `plugin.json`, `.claude-plugin/plugin.json` and
`.claude-plugin/marketplace.json` are kept identical by an automated check.

## Why a plugin and not just the MCP server

The MCP server exposes capability. It does not carry judgement, and that is the
part that decides whether a test is any good:

- A `validationScore` of 45 at low confidence, on a `traffic_backed` read, is
  easy to misreport as "mixed signal, refine the angle". It is closer to "not
  settled, and it cannot speak to demand at all".
- A `targetCustomer` of "founders" produces targeting that underperforms, and
  the user blames the product rather than the brief.

Both skills also restate the hard rules: an Ad Run is evidence about a
positioning rather than proof about a business, no agent can spend the user's
money, and spend or cost-per figures are never reported or estimated.

## Auth

Nothing to configure, and no API key. Neither `mcp.json` nor `.mcp.json`
carries credentials, by design. The server uses OAuth 2.1 with dynamic client
registration; each client starts the browser sign-in its own way (see above).

## Privacy and support

- Privacy policy: <https://www.idea-launch.io/privacy-policy>
- Terms of service: <https://www.idea-launch.io/tos>
- Support: <https://www.idea-launch.io/contact-us>

## License

MIT — see [LICENSE](./LICENSE). The package is manifests and skill documents;
the Idea Launch service itself is governed by its
[terms of service](https://www.idea-launch.io/tos).

## Conformance

This package targets Agent Plugins v1.0.0 and the Agent Skills specification,
and additionally ships the Claude Code plugin layout. Conformance is enforced
by an automated check on every change: the manifest's closed field set and name
rules, the `streamable-http` server shape, each skill's frontmatter, version
consistency across manifests, and that no MCP file carries credential-shaped
values, since this package is distributed publicly. `claude plugin validate
--strict` also passes.
