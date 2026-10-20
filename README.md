# AI Reserve for GitHub Copilot

Two things you can do with GitHub Copilot and AI Reserve, and most teams do
both:

1. **Run Copilot on AI Reserve models.** Our models in Copilot's own model
   picker, for Chat, agent mode and edits. Any AI Reserve key works and is
   what pays: whatever the key belongs to (a developer, a team, the
   organization) is debited, with its spend caps and model rules applied.
   Three ways in, differing only in who sets them up: the **AI Reserve VS
   Code extension** (each developer, one minute), **organization-wide custom
   models** (the GitHub admin, once; needs GitHub Enterprise; covers
   github.com chat too), or the manual **Custom Endpoint / Copilot CLI**
   variables. Step by step in `ADMIN-INSTALL-GUIDE.md` and on
   https://docs.aireserve.com/connect/github-copilot.
2. **The AI Reserve assistant in Copilot Chat** (the rest of this package).
   Tools inside the Copilot developers already use: ask any of our models by
   name ("using fable 5.1 ping me"), see prices, check usage and budget,
   manage API keys. Sign in with the portal account, nothing to paste.

GitHub retired App-based Copilot Extensions (the `@vendor` apps) on
10 November 2025 in favor of the Model Context Protocol. The assistant is
therefore not an app bundle to upload: it is the install surface for the
[AI Reserve MCP server](https://docs.aireserve.com/connect/mcp), which
Copilot reads directly. The server is live at `https://api.aireserve.com/mcp`
and signs users in over OAuth (dynamic client registration), so there is no
key to distribute for IDE use.

## Where it works

Our models in the picker (1):

| Surface                                            | How it connects                                                                                               | Who sets it up             |
| -------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- | -------------------------- |
| VS Code, VS Code Insiders                          | the AI Reserve extension (`.vsix` today, Marketplace listing pending), paste a key once                       | each developer, one minute |
| Every client, including Copilot Chat on github.com | the GitHub admin adds AI Reserve once as an OpenAI-compatible custom model provider (needs GitHub Enterprise) | the GitHub admin, once     |
| VS Code without the extension, Copilot CLI         | VS Code Custom Endpoint file, or the CLI's `COPILOT_PROVIDER_*` variables                                     | each developer             |

The assistant (2):

| Surface                                         | How it connects                                                                                                                            | Who sets it up                                            |
| ----------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------- |
| VS Code, VS Code Insiders                       | one-click link or `.vscode/mcp.json`, OAuth sign-in                                                                                        | each developer (or commit `.vscode/mcp.json` to the repo) |
| JetBrains IDEs, Visual Studio, Eclipse, Xcode   | same `servers` JSON in the Copilot MCP settings, OAuth sign-in                                                                             | each developer                                            |
| Copilot CLI and the GitHub Copilot desktop app  | `copilot mcp add …`, OAuth sign-in; the app reads the CLI's servers                                                                        | each developer                                            |
| github.com: Copilot cloud agent and code review | repository MCP configuration with a token in a `COPILOT_MCP_*` secret                                                                      | repository admin                                          |
| github.com: Copilot Chat                        | not possible for any vendor (GitHub only wires its own MCP server there); use AI Reserve models in the picker instead, see the admin guide | GitHub admin                                              |

Copilot Business and Enterprise: a GitHub admin must switch on the
**MCP servers in Copilot** policy once (it is off by default). Everything
else in the assistant table works on Copilot Pro and Free without admin
involvement.

## What is in the package

| file                          | role                                                                                                                                                                                              |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `vscode/mcp.json`             | Workspace or user-profile configuration for VS Code; the same `servers` shape JetBrains, Visual Studio, Eclipse and Xcode read                                                                    |
| `copilot-cli/mcp-config.json` | `~/.copilot/mcp-config.json` for the Copilot CLI and desktop app; the same `mcpServers` shape works as a repository-root `.mcp.json`                                                              |
| `cloud-agent/mcp.json`        | Repository → Settings → Copilot → MCP servers, for the cloud agent and code review                                                                                                                |
| `registry/server.json`        | The server's entry in the MCP registry format, published to the official MCP Registry as `io.github.Audacity-Investments/aireserve` 1.0.0; paste it into a company MCP registry allowlist as well |
| `ADMIN-INSTALL-GUIDE.md`      | The rollout guide for the customer's GitHub administrator                                                                                                                                         |

No credential lives in this package. IDE users sign in; the cloud agent's
token is minted in the portal by whoever configures that repository and
stored as a GitHub Agents secret.

## Install

### VS Code

[![Add to VS Code](https://img.shields.io/badge/VS_Code-Add_AI_Reserve-0098FF?style=flat-square&logo=githubcopilot&logoColor=white)](https://insiders.vscode.dev/redirect?url=vscode%3Amcp%2Finstall%3F%257B%2522name%2522%253A%2522aireserve%2522%252C%2522type%2522%253A%2522http%2522%252C%2522url%2522%253A%2522https%253A%252F%252Fapi.aireserve.com%252Fmcp%2522%257D)
[![Add to VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-Add_AI_Reserve-24bfa5?style=flat-square&logo=githubcopilot&logoColor=white)](https://insiders.vscode.dev/redirect?url=vscode-insiders%3Amcp%2Finstall%3F%257B%2522name%2522%253A%2522aireserve%2522%252C%2522type%2522%253A%2522http%2522%252C%2522url%2522%253A%2522https%253A%252F%252Fapi.aireserve.com%252Fmcp%2522%257D)

Click **Install** when VS Code asks, then **Allow** when it asks to sign in to
AI Reserve; a browser tab signs you in with your portal account. Or copy
`vscode/mcp.json` to `.vscode/mcp.json` in the workspace (commit it and every
clone gets the assistant) or to your user profile's `mcp.json`. VS Code 1.99
or newer; verify with **MCP: List Servers**.

### JetBrains, Visual Studio, Eclipse, Xcode

Open Copilot Chat in **Agent** mode, click the tools icon (_Configure your
MCP server_) → **Add MCP Tools**, paste the contents of `vscode/mcp.json`, and
click **Allow** on the sign-in prompt. In Visual Studio, click **Auth** in the
CodeLens above the server if the prompt does not appear.

### Copilot CLI and the GitHub Copilot app

```shell
copilot mcp add --transport http aireserve https://api.aireserve.com/mcp
```

The first use opens the sign-in in your browser (Copilot CLI 1.0.11 or
newer). The GitHub Copilot desktop app picks the server up automatically; to
add it from the app instead, open **Customize → MCP → Add custom server**,
choose HTTP, paste `https://api.aireserve.com/mcp`, leave the headers empty
and sign in when prompted. `copilot-cli/mcp-config.json` is the equivalent
file form.

### github.com: Copilot cloud agent and code review

The cloud agent runs without a browser, so it carries a token:

1. In the AI Reserve portal, open **API Keys → Management Keys** and mint an
   MCP access token with the **Messages** scope
   (`https://portal.aireserve.com/keys?type=mcp`).
2. In the repository, add it as an Agents secret named
   `COPILOT_MCP_AIRESERVE_TOKEN` (Settings → Environments → `copilot`).
   GitHub exposes only `COPILOT_MCP_*` names to MCP servers.
3. Paste `cloud-agent/mcp.json` under Settings → Copilot → MCP servers and
   save. Tools are allowlisted explicitly because the agent calls them without
   asking first; `send-message` spends from the token owner's budget.

## Try it

In Copilot Chat, switch to **Agent** mode and type:

```text
using fable 5.1 ping me
```

Copilot resolves the model with `list-models`, sends the message with
`send-message`, and answers with the reply and the exact model that produced
it: `Model: claude-fable-5-1`. The same loop is one command,
`/mcp.aireserve.ask-model`, and `#send-message` pins the tool. Other asks that
work out of the box: "which models can I use and what do they cost", "what did
my team spend this week", "how much budget do I have left", "list my API
keys". Each answer is scoped to what the signed-in person can already see in
the portal; anything that spends money or changes the account asks for
confirmation. Full surface: [MCP tool reference](https://docs.aireserve.com/connect/mcp-tools).

## Models in the Copilot picker (the other half)

Separately from the assistant, two ways to put our models in Copilot's own
model picker:

- **Each developer, one minute:** install the AI Reserve VS Code extension
  (download the `.vsix` from `https://portal.aireserve.com/connect/copilot`
  until the Marketplace listing is live), open the picker → **Manage Models**
  → **AI Reserve**, paste any key from the portal's API Keys page, and say
  yes when it offers to make AI Reserve the default for new chats. New
  catalog models show up on their own; the list refreshes every five minutes.
- **The GitHub admin, once (GitHub Enterprise):** add AI Reserve as an
  OpenAI-compatible **custom model** provider so our models appear in every
  developer's picker, including Copilot Chat on github.com, with nothing to
  install. The model list there is the admin's: when AI Reserve adds a model,
  add its id to that list.

Both are step by step in `ADMIN-INSTALL-GUIDE.md` and on
`https://docs.aireserve.com/connect/github-copilot`.

## Publishing to the registries

`registry/server.json` is the manifest both registries consume, named
`io.github.Audacity-Investments/aireserve` so the repository's own GitHub
Actions OIDC identity authorizes the publish (no secret, no PAT). The
`MCP Registry publish` workflow (`.github/workflows/mcp-registry-publish.yml`)
validates the manifest against the live registry on every run and, in
`publish` mode, publishes it to the official MCP Registry
(registry.modelcontextprotocol.io), which VS Code's MCP gallery and the GitHub
Copilot app's catalog read from. Version 1.0.0 is published and active; bump
`version` to publish again.

Listing on GitHub's MCP Registry page (github.com/mcp) is a manual curation
step on GitHub's side after that publish; the workflow's job summary prints
the onboarding request to post in the github/github-mcp-server discussions
(or to send to partnerships@github.com). Once onboarded, later versions sync
automatically. Until then, a customer that restricts MCP to an approved
registry adds this entry to their own. The same files are served publicly
from the docs site (`https://docs.aireserve.com/files/github-copilot/server.json`,
`mcp.json`, `mcp-config.json`, `cloud-agent-mcp.json`) for customers without
access to this repository.
