# GL.iNet 4.x JSON-RPC notes

Official reference: https://dev.gl-inet.com/router-4.x-api/ (incomplete, and it lags behind firmware releases).

> **Verification status.** Items marked ✅ have been checked on a real GL-AX1800. Items marked ❓ come from documentation or memory and **must be checked** (see "Capturing calls" below) before code depends on them. Update this file as you verify things.

Endpoint: `POST http://<router>/rpc`, `Content-Type: application/json`, JSON-RPC 2.0.

## Login ❓

1. **Challenge** (no authentication needed):
   ```json
   {"jsonrpc":"2.0","id":1,"method":"challenge","params":{"username":"root"}}
   ```
   The result contains `alg` (1 = MD5, 5 = SHA-256, 6 = SHA-512), `salt` and `nonce`. Newer firmware also sends `hash-method` (e.g. `"md5"` or `"sha256"`).

2. **Compute**:
   ```
   cipher = crypt(password, "$" + alg + "$" + salt + "$")   // Md5Crypt / Sha2Crypt from commons-codec
   hash   = H("root" + ":" + cipher + ":" + nonce)          // H = md5 by default, or whatever hash-method says; lowercase hex
   ```

3. **Login**:
   ```json
   {"jsonrpc":"2.0","id":2,"method":"login","params":{"username":"root","hash":"<hash>"}}
   ```
   The result contains `sid`.

4. **Call**:
   ```json
   {"jsonrpc":"2.0","id":3,"method":"call","params":["<sid>","<module>","<method>",{ ... }]}
   ```

Errors come back in the standard JSON-RPC `error` object. Record the codes for "access denied" and "invalid session" here once you have seen them: ❓

## Calls the app needs

| Purpose | Module.method | Status | Notes |
|---|---|---|---|
| Model, firmware, MAC, serial | `system.get_info` | ❓ | Used for the status screen and the identity check |
| Overall status | `system.get_status` | ❓ | |
| Read Wi-Fi config | `wifi.get_config` | ❓ | Record the band and iface structure |
| Write Wi-Fi config | `wifi.set_config` | ❓ | Record the exact param names: enabled, ssid, key, encryption, per-iface |
| Read network mode | ❓ | ❓ | Capture from the web UI's "Network Mode" page |
| Set network mode (Router / AP) | ❓ | ❓ | Capture it. Also record what the response looks like before the device drops the connection |

## Capturing calls

1. Open the web UI (`http://192.168.8.1`) in desktop Chrome or Firefox and log in.
2. DevTools → Network → filter on `rpc`. Turn on "Preserve log".
3. Do the action in the UI (for example, change the network mode to Access Point).
4. Copy the **request payload** and the **response**, then remove the `sid` and any passwords or keys before pasting them here.
5. Note the firmware version (shown in System → Upgrade or in `system.get_info`).

Do **not** commit `.har` files or raw captures. They contain session ids and Wi-Fi passwords. `.gitignore` excludes `*.har` and `/captures-local/`.

## Firmware versions seen
| Firmware | Date checked | Notes |
|---|---|---|
| | | |
