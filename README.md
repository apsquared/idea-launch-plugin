# Idea Launch — Agent Plugin

A portable [Agent Plugins](https://agent-plugins.org) v1.0.0 package that
connects the Idea Launch MCP server and adds the judgement needed to use it
well.

```
agent-plugin/
├── plugin.json                          # manifest
├── mcp.json                             # remote MCP server (streamable HTTP)
└── skills/
    ├── reading-a-market-read/SKILL.md   # how to interpret a result honestly
    └── setting-up-an-idea-test/SKILL.md # how to get a test worth paying for
```

## Why a plugin and not just the MCP server

The MCP server exposes capability. It does not carry judgement, and that is the
part that decides whether a test is any good:

- A `validationScore` of 45 at low confidence, on a `traffic_backed` read, is
  easy to misreport as "mixed signal, refine the angle". It is closer to "not
  settled, and it cannot speak to demand at all".
- A `targetCustomer` of "founders" produces targeting that underperforms, and
  the user blames the product rather than the brief.

Both skills also restate the two hard rules: an Ad Run is evidence about a
positioning rather than proof about a business, and no agent can spend the
user's money — starting a run is confirmed by the user in the web app.

## Installing

Install through any client that supports the Agent Plugins format; the client
handles discovery, installation, and permissions. Installing the plugin also
configures the MCP server, so there is no separate connection step.

For a client that does not support the format yet, connect the MCP server
directly — see <https://www.idea-launch.io/mcp>.

## Auth

None to configure. `mcp.json` carries no credentials by design. The first call
opens a browser sign-in over OAuth 2.1 with dynamic client registration.

## Conformance

`tests/agent-plugin.smoke.test.ts` in the main repository encodes the normative
constraints from the Agent Plugins and Agent Skills specifications, so the
package cannot drift out of conformance without a test failing.
