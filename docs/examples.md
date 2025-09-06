## Usage Examples

Practical examples for common workflows.

---

### 1) Log in and store cookie

```bash
HOST=https://<your-domain>
curl -i -c cookies.txt -H 'Content-Type: text/plain' \
  --data '<password>' $HOST/login | sed -n 's/Set-Cookie: \(jwtToken[^;]*\).*/\1/p'
```

---

### 2) Update panel settings

```bash
curl -b cookies.txt -X POST $HOST/panel \
  -F remoteDNS=https://cloudflare-dns.com/dns-query \
  -F localDNS=1.1.1.1 \
  -F VLTRFakeDNS=true \
  -F ports=443 -F ports=8443 \
  -F VLConfigs=true -F TRConfigs=false \
  -F cleanIPs=1.2.3.4,1.1.1.1 \
  -F enableIPv6=true
```

---

### 3) Download WireGuard ZIP

```bash
curl -b cookies.txt -o warp-configs.zip $HOST/get-warp-configs
```

For Amnezia-tuned configs:
```bash
curl -b cookies.txt -o warp-pro.zip "$HOST/get-warp-configs?app=amnezia"
```

---

### 4) Get Normal subscription (base64 list)

```bash
curl "$HOST/sub/<SUB_PATH>" | base64 -d
```

Full normal for specific clients:
```bash
curl "$HOST/sub/<SUB_PATH>?app=xray" | jq .
curl "$HOST/sub/<SUB_PATH>?app=sfa" | jq .
curl "$HOST/sub/<SUB_PATH>?app=clash" | jq .
```

---

### 5) Fragment subscriptions

```bash
curl "$HOST/fragsub/<SUB_PATH>?app=xray" | jq .
curl "$HOST/fragsub/<SUB_PATH>?app=hiddify-frag" | base64 -d
```

---

### 6) Warp subscriptions

```bash
curl "$HOST/warpsub/<SUB_PATH>?app=xray" | jq .
curl "$HOST/warpsub/<SUB_PATH>?app=clash" | jq .
curl "$HOST/warpsub/<SUB_PATH>?app=singbox" | jq .
curl "$HOST/warpsub/<SUB_PATH>?app=hiddify" | base64 -d
```

Pro variants:
```bash
curl "$HOST/warpsub/<SUB_PATH>?app=xray-pro" | jq .
curl "$HOST/warpsub/<SUB_PATH>?app=clash-pro" | jq .
curl "$HOST/warpsub/<SUB_PATH>?app=hiddify-pro" | base64 -d
```

---

### 7) Geo lookup for an IP

```bash
curl -X POST $HOST/my-ip --data '8.8.8.8'
```

