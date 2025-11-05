## UI Pages and Components

All pages render HTML responses from the Worker. They support a dark mode toggle persisted via `localStorage`.

---

### Home Panel (`/panel`)

Rendered by: `renderHomePage(proxySettings, isPassSet)`

Sections:
- VLESS/Trojan settings: DNS, FakeDNS, ports, protocols, chain proxy, custom CDN.
- Fragment settings: length, interval, packets.
- WARP settings: endpoints, FakeDNS, IPv6, update action, Best Interval.
- WARP Pro settings: UDP noise for Xray/Clash, Hiddify and NikaNG modes, Amnezia noise.
- Routing rules: bypass LAN/Iran/China/Russia, block Ads/Porn, block UDP/443, custom rules.
- Subscription tables: Normal, Full Normal, Fragment, Warp, Warp Pro with QR/URL and downloads per client.
- IP info panel: queries `ipwho.is` and `/my-ip` for geo details.

Key client-side handlers (embedded JS):
- `applySettings(event, form)` POSTs to `/panel` with validation.
- `getWarpConfigs()` POSTs to `/update-warp`.
- `downloadWarpConfigs(isAmnezia?)` GET `/get-warp-configs`.
- `dlURL(url)` downloads JSON config via selected sub route.
- `resetPassword(event)` POST `/panel/password`.
- `logout(event)` GET `/logout`.

---

### Login (`/login`)

Rendered by: `renderLoginPage()`

Behavior:
- Submits password as `text/plain` to `/login`.
- On success, redirects to `/panel`.

---

### Secrets Generator (`/secrets`)

Rendered by: `renderSecretsPage()`

Features:
- Generates: `UUID`, Trojan password, and a random subscription path.
- Copy-to-clipboard UI for each value.

---

### Error Page

Rendered by: `renderErrorPage(error)`

Usage:
- Displayed on unhandled exceptions inside router.

