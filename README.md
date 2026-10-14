# Elfsight AI Chatbot — MCP Server

> This repository holds the documentation and the registry manifest (`server.json`) of the
> Elfsight AI Chatbot MCP server. The server itself is hosted by Elfsight; there is nothing to
> build or run here. For help, contact [Elfsight Help Center](https://help.elfsight.com/).

A remote [Model Context Protocol](https://modelcontextprotocol.io) server over your
[Elfsight AI Chatbot](https://elfchatbot.com/) widget data. Point an MCP-capable
AI agent at it and it can read your conversations, leads, knowledge base and analytics —
and add answers to your knowledge base — without you copy-pasting anything into a chat window.

The server is account-scoped: once you sign in, the agent can reach every AI Chatbot widget on
your Elfsight account, and nothing else (no other Elfsight products or data).

Server URL: `https://widget-data.service.elfsight.com/mcp/ai_chat` (Streamable HTTP).

## Connect

Sign-in is OAuth. Add the server URL to your MCP client; the client opens a browser window,
you sign in to Elfsight and approve access. There is no key to copy.

### Claude Code

```bash
claude mcp add --transport http elfsight-ai-chatbot https://widget-data.service.elfsight.com/mcp/ai_chat
```

Then run `/mcp` inside Claude Code and choose **Authenticate** for `elfsight-ai-chatbot`.

### Cursor

This repository is also a [Cursor plugin](https://cursor.com/docs/plugins/building): the manifest
is in [`.cursor-plugin/plugin.json`](./.cursor-plugin/plugin.json), the server connection in
[`mcp.json`](./mcp.json), and a rule with usage notes for the agent in [`rules/`](./rules).

To connect without the plugin, add the server to `.cursor/mcp.json` and authenticate it when
Cursor prompts:

```json
{
  "mcpServers": {
    "elfsight-ai-chatbot": {
      "url": "https://widget-data.service.elfsight.com/mcp/ai_chat"
    }
  }
}
```

### Cline

In Cline, open **MCP Servers → Remote Servers** and add:

- Server Name: `elfsight-ai-chatbot`
- Server URL: `https://widget-data.service.elfsight.com/mcp/ai_chat`
- Transport: **Streamable HTTP**

Then choose **Authenticate** for the server and sign in to Elfsight in the browser.

The same in Cline's own settings file, `cline_mcp_settings.json`. Recent versions keep it in
`~/.cline/data/settings/`, older ones in the editor's
`User/globalStorage/saoudrizwan.claude-dev/settings/`. Keep `type`: without it Cline falls
back to the legacy SSE transport. `.vscode/mcp.json` belongs to VS Code's built-in MCP
support; Cline does not read it.

```json
{
  "mcpServers": {
    "elfsight-ai-chatbot": {
      "type": "streamableHttp",
      "url": "https://widget-data.service.elfsight.com/mcp/ai_chat"
    }
  }
}
```

### Other clients

Any client that supports remote MCP servers with OAuth (dynamic client registration and
PKCE) should connect with the server URL alone. **We have tested Claude Code, Cursor
(through the plugin), Cline and VS Code only**; the VS Code snippet below is verified, the
Windsurf one is the usual shape of its config.

VS Code (`.vscode/mcp.json`):

```json
{
  "servers": {
    "elfsight-ai-chatbot": {
      "type": "http",
      "url": "https://widget-data.service.elfsight.com/mcp/ai_chat"
    }
  }
}
```

Windsurf (`mcp_config.json`):

```json
{
  "mcpServers": {
    "elfsight-ai-chatbot": {
      "serverUrl": "https://widget-data.service.elfsight.com/mcp/ai_chat"
    }
  }
}
```

## What you can ask it

- **`list_assistants`** — "Which AI Chatbot widgets do I have?"
- **`get_analytics`** — "How did my AI Chatbot perform over the last 30 days?"
- **`search_conversations`** — "Show me conversations from last week about shipping."
- **`get_conversation`** — "What happened in conversation `<id>`?"
- **`list_leads`** — "List the new leads from this week that I haven't reviewed yet."
- **`search_knowledge`** — "Does my knowledge base say anything about refunds?"
- **`list_knowledge_gaps`** — "What do visitors ask that my assistant can't answer?"
- **`answer_knowledge_gap`** — "Add this as a knowledge base answer and close that gap."

`answer_knowledge_gap` is the only tool that writes. Three ready-made prompts wrap common
workflows end to end: `weekly_report`, `knowledge_gaps_review`, `lead_followup`.

## Privacy

Everything a tool returns can be sent to your AI provider (Anthropic, OpenAI, etc.) as part
of your conversation with the agent. Keep that in mind:

- **Lead contact fields are masked by default.** A lead's name, email and phone come back
  masked (e.g. `j***@example.com`, `+7***4567`). Pass `contacts: "full"` on `get_conversation`
  or `list_leads` to get the real values in that call.
- **Masking covers those fields only.** Message text, page URLs and attachment links are
  returned as they are. Visitors often type their email or phone into the chat, so
  `get_conversation` can show contact details even when the lead fields are masked. Treat
  everything a tool returns as possibly containing personal data.
- **Attachments are links, not content.** File attachments come back as URLs, so a tool call
  never puts a file's contents into the model's context.

Internal and test conversations are not filtered out of any tool — if you use your own widget
for testing, expect to see that traffic in the results too.

## Limits and rate limiting

- `search_conversations`, `list_leads`, `search_knowledge` and `list_knowledge_gaps` return
  20 items by default, 100 at most. A truncated result says so and reports the total.
  `search_conversations` and `list_leads` page with `cursor`; for the other two, narrow the
  query.
- `get_conversation` returns the 100 most recent messages by default, 500 at most.
- Periods are rolling windows ending now — `7d`, `30d` or `90d`. Arbitrary date ranges are
  not supported. `list_leads` also accepts no period at all, which means the whole history.
- Requests are rate-limited per account and client. Going over the limit returns a `429` with
  a `Retry-After` header — wait and retry.

## Support

Questions or issues — contact [Elfsight Help Center](https://help.elfsight.com/) or your
account manager.
