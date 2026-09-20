# Chrome2api local inference provider

CodexPro wraps the text-only OpenAI-compatible HTTP surface documented by
[`xszwow/Chrome2api`](https://github.com/xszwow/Chrome2api). Chrome2api remains a
separate node-local process; CodexPro does not copy or redistribute its native
runner, Chrome runtime DLLs, model weights, cookies, or browser profile.

```text
authenticated ChatGPT connector
  -> CodexPro fabric(chrome_complete)
  -> validate bounded text request
  -> fixed literal-loopback endpoint
  -> Chrome2api /v1/chat/completions
  -> validate bounded JSON response
  -> request root + response root + receipt root
```

## Configuration

Start Chrome2api according to its own repository instructions. Its default API
address needs no CodexPro configuration:

```text
http://127.0.0.1:11435/v1
```

To use a different local port, set the startup environment variable before
starting CodexPro:

```powershell
$env:CODEXPRO_CHROME2API_URL = "http://127.0.0.1:11435/v1"
```

Only literal `127.0.0.1` and `[::1]` HTTP URLs ending at `/v1` are admitted.
The model or MCP caller cannot override the endpoint.

## MCP actions

The existing `fabric` tool provides:

- `chrome_contract` - returns the trust boundary and hard limits without
  contacting Chrome2api.
- `chrome_status` - calls `/v1/models` and verifies that
  `chrome-gemini-nano` is advertised.
- `chrome_complete` - sends one non-streaming text completion and returns a
  deterministic receipt binding the admitted request and returned text.

Example arguments:

```json
{
  "action": "chrome_complete",
  "system": "Answer concisely using only local inference.",
  "prompt": "Summarize the node inventory.",
  "max_tokens": 256,
  "temperature": 0.2,
  "timeout_ms": 120000
}
```

## Enforced boundary

- text only; no image, audio, data URL, or local-file fields;
- no arbitrary URL, redirects, Authorization header, cookies, or browser state;
- 65,536 prompt bytes, 16,384 system bytes, and 81,920 combined input bytes;
- 2,048 maximum requested output tokens;
- 256 KiB maximum HTTP response;
- 2-120 second deadline;
- unstable upstream IDs and timestamps are excluded from proof identity.

Chrome2api's own API has no authentication boundary. Keep it on loopback.
CodexPro authentication still protects the public MCP endpoint.

## Deliberate exclusions

The donor supports local image/audio paths and simulated SSE. Those surfaces
are not exposed because a public MCP caller must not be able to make the local
runtime read arbitrary files, and buffering a simulated stream adds no proof
value. Structured output, tools, embeddings, and Responses API are not claimed.
