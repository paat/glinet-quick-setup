# Spec

## Target user
A non-technical person who owns a GL.iNet Flint and wants to (a) change Wi-Fi quickly or (b) turn the router into a plain access point behind another router. They should not need to know IP addresses, the admin panel, or networking terms.

## Features (v1)

### F1. Find and connect
- On launch, the app finds the router automatically:
  1. Last known IP from the saved profile.
  2. `192.168.8.1` (factory default in Router mode).
  3. mDNS / DNS-SD (`NsdManager`). It is not yet confirmed that the Flint advertises itself; see ROADMAP M0.
  4. A scan of the phone's current /24 subnet (see ARCHITECTURE "Discovery").
- If more than one GL.iNet device is found, the user picks one from a list (model, IP, MAC).
- The app asks for the admin password once and stores it encrypted (Android Keystore).
- Manual IP entry is available as an "Advanced" fallback.

### F2. Wi-Fi quick setup
- A toggle for each band (2.4 GHz and 5 GHz) and a "both" master toggle.
- SSID and password fields, with one checkbox: "Same name and password on both bands" (on by default).
- Validation: SSID 1–32 bytes, WPA2/WPA3 password 8–63 characters.
- **Apply** shows a warning: *"Wi-Fi will restart. If your phone is connected to this router's Wi-Fi, it will disconnect for a moment."*
- After applying, the app treats a dropped connection as expected. It offers to join the new SSID (`WifiNetworkSuggestion` / `WifiNetworkSpecifier`) and checks that the change was applied.
- Guest Wi-Fi is out of scope for v1.

### F3. Network mode
- Two large choices: **Router** and **Access Point** (wired: the Flint's WAN port connects to the main router's LAN, and the Flint becomes a Wi-Fi AP plus Ethernet switch).
- Before switching, the app shows what will happen in plain language and saves the device identity (MAC/serial) to the profile.
- After switching to AP: the router gets a new IP from the main router's DHCP. The app shows "Finding your router…" and runs discovery (F1, steps 3–4), checking the identity against the saved MAC/serial.
- After switching to Router: the app expects the router at `192.168.8.1` (or the configured LAN IP) and tells the user to connect to its Wi-Fi or LAN if needed.
- If discovery fails, the app shows tips: check the cable, look in the main router's DHCP client list, or factory reset (hold the reset button for 10 s) as a last resort.

### F4. Status screen
- Shows model, firmware version, current mode, IP, and SSID plus on/off state for each band.

## Non-goals (v1)
- Repeater, WDS and Extender modes, VPN, firewall, port forwarding.
- Firmware 3.x.
- GoodCloud / remote (non-LAN) access.
- iOS.

## Safety rules
- Only connect to private or link-local addresses (10/8, 172.16/12, 192.168/16, 169.254/16). The app refuses anything else, even though cleartext HTTP is allowed app-wide.
- Never log passwords, hashes or session ids.
- Every apply is preceded by a confirmation dialog in plain language.
