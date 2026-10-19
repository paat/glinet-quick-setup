# Notes for coding agents and contributors

Read `docs/SPEC.md`, `docs/ARCHITECTURE.md` and `docs/GLINET_RPC.md` before writing code.

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
