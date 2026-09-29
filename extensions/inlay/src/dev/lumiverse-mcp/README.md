# Lumiverse MCP live integration test driver

A small stdio MCP server that lets an AI harness drive one realistic live test
against a running Lumiverse instance and inspect how the **Inlay Illustrator**
extension reacts. It is deliberately **not** a general Lumiverse automation
platform: it only knows how to authenticate, list/select characters, create a
test chat, inspect character cards and Inlay settings, safely patch/reset
settings, dry-run prompt assembly, send one full generation turn, and read the
stored result—including the exact image prompt shown by the Inlay lightbox—back.

```
src/dev/lumiverse-mcp/
├── client.ts         # thin HTTP client for Lumiverse's existing API
├── inlay-markers.ts  # detection of Inlay markup (reuses backend exports)
├── tools.ts          # tool logic (plain functions, unit-testable)
├── server.ts         # stdio MCP server (MCP protocol on stdout only)
├── smoke.ts          # opt-in live smoke CLI (same client module)
├── client.test.ts    # mocked-fetch client tests
├── tools.test.ts     # tool logic + MCP stdout-isolation tests
└── README.md         # this document
```

## 1. Configure credentials

Credentials come from environment variables only — never from MCP tool
arguments, and they are never printed or written into result artifacts.

| Variable             | Default               | Purpose                          |
| -------------------- | --------------------- | -------------------------------- |
| `LUMIVERSE_URL`      | `http://localhost:7860` | Base URL of the Lumiverse instance |
| `LUMIVERSE_USERNAME` | *(none)*              | Lumiverse account name           |
| `LUMIVERSE_PASSWORD` | *(none)*              | Lumiverse account password       |

Authentication follows the same sequence as the parent repository's
`scripts/migrate-sillytavern.ts`:

1. `POST /api/auth/sign-in/email` with `<username>@lumiverse.local`,
2. then with the raw username,
3. the `better-auth.session_token` is captured from `Set-Cookie` (fallback: a
   `token` field in the response body),
4. the resulting cookie / bearer token is sent on every subsequent request.

## 2. Start the MCP server

```bash
bun install                       # once, after cloning (adds @modelcontextprotocol/sdk)
bun run dev:mcp                   # starts the stdio MCP server
```

The server writes **only MCP protocol data to stdout** and diagnostics to
stderr. Enable its own diagnostics with:

```bash
LUMIVERSE_MCP_DEBUG=1 bun run dev:mcp
```

## 3. Example harness configuration

Point your MCP client at:

```json
{
  "mcpServers": {
    "inlay-illustrator": {
      "command": "bun",
      "args": ["run", "dev:mcp"],
      "cwd": "C:/Users/eme4/Lumiverse/inlay-illustrator",
      "env": {
        "LUMIVERSE_URL": "http://localhost:7860",
        "LUMIVERSE_USERNAME": "your-user",
        "LUMIVERSE_PASSWORD": "your-password",
        "LUMIVERSE_MCP_DEBUG": "1"
      }
    }
  }
}
```

`bun run dev:mcp` resolves relative to the repository root, so `cwd` must point
at this repository (or use an absolute path to `src/dev/lumiverse-mcp/server.ts`).

## 4. Inspect or change Inlay settings

The settings tools use Lumiverse's authenticated WebSocket bridge—the same
extension channel used by the Inlay frontend—and expose only fixed,
allowlisted operations:

- `inlay_describe_config`: current/default/type/enum metadata for every field.
- `inlay_patch_config`: partial patch with an optional non-persisting `dry_run`.
- `inlay_reset_config`: reset selected fields, or all fields with explicit
  `all=true` and `confirm_all=true`.
- `inlay_get_character_tags`: durable generated appearance tags for a chat.
- `inlay_get_image_details`: exact positive/negative prompt and perspective
  shown when a generated image is clicked.

The driver never exposes arbitrary extension messages or extension storage.

## 5. Select a character and create a test chat

Full character records (system prompt, description, personality, scenario,
first message, example messages, creator notes, post-history instructions,
tags, …) are available through `lumiverse_get_character`:

```
<use_mcp_tool>
<server_name>inlay-illustrator-live</server_name>
<tool_name>lumiverse_get_character</tool_name>
<arguments>{"character_id": "1a831e02-8d7a-4fcb-9201-addac329db53"}</arguments>
</use_mcp_tool>
```

Long text fields are capped at 10 000 chars and arrays at 50 items; the
response lists which fields were truncated in `truncated_fields`.

1. `lumiverse_list_characters` — find a character (`search`, `limit` ≤ 200,
   `offset`). Returns compact records: `id`, `name`, `tags`, `updated_at`,
   `image_id`.
2. `lumiverse_select_character` — validates the character exists and stores its
   ID as the driver's selected character (test-driver state only; the open
   Lumiverse browser tab is never navigated).
3. `lumiverse_create_test_chat` — creates a normal chat via
   `POST /api/v1/chats`, lets Lumiverse's chat route perform its normal
   greeting processing, stores the new chat ID, and returns the chat plus the
   initial greeting messages.

## 6. Run a dry run

`lumiverse_dry_run` calls `POST /api/v1/generate/dry-run` and returns the
assembled messages, assembly breakdown, model/provider, parameters, usage and
token statistics (messages are truncated to keep results bounded).

**Why this does not execute Inlay:** Inlay Illustrator starts its automatic
pipeline from the `GENERATION_ENDED` handler in `src/backend.ts`. A dry run
only assembles the prompt and runs registered interceptors (`stripInlayFromMessages`);
it does not emit `GENERATION_ENDED`, so Inlay's parser/image pipeline never
runs. Use dry run to validate prompt assembly and the interceptor — not the
illustration pipeline.

## 7. Send a full test turn and wait for Inlay

`inlay_send_test_turn` performs the same essential sequence as Lumiverse's
InputArea:

1. `POST /api/v1/chats/:chatId/messages` with
   `{ "is_user": true, "name": "<user_name>", "content": "<content>" }`,
2. `POST /api/v1/generate` with
   `{ chat_id, user_input, connection_id?, persona_id?, preset_id?, "generation_type": "normal" }`,
3. poll `GET /api/v1/generate/status/:chatId` until `completed`, `stopped`,
   `error`, or `generation_timeout_ms` (default 120 s),
4. fetch the chat tail and identify the new assistant message,
5. keep polling that assistant message until Inlay markup appears (the
   `<!-- inlay_illustrator -->` block or `data-inlay-illustrator="true"`), an
   Inlay error/status can be inferred, or `inlay_timeout_ms` (default 300 s)
   expires.

Completion of the main text generation is **not** treated as completion of
Inlay's asynchronous image pipeline. The tool returns the user message ID,
generation ID, assistant message ID/text, whether Inlay output was detected,
discovered image IDs/URLs, elapsed generation and Inlay times, the terminal
status, and a concise timeout/error explanation.

Compatibility note: some Lumiverse versions do not expose the single-message
GET route (`/chats/:chatId/messages/:messageId`); the client automatically
falls back to the tail-list endpoint and scans for the message ID, so the
driver works on both versions.

The driver also listens to the extension's live status channel while polling:
when Inlay's pipeline fails after `GENERATION_ENDED` (for example a parser
error), `inlay_send_test_turn` reports `inlay_status: "error"` with the exact
extension error instead of timing out silently.

`inlay_get_result` re-reads a stored assistant message (latest by default) and
returns the clean narrative text (with Inlay presentation markup stripped via
the production `stripInlayContent` helper), whether markup exists, image
IDs/URLs, a compact representation of the Inlay blocks, and the Inlay-owned
metadata (`extra.spindle_metadata.inlayIllustrator*`). Large payloads are
bounded: narrative text is capped at 20 000 characters and messages at 2 000
characters, each flagged with a `truncated` flag. Base64 payloads are never
returned.

## 8. Inspect Inlay logs

The driver cannot attach to the stdout of an already-running PowerShell
process; that is unreliable on Windows and out of scope. Instead:

1. Open the Inlay Illustrator settings UI and enable its **debugLogging**
   option, or set it through the extension's config.
2. Production logs are emitted as `[Inlay:<stage>]` messages (for example
   `[Inlay:generation_ended_event]`, `[Inlay:json_parse_done]`,
   `[Inlay:auto_generation_error]`). Look for them in the Lumiverse host's
   extension/console log output.
3. The MCP driver reports HTTP requests, polling stages, timeouts, and
   detected message state to **stderr** when `LUMIVERSE_MCP_DEBUG=1` is set —
   never on MCP stdout.

## 9. Opt-in live smoke command

For a quick manual check against a live instance without an MCP client:

```bash
LUMIVERSE_USERNAME=... LUMIVERSE_PASSWORD=... bun run dev:mcp:smoke
# optional: also send one full generation turn
LUMIVERSE_USERNAME=... LUMIVERSE_PASSWORD=... bun run dev:mcp:smoke --turn "Hello!" --inlay-timeout 300000
```

The smoke CLI authenticates, prints status, lists characters, selects the
first one, creates a test chat, and runs a dry run. With `--turn` it then
sends a full turn and waits for Inlay output. It never runs as part of
`bun test`.

## 10. Out of scope

- Browser navigation of the open Lumiverse tab (selection is test-driver state
  only).
- Attaching to the stdout of an already-running PowerShell process.
- Direct calls to Inlay's sidecar parser endpoint (the live test uses normal
  chat generation so Inlay resolves its configured parser connection through
  `config.parserConnectionId` and `spindle.connections.get()` naturally).
- Any Lumiverse database access or modification of Lumiverse itself.

## Tests

```bash
bun test src/dev/lumiverse-mcp   # mocked fetch; no live network calls
bun run typecheck
bun run build
```

Covered: Set-Cookie auth, body-token auth, auth failure without password
leakage, stale-session re-auth, character selection, chat creation, missing
selected character/chat errors, dry-run request construction, message creation
followed by generation, generation polling completion/timeout/stop/error,
Inlay result polling after main generation, Inlay markup/image detection
(against output from the production `renderInlaidMessage`), MCP stdout
isolation (spawns the real server), and bounded response sizes.
