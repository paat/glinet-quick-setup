# glinet-quick-setup

An open-source Android app that does the two most common setup tasks on a **GL.iNet GL-AX1800 (Flint)** router, without opening the web admin panel:

1. **Wi-Fi**: turn it on or off, and set the SSID and password quickly (per band or both bands).
2. **Network mode**: switch between **Router** and **Access Point** (wired AP / Ethernet switch) mode. In AP mode the app **finds the router's new IP address automatically**.

The app is meant for non-technical users. It finds the router, asks for the admin password once, and then offers a few large buttons.

> Status: **design / pre-M0**. Only documentation exists so far. See [docs/ROADMAP.md](docs/ROADMAP.md).

## How it works

- It talks to the router's local **JSON-RPC API** (`http://<router>/rpc`, GL.iNet firmware 4.x). This is the same API the official web UI uses. Nothing goes through the cloud.
- The admin password is stored only on the phone, encrypted with the Android Keystore.
- Discovery uses mDNS first, then a fast scan of the local /24 subnet that fingerprints the GL.iNet `/rpc` challenge. The device's MAC or serial is checked to confirm it is the same router.

## Docs

| Doc | Contents |
|---|---|
| [docs/SPEC.md](docs/SPEC.md) | Features, user flows, non-goals |
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | App modules, transport, discovery, Android specifics |
| [docs/GLINET_RPC.md](docs/GLINET_RPC.md) | Login algorithm, known RPC calls, how to capture unknown ones |
| [docs/ROADMAP.md](docs/ROADMAP.md) | Milestones |
| [AGENTS.md](AGENTS.md) | Notes for coding agents and contributors |

## Supported devices

| Device | Firmware | Status |
|---|---|---|
| GL-AX1800 (Flint) | 4.x | target |
| Other GL.iNet 4.x routers | 4.x | probably work, not tested |
| GL.iNet 3.x firmware | 3.x | not supported (different API) |

## Disclaimer

This is an independent community project. It is not affiliated with or endorsed by GL.iNet. "GL.iNet" and "Flint" are trademarks of their owner. Changing the network mode can make the router unreachable until it is found again or reset. Use at your own risk.

## License

[MIT](LICENSE)
