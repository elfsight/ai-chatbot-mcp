# Installing the Elfsight AI Chatbot MCP server (for AI agents)

This file is a machine-readable install guide: an agent capable of editing config files or
running shell commands can follow it directly.

## 1. Sign-in

The server uses OAuth. The user signs in to Elfsight in a browser; an agent cannot do that
step for them. There is no API key to ask for.

## 2. Configure the connection

- Server URL: `https://mcp.elfchatbot.com/mcp`
- Transport: Streamable HTTP
- Auth: OAuth, discovered by the client from the server's `401` response. Do not add an
  `Authorization` header yourself and do not put credentials in the URL.

Add it to the MCP settings of the client you are running in, not to another editor's config
found in the workspace. Use whatever mechanism your host application exposes for a remote HTTP
MCP server (see `README.md` in this repository for the usual config shapes).

Cline: add it to `cline_mcp_settings.json` and keep `"type": "streamableHttp"`; without it Cline
uses the legacy SSE transport and the connection fails. Recent Cline versions keep the file in
`~/.cline/data/settings/`, older ones in the editor's
`User/globalStorage/saoudrizwan.claude-dev/settings/`. Do not create `.vscode/mcp.json` for
Cline: that file is VS Code's built-in MCP config and Cline does not read it.

```json
{
  "mcpServers": {
    "elfsight-ai-chatbot": {
      "type": "streamableHttp",
      "url": "https://mcp.elfchatbot.com/mcp"
    }
  }
}
```

Then ask the user to finish
the browser sign-in when the client prompts for it — in Claude Code that is `/mcp` →
**Authenticate**.

## 3. Verify the connection

Call the `list_assistants` tool with no arguments. A working connection returns a JSON object
with an `items` array (the account's AI Chatbot widgets) and a `count`. Report the result to the
user, including the assistant names, so they can confirm this is the right account.

## 4. Common errors

- **401 Unauthorized** — the user has not signed in yet, or the session expired. Ask them to
  authenticate again in the client; do not retry in a loop.
- **429 Too Many Requests** — the rate limit was exceeded. The response includes a
  `Retry-After` value in seconds; wait at least that long before retrying.
- **`list_assistants` returns an empty `items` array** — the account has no AI Chatbot widgets
  set up yet. This is not a connection problem; every other tool will report the same until
  a widget exists.
- **A tool call error mentioning "assistant_id is required"** — the account has more than one
  AI Chatbot widget. Call `list_assistants` first and pass one of the returned `assistant_id`
  values to subsequent calls.

## 5. What not to do

- Do not guess or hardcode an `assistant_id` — always resolve it from `list_assistants` unless
  the account is known to have exactly one widget.
- Do not attempt to call any tool not listed in `tools/list`; the server exposes a fixed,
  versioned set.
- `answer_knowledge_gap` is the only tool that writes. Do not call it without the user's
  explicit confirmation of the question/answer text.
