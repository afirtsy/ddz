## HTTP Endpoints

This reference details all HTTP routes handled by `src/worker.js`.

Base URL: `https://<your-domain>`

Auth: Cookie-based session with `jwtToken` set by POST `/login`.

---

### Panel

- GET `/panel`
  - Renders admin panel UI. If a password exists, requires authenticated session.

- POST `/panel`
  - Updates settings in KV. Requires authentication.
  - Body: `multipart/form-data` containing fields such as `remoteDNS`, `ports`, `VLConfigs`, `TRConfigs`, `cleanIPs`, `customCdnAddrs`, `warpEndpoints`, noise settings, routing flags, etc.

- POST `/panel/password`
  - Sets/replaces panel password. If already set, requires authentication. Body is plain text with new password.

---

### Authentication

- GET `/login`
  - Login page.

- POST `/login`
  - Body: `text/plain` with password. On success, sets `jwtToken` cookie.

- GET `/logout`
  - Clears session cookie.

---

### Utilities

- GET `/secrets`
  - Secrets generator page for UUID, Trojan password, and subscription path.

- POST `/my-ip`
  - Body: raw IP string. Returns JSON from `ip-api.com`.

---

### WARP

- POST `/update-warp`
  - Refreshes WARP accounts and persists to KV. Requires auth.

- GET `/get-warp-configs`
  - Returns a ZIP (`application/zip`) of WireGuard profiles based on KV.
  - Optional query `?app=amnezia` includes Amnezia noise params in the interface.

---

### Subscriptions

- GET `/sub/:subPath`
  - Returns subscription based on `app` query:
    - `?app=sfa` → Sing-Box custom config JSON
    - `?app=clash` → Clash normal config JSON
    - `?app=xray` → Xray full normal JSON
    - default → Base64-encoded list of VLESS/Trojan URIs

- GET `/fragsub/:subPath`
  - `?app=hiddify-frag` → Base64-encoded fragment-style links with headers
  - default → Xray fragment JSON including Best Ping and WorkerLess profiles

- GET `/warpsub/:subPath`
  - Clients:
    - `clash` | `clash-pro` → Clash Warp config (JSON), Pro adds Amnezia options
    - `singbox` → Sing-Box Warp config JSON
    - `hiddify` | `hiddify-pro` → Base64-encoded Hiddify Warp links
    - `xray` | `xray-pro` → Xray Warp/ WoW JSON; `xray-pro` adds UDP noise freedom outbound
    - `nikang` → Xray Warp JSON with NikaNG noise fields

---

### WebSockets (Protocols)

- Any WS upgrade to path starting with `/tr`
  - Handled by Trojan over WS (`TROverWSHandler`).

- Any other WS upgrade
  - Handled by VLESS over WS (`VLOverWSHandler`), with UDP/53 DNS via DoH.

---

### Examples

- Update settings
```bash
curl -X POST \
  -H "Cookie: jwtToken=<token>" \
  -F ports=443 -F VLConfigs=true -F TRConfigs=false \
  https://<host>/panel
```

- Download Warp ZIP
```bash
curl -H "Cookie: jwtToken=<token>" -o warp.zip https://<host>/get-warp-configs
```

- Get Xray Fragment JSON
```bash
curl https://<host>/fragsub/<SUB_PATH>?app=xray | jq .
```

