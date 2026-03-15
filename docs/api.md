## BPB Worker Panel - Public API Reference

### Overview
This document lists all public modules, exported functions, and HTTP endpoints exposed by the BPB Worker Panel. Use it as the canonical reference for integration and extension.

---

### Runtime Entry

- `src/worker.js`
  - `export default { async fetch(request, env) }`
    - Main request handler. Initializes globals and routes to endpoints below.

---

### Authentication (`src/authentication/auth.js`)

- `Authenticate(request, env): Promise<boolean>`
  - Validates session via `jwtToken` cookie.
- `login(request, env): Promise<Response>`
  - GET: renders login page. POST: validates password and sets `jwtToken` cookie.
- `logout(): Response`
  - Clears auth cookie.
- `resetPassword(request, env): Promise<Response>`
  - POST: sets new password in KV. Requires auth if password already set.

---

### KV Dataset (`src/kv/handlers.js`)

- `getDataset(request, env): Promise<{ proxySettings: object, warpConfigs: object }>`
  - Reads `proxySettings` and `warpConfigs` from KV, bootstraps defaults when missing.
- `updateDataset(request, env): Promise<object>`
  - POST form to update `proxySettings` in KV. Supports reset.
- `updateWarpConfigs(request, env): Promise<Response>`
  - Auth-only POST to refresh Cloudflare WARP accounts and persist to KV.

---

### Helpers (`src/helpers/helpers.js`)

- `isValidUUID(uuid: string): boolean`
- `resolveDNS(domain: string): Promise<{ ipv4: string[], ipv6: string[] }>`
- `isDomain(address: string): boolean`
- `handlePanel(request, env): Promise<Response>`
  - GET: renders panel. POST: updates settings (auth required).
- `fallback(request): Promise<Response>`
  - Proxy to fallback domain.
- `getMyIP(request): Promise<Response>`
  - POST body: IP string. Returns Geo info JSON.
- `getWarpConfigs(request, env, isPro?: boolean): Promise<Response>`
  - Returns ZIP of WireGuard profiles.

`src/helpers/init.js`
- `initializeParams(request, env): void`
  - Initializes global runtime parameters and guards required secrets/KV.

---

### Protocols

`src/protocols/vless.js`
- `VLOverWSHandler(request): Promise<Response>`
  - WebSocket handler implementing VLESS over WS with DNS-over-HTTPS for UDP/53.

`src/protocols/trojan.js`
- `TROverWSHandler(request): Promise<Response>`
  - WebSocket handler implementing Trojan over WS.

`src/protocols/warp.js`
- `fetchWarpConfigs(env): Promise<{ error: string|null, configs: string }>`
  - Generates two WARP accounts and stores them in KV.

---

### Core Config Generators (`src/cores-configs`)

`helpers.js`
- `getConfigAddresses(cleanIPs: string, enableIPv6: boolean): Promise<string[]>`
- `extractWireguardParams(warpConfigs, isWoW: boolean): { warpIPv6, reserved, publicKey, privateKey }`
- `generateRemark(index, port, address, cleanIPs, protocol, configType?): string`
- `randomUpperCase(str: string): string`
- `getRandomPath(length: number): string`
- `base64ToDecimal(base64: string): number[]`
- `isIPv4(address: string): boolean`
- `isIPv6(address: string): boolean`
- `getDomain(url: string): { host: string, isHostDomain: boolean }`

`normalConfigs.js`
- `getNormalConfigs(request, env): Promise<Response>`
  - Returns base64 subscription text of VLESS/Trojan URIs.
- `getHiddifyWarpConfigs(request, env, isPro: boolean): Promise<Response>`
  - Returns base64-encoded Hiddify Warp links.

`xray.js`
- `getXrayCustomConfigs(request, env, isFragment: boolean): Promise<Response>`
- `getXrayWarpConfigs(request, env, client: 'xray'|'xray-pro'|'nikang'|...): Promise<Response>`

`clash.js`
- `getClashNormalConfig(request, env): Promise<Response>`
- `getClashWarpConfig(request, env, isPro?: boolean): Promise<Response>`

`sing-box.js`
- `getSingBoxCustomConfig(request, env): Promise<Response>`
- `getSingBoxWarpConfig(request, env): Promise<Response>`

---

### Pages (`src/pages`)

- `renderHomePage(proxySettings, isPassSet): Promise<Response>`
- `renderLoginPage(): Promise<Response>`
- `renderSecretsPage(): Promise<Response>`
- `renderErrorPage(error): Promise<Response>`

---

### HTTP Endpoints and Routing

All endpoints are served from `src/worker.js` in the default export `fetch` router. Unless stated, methods are GET. Auth required where noted.

- `/panel`
  - GET: Panel HTML (auth enforced if password set)
  - POST: Update settings (auth required)
- `/panel/password` (POST)
  - Change password. Requires auth if a password already exists.
- `/login`
  - GET login page; POST sets session cookie on success.
- `/logout`
  - Clears session cookie.
- `/secrets`
  - Secrets generator page.
- `/my-ip` (POST)
  - Body: raw IP string. Returns Geo info JSON.
- `/update-warp` (POST, auth required)
  - Refresh WARP accounts in KV.
- `/get-warp-configs` (GET, auth required)
  - Optional `?app=amnezia` to include Amnezia noise settings.
  - Returns application/zip of WireGuard profiles.
- `/sub/:subPath`
  - `?app=sfa|clash|xray` selects generator; default returns normal base64 subscription.
- `/fragsub/:subPath`
  - `?app=hiddify-frag` for Hiddify-fragment; default returns Xray fragment JSON.
- `/warpsub/:subPath`
  - `?app=clash|clash-pro|singbox|hiddify|hiddify-pro|xray|xray-pro|nikang`
  - Returns client-specific Warp configs JSON or encoded text depending on client.
- WebSocket upgrades:
  - Path starts with `/tr` => `TROverWSHandler`
  - Otherwise => `VLOverWSHandler`
- default: `fallback` proxy to `FALLBACK` origin.

Auth is cookie-based: `jwtToken` set by POST `/login`. Use it for subsequent POSTs and protected downloads.

---

### Usage Examples

- Login
```http
POST /login
Content-Type: text/plain

<password>
```

- Update settings
```http
POST /panel
Cookie: jwtToken=<token>
Content-Type: multipart/form-data

<form fields>
```

- Download WireGuard ZIP
```http
GET /get-warp-configs
Cookie: jwtToken=<token>
```

- Fetch Xray custom configs (fragment)
```http
GET /fragsub/<SUB_PATH>?app=xray
```

- Get Geo for an IP
```http
POST /my-ip

8.8.8.8
```

---

### Environment and KV

- Required KV binding: `kv`
- Required secrets: `UUID`, `TR_PASS`
- Optional: `PROXYIP`, `DOH_URL`, `FALLBACK`, `SUB_PATH`

