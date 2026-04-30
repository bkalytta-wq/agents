---
"agents": patch
---

fix(mcp): forward large JSON-RPC payloads via the request body instead of the `cf-mcp-message` header

The streamable-HTTP transport in `agents/mcp` placed the entire JSON-RPC payload (base64-encoded) into the `cf-mcp-message` HTTP header on the internal Worker → Durable Object hop. Cloudflare's edge limits the combined request header size to ~32 KB; base64 inflates by ~33%, so any tool call with arguments larger than ~16 KB tripped the limit and surfaced as an opaque `TLS Alert: record_overflow` (alert 22) failure ~700 ms into the request, before the Worker handler ran.

This change keeps the existing header path as the fast path for small payloads (≤4 KB raw) and switches to a body-forwarding path for larger payloads: the front-door Worker pre-stashes the JSON-RPC payload via a separate POST to the Durable Object and passes only a short opaque id on the WebSocket upgrade. Fully backward compatible — header-mode is unchanged for the vast majority of tool calls.

Fixes the `TLS record_overflow` failure mode reported when MCP tools send tool arguments larger than ~15 KB (e.g. memory.create with substantial content).
