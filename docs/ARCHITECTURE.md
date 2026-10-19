# Architecture

## Stack
- Kotlin, Jetpack Compose (Material 3), minSdk 26, target the latest SDK.
- Coroutines + Flow; a single-activity app; ViewModels.
- OkHttp + kotlinx.serialization for JSON-RPC.
- Apache `commons-codec` for `Md5Crypt` / `Sha2Crypt` (the router's login hash).
- DataStore + Android Keystore (or `EncryptedSharedPreferences`) for the device profile and password.
- Optional later: `sshj` as an SSH/UCI fallback transport.

## Modules (Gradle)
```
app/                 UI (Compose screens, navigation, ViewModels)
core/rpc/            JSON-RPC client, login, session handling, typed API wrappers
core/discovery/      mDNS + subnet scan + GL fingerprint + identity check
core/model/          Shared data classes (DeviceInfo, WifiConfig, NetMode, ...)
core/storage/        Device profile + encrypted credentials
```
Start as one `app` module with these as packages if that is simpler. Split later.

## Transport
```kotlin
interface RouterTransport {
    suspend fun login(password: String): Session
    suspend fun call(module: String, method: String, params: JsonObject = JsonObject(emptyMap())): JsonElement
}
```
`RpcTransport` is the only implementation in v1. Typed wrappers (`WifiApi`, `SystemApi`, `NetModeApi`) are built on top. That way the method names live in one place, and firmware differences can be handled per version.

Session handling:
- Re-login once when a call fails with an "access denied" / invalid-session error.
- After an apply that restarts networking, ignore a dropped connection or timeout and run verification instead.

## Discovery
Order: saved IP → `192.168.8.1` → mDNS → subnet scan. Stop at the first verified hit.

**Subnet scan**
1. Get the active Wi-Fi `Network` and its `LinkProperties`. Use the IPv4 `LinkAddress` to get the prefix. Clamp anything larger than a /24 to the phone's own /24.
2. Bind HTTP to that network (`network.socketFactory` for OkHttp, or `bindProcessToNetwork`). Otherwise Android may send requests over mobile data when the Wi-Fi has no internet.
3. For each host, with about 32–64 in parallel: TCP connect to port 80 with a 300 ms timeout. If it connects, POST the `challenge` request to `/rpc` (see GLINET_RPC.md).
4. A **GL fingerprint** is a valid JSON-RPC result that contains `salt`, `nonce` and `alg`.
5. **Identity check**: if a saved profile exists, log in and compare the MAC/serial from `system.get_info` with the saved values.

**What does not work on Android:** reading `/proc/net/arp` (blocked since Android 10) or getting other devices' MACs without logging in. Do not rely on either.

## Android specifics
- **Cleartext HTTP**: `res/xml/network_security_config.xml` with `<base-config cleartextTrafficPermitted="true">`, referenced from the manifest. A private-IP allowlist in code (`PrivateAddressGuard`) makes this safe.
- **Permissions**: `INTERNET`, `ACCESS_NETWORK_STATE`, `ACCESS_WIFI_STATE`, `CHANGE_WIFI_STATE` (for Wi-Fi suggestions), `CHANGE_WIFI_MULTICAST_STATE` (for mDNS multicast lock). Reading the current SSID needs location permission plus location services on. Avoid it: v1 does not need the SSID.
- **Expected disconnects**: show an "Applying…" screen with a timeout and a recovery path. Never show a generic "network error" during an apply.

## Testing
- Unit tests: the login hash (with known vectors captured from a real device), JSON-RPC encoding, the subnet range maths and the private-IP guard.
- A fake router: a small local HTTP server (MockWebServer) that speaks the `/rpc` protocol, used in instrumentation and ViewModel tests.
- Manual test matrix on a real Flint: see ROADMAP.
