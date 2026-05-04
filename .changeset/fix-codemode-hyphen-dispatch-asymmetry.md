---
"@cloudflare/codemode": patch
---

fix(codemode): accept raw hyphenated tool names in runtime dispatch

`DynamicWorkerExecutor` previously sanitized provider keys at registration but
the sandbox proxy still passed the raw lookup key (`codemode["foo-bar"]({})`)
through to the dispatcher unchanged, producing `Tool not found` errors for
callers that obtained tool names via discovery. The dispatcher now registers
each provided tool under both its raw name and its sanitized identifier, so
both `codemode.foo_bar({})` and `codemode["foo-bar"]({})` resolve to the
same underlying function. Closes a follow-up gap left by #1117 / #806.
