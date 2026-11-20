# Notes for coding agents and contributors

Read `docs/SPEC.md`, `docs/ARCHITECTURE.md` and `docs/GLINET_RPC.md` before writing code.

## Code ownership
AI agents own this codebase: they write, review, merge and maintain it under this repo's gates and
invariants, and there is no human developer who will catch what you miss. The owner sets product direction
and judges results, not code. Ownership covers code, tests, CI and docs; it adds no authority over data,
production, secrets or spend.
- You answer for the whole codebase, including what earlier agents wrote. A defect you find is yours to
  handle: fix the root cause when it causes or blocks your task, otherwise record it where this repo tracks
  work. "Pre-existing" or "not my change" is never a reason to walk past it.
- Security and customer-data findings never go into a public issue, PR or commit: flag them to the
  owner privately.
- A TODO or "needs human review" note finishes nothing: complete the work, or record the blocker and its
  unblock condition.
- Report to the owner in product terms (what works, what does not), not in code.

## Rules
- **Do not guess RPC method names or params.** Anything marked ❓ in `docs/GLINET_RPC.md` must be verified on a real device (or come from a capture the maintainer provides) before code depends on it. Put all method names in the typed API wrappers (`WifiApi`, `SystemApi`, `NetModeApi`) so they live in one place.
- Never log or commit passwords, login hashes, `sid` values, Wi-Fi keys or `.har` captures.
- Only connect to private or link-local IPv4 addresses (`PrivateAddressGuard`).
- The UI is for non-technical users: plain language, big targets, a confirmation before every apply, and no raw error dumps.
- A dropped connection right after an apply is **expected**. Handle it as a state, not an error.

## Build
- JDK 17, Android SDK (latest stable), Gradle wrapper.
- `./gradlew assembleDebug` and `./gradlew test` must pass before every commit.
- Talking to the router needs a phone or emulator on the same LAN as the router. An emulator's NAT usually cannot reach LAN devices or receive mDNS, so use a real phone over ADB for discovery work.

## Workflow
- Small PRs, one per milestone item from `docs/ROADMAP.md`. Tick items off as you go.
- When you verify an RPC call, update `docs/GLINET_RPC.md` in the same PR (✅, firmware version, redacted example).
