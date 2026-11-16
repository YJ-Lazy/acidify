# SnowLuma migration spike

Status: investigation only; no runtime code has been changed.

## Goal

Evaluate Acidify/Yogurt as an independent QQ protocol process for a future SnowLuma-compatible runtime. Keep the protocol process and the TypeScript application separated by the documented Milky HTTP/WebSocket boundary.

## Findings verified on this branch

- `yogurt` depends on both `:acidify-core` and `:acidify-milky`.
- Yogurt starts a Ktor CIO server using the configured Milky host and port.
- The configured HTTP prefix is mounted before `configureMilky()`.
- `configureMilky()` mounts:
  - `/api` for API operations,
  - `/event` for WebSocket and SSE event streams,
  - webhook delivery.
- The server supports an HTTP access token. Keep it enabled for any non-loopback listener.
- Startup requires a configured `signApiUrl`; this is an external runtime dependency and must be validated before calling the migration viable.
- The README documents JVM, Windows Native, Linux x64/ARM64, macOS ARM64 and Node.js support for the relevant Acidify modules.

## Proposed process boundary

```text
QQ account
   |
Yogurt (Acidify protocol runtime)
   |  Milky HTTP API + WebSocket events
Independent adapter (new code; no SnowLuma source copied)
   |
OneBot v11 compatibility surface
   |
Existing bot clients / automation
```

Do not embed Yogurt into the TypeScript process for the first spike. A process boundary makes crashes, upgrades, protocol lifecycle and licensing boundaries easier to reason about.

## Compatibility work required

1. Pin the exact Yogurt build and record its protocol/core commit hashes.
2. Verify QR login, login expiry, reconnect and clean shutdown on Windows x64.
3. Verify Milky event delivery over WebSocket and authenticated API requests.
4. Map Milky friend/group message events into a neutral internal event model.
5. Map the required send-message operations and message segments.
6. Add an independently implemented OneBot v11 adapter only after the required Milky operations and event schemas have been confirmed.
7. Exercise multi-account isolation, event ordering, duplicate delivery, media handling and reconnect behavior.
8. Add Windows packaging and a smoke-test workflow after the runtime contract is stable.

## Acceptance criteria

- No official QQ desktop process or injected native addon is required by the protocol runtime.
- Login and message send/receive succeed using a test account under the supported login flow.
- HTTP API access is authenticated; event transport is not exposed to an untrusted network.
- Reconnect does not duplicate active sessions or silently lose event subscriptions.
- A compatibility test suite covers the OneBot actions/events actually used by existing consumers.
- Runtime and adapter can be upgraded independently.

## Not in scope for this spike

- Bypassing account verification, risk controls or login challenges.
- Copying or adapting SnowLuma source files into this repository.
- Publishing a SnowLuma-derived modification. The SnowLuma repository's license requires prior written permission for public distribution of modified or derivative versions; obtain that permission before making changes to its public test branch.

## Current blocker

The exact Milky API action names and serialized event schemas still need to be inspected from the matching Milky specification and Acidify source. Do not implement a guessed OneBot mapping. The next engineering step is to capture a real authenticated Yogurt session and build contract fixtures from documented Milky payloads.
