# Roadmap

## M0: Verify against the device (before writing app logic)
- [ ] Record the Flint's firmware version.
- [ ] Verify the login flow with a script (curl/Python) and fill in GLINET_RPC.md. Save one known test vector (salt, nonce, password, expected hash) for unit tests, **using a throwaway password**.
- [ ] Capture `system.get_info`, `wifi.get_config` and `wifi.set_config`.
- [ ] Capture the Network Mode get/set calls (Router ↔ Access Point).
- [ ] In AP mode: does the Flint advertise itself over mDNS? (Check with a network-discovery app or `avahi-browse -a`.) Does it still answer on `192.168.8.1` (fallback alias)? Write down the results.

## M1: Project skeleton and RPC client
- [ ] Gradle project, Compose, CI (GitHub Actions: build plus unit tests).
- [ ] `RpcTransport`: login, call, re-login on an invalid session.
- [ ] Login hash plus unit tests with the M0 vector.
- [ ] `PrivateAddressGuard` plus `network_security_config.xml`.

## M2: Connect and status
- [ ] Discovery: saved IP → 192.168.8.1 → manual entry.
- [ ] Password prompt plus encrypted storage.
- [ ] Status screen (F4).

## M3: Wi-Fi quick setup (F2)
- [ ] Toggles, SSID/password, both bands.
- [ ] Handling of the expected disconnect, then verification.

## M4: Network mode plus auto-discovery (F3)
- [ ] mDNS discovery (if M0 says it works).
- [ ] Subnet scan with GL fingerprint and identity check.
- [ ] Router → AP → rediscover. AP → Router.
- [ ] Recovery tips screen.

## M5: Release
- [ ] Tests on a real device across the full matrix (below).
- [ ] F-Droid metadata and a signed GitHub release APK.

## Manual test matrix
| Case | Expected |
|---|---|
| Phone on the Flint's Wi-Fi, Router mode, change SSID | App reconnects or prompts, new SSID applied |
| Router → AP, phone on the main router's Wi-Fi | Flint found by scan or mDNS within 30 s |
| Router → AP, phone on the Flint's Wi-Fi | Phone drops, rejoins, Flint found |
| AP → Router | Flint found at 192.168.8.1 |
| Two GL devices on the same LAN | Picker shown, identity check picks the right one |
| Wrong password | Clear message, no lockout loop |
| Wi-Fi without internet | Requests still go over Wi-Fi (network binding) |
