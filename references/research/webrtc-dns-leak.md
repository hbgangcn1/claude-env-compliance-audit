# WebRTC & DNS Leak Closure for Chrome/Edge on Windows 11

Scope: third-party tools and settings only (no system changes made). Targets the two requirements
from the environment-cleanup guide (block non-proxied WebRTC UDP) and the VPN/proxy guide (DNS
resolution must travel with the proxy path), verified per this document's section 3.

---

## 1. WebRTC blocking options (extension names + store URLs + does it truly hide the local IP?)

Chrome and Edge have **no user-facing WebRTC IP policy in `chrome://settings` / `edge://settings`** —
Chrome removed the GUI flag years ago, so control comes from an extension, an enterprise policy, or
a command-line switch. Note the two distinct things a "leak" can expose:

- **Local/private IP** (e.g. `192.168.1.37`) — exposed via WebRTC host candidates.
- **Real public IP** — exposed via STUN server-reflexive candidates when UDP bypasses the
  VPN/proxy tunnel. This is the leak that matters behind a proxy; only
  `disable_non_proxied_udp` closes it.

### Tools (7 named tools)

| # | Tool | Store / official URL | Local IP hidden? | Real public IP hidden behind proxy? |
|---|------|----------------------|------------------|-------------------------------------|
| 1 | **WebRTC Control** (named by the environment-cleanup guide) | Chrome Web Store: https://chromewebstore.google.com/detail/webrtc-control/fjkmabmdepjfammlpliljpnbhleegehm — Edge Add-ons: https://microsoftedge.microsoft.com/addons/detail/webrtc-control/eepeadgljpkkjpbfecfkijnnliikglpl | **Yes — fully.** When toggled OFF it neuters `RTCPeerConnection`, `getUserMedia`, `MediaStreamTrack`, `RTCSessionDescription`, so no ICE candidates are ever generated. | **Yes** (no candidates at all). Trade-off: all WebRTC features (Meet/Teams/voice in browser) break while it is off. |
| 2 | **WebRTC Leak Prevent** | Chrome Web Store: https://chromewebstore.google.com/detail/webrtc-leak-prevent/eiadekoaikejlgdbkbdfeijglgfdalml (source: https://github.com/a449707101729771095/WebRTC-Leak-Prevent) | **Yes.** Exposes the hidden `webRTCIPHandlingPolicy` preference; default `default_public_interface_only` drops private-interface candidates. | **Only if you change its option to `disable_non_proxied_udp`** (its options page supports all four RFC 8828 policies). Default policy still lets STUN discover your real public IP. |
| 3 | **WebRTC Network Limiter** (Google's own extension) | Chrome Web Store: https://chromewebstore.google.com/detail/webrtc-network-limiter/npeicpdbkakmehahjeeohfdhnlpdklia | **Yes.** Blocks private IPs (192.168.x/10.x) and public IPs of interfaces not used for web traffic; forces WebRTC through the configured proxy. | **Mostly, but the crucial switch is off by default**: "disable non-proxied UDP" must be enabled on its Options page. Until then, in a newer Chrome the default install behavior can still allow both default public + private addresses. |
| 4 | **WebRTC Protect – Protect IP Leak** | Chrome Web Store: https://chromewebstore.google.com/detail/webrtc-protect-protect-ip/bkmmlbllpjdpgcgdohbaghfaecnddhni | **Yes — fully.** WebRTC is blocked by default; toolbar button can temporarily re-enable it. Also blanks the Media Device Enumeration API. | **Yes** while WebRTC is blocked. Same call-quality/feature trade-off as #1. |
| 5 | **Chrome enterprise policy `WebRtcIPHandling`** (settings route, no extension) | https://chromeenterprise.google/intl/en_uk/policies/web-rtc-ip-handling/ | **Yes** with value `default_public_interface_only`. | **Yes** with value `disable_non_proxied_udp` (WebRTC may only use UDP via a configured SOCKS proxy, else falls back to TCP). Set at `HKLM/HKCU\Software\Policies\Google\Chrome`, value `WebRtcIPHandling` = `disable_non_proxied_udp`; verify at `chrome://policy`. Policy **takes precedence over any extension**. |
| 6 | **Edge enterprise policy `WebRtcIPHandling` / `WebRtcIPHandlingUrl`** (settings route, no extension) | https://learn.microsoft.com/en-us/deployedge/microsoft-edge-policies/webrtciphandlingurl (also `WebRtcIPHandling`) | **Yes** (`default_public_interface_only`). | **Yes** (`disable_non_proxied_udp`). Registry: `SOFTWARE\Policies\Microsoft\Edge`, e.g. value `WebRtcIPHandling` = `disable_non_proxied_udp`, or per-URL dictionary `WebRtcIPHandlingUrl` like `[{"handling":"disable_non_proxied_udp","url":"*"}]`. Verify at `edge://policy`. |
| 7 | **Chromium launch switch `--webrtc-ip-handling-policy=disable_non_proxied_udp`** | Documented by ungoogled-chromium: https://github.com/ungoogled-software/ungoogled-chromium/pull/2359 | **Yes.** | **Yes.** Applies to the whole browser process; brittle (lost on shortcut change) — least preferred of the three routes. |

**Recommendation for this machine:** Edge policy `WebRtcIPHandling = disable_non_proxied_udp` (#6)
plus, if you want a visible toggle in Chrome, **WebRTC Control** (#1) — it is the extension the guide
names and the only one whose OFF state guarantees *zero* ICE candidates. If you need in-browser video
calls to keep working, use **WebRTC Leak Prevent** (#2) set to `disable_non_proxied_udp` instead of a
hard block: local IP hidden, UDP restricted to the proxy path, TCP fallback keeps calls alive.

Note: Chrome/Edge also mangle local IPs into mDNS `.local` hostnames by default
(`chrome://flags/#webrtc-hide-local-ips-with-mdns`) — that is obfuscation of the *local* IP only and
does nothing against STUN discovery of your *real public* IP, so it is not a substitute for the
tools above.

---

## 2. DNS leak prevention options (client features + Windows DoH configuration)

"A DNS leak" = any name resolution that leaves via a path other than the proxy tunnel (ISP resolver,
plain UDP/53 to a public resolver, a browser DoH bypassing the proxy core).

### 2a. Proxy-client features (DNS travels with the proxy path)

| # | Tool | Official URL | What to configure |
|---|------|--------------|-------------------|
| 8 | **Clash / mihomo (Clash Meta core) — fake-ip mode** | DNS config docs: https://wiki.metacubex.one/en/config/dns/ | ```yaml<br>dns:<br>  enable: true<br>  enhanced-mode: fake-ip<br>  fake-ip-range: 198.18.0.1/16<br>  fake-ip-filter: ['*.lan', '*.local', '+.msftconnecttest.com']<br>  nameserver: ['https://1.1.1.1/dns-query', 'https://dns.google/dns-query']<br>  respect-rules: true   # DNS upstreams follow routing rules → through proxy<br>```<br>With TUN (your Wintun adapter) also set `dns-hijack: ['any:53']` so stray port-53 queries are captured. fake-ip returns a synthetic `198.18.x.x` answer locally and the core resolves the real name **inside the tunnel** — the local network never sees the query. Put domain rules before IP rules (tag IP rules `no-resolve`) or the core itself will do an early local lookup. |
| 9 | **sing-box — remote DNS via `detour`** | https://sing-box.sagernet.org/configuration/dns/ | Give the remote DNS server a proxy outbound: `{ "type": "https", "tag": "remote", "server": "1.1.1.1", "path": "/dns-query", "detour": "proxy" }`, set `"final": "remote"` (or a rule routing proxied domains to it), and in `route` keep `"action": "hijack-dns"` for TUN. The `detour` field is exactly what makes resolution travel with the proxy path. Avoid `dhcp://auto` as the only server — that is a direct-to-ISP query (a known leak pattern in generated configs; see https://github.com/2dust/v2rayN/issues/8865). |

### 2b. Windows 11 native DoH (covers everything *outside* the proxy core)

**Windows 11 Settings DoH** (built-in since launch; no third-party software):

1. `Win+I` → **Network & Internet** → your adapter (Wi-Fi/Ethernet) → **Hardware properties**
   (or the connection page) → **DNS server assignment** → **Edit**.
2. Set to **Manual**, toggle **IPv4** on.
3. Preferred DNS `1.1.1.1`, Alternate `1.0.0.1` (or Google `8.8.8.8`/`8.8.4.4`, Quad9
   `9.9.9.9`/`149.112.112.112`) and set the **DNS over HTTPS** dropdown to
   **"Encrypted only (DNS over HTTPS)"** for each — "Encrypted only" is the strict mode;
   "Encrypted preferred" silently falls back to plaintext UDP/53 on handshake failure.
4. The DoH dropdown only appears for Windows' built-in known list (Cloudflare `1.1.1.1`/`1.0.0.1`,
   Google, Quad9 — see https://learn.microsoft.com/en-us/windows-server/networking/dns/doh-client-support).
   **Your current `1.1.1.2` (Cloudflare Families) is NOT on that list**, so its DoH template must be
   registered first (PowerShell, admin):
   ```powershell
   Add-DnsClientDohServerAddress -ServerAddress 1.1.1.2 -DohTemplate https://security.cloudflare-dns.com/dns-query -AutoUpgrade $true -AllowFallbackToUdp $false
   Add-DnsClientDohServerAddress -ServerAddress 1.0.0.2 -DohTemplate https://security.cloudflare-dns.com/dns-query -AutoUpgrade $true -AllowFallbackToUdp $false
   ```
   Cloudflare Families endpoints (official): `1.1.1.2` (malware blocking) →
   `https://security.cloudflare-dns.com/dns-query`; `1.1.1.3` (malware + adult) →
   `https://family.cloudflare-dns.com/dns-query` — https://developers.cloudflare.com/1.1.1.1/setup/
   Current state: with plain `1.1.1.2` and no DoH configured, your queries are encrypted **not at
   all** — 1.1.1.2 on UDP/53 is plaintext DNS.
5. Reboot/reconnect, confirm the DNS servers show **"(Encrypted)"** in the adapter properties.

**Chrome / Edge Secure DNS (browser layer):**

- Chrome: `chrome://settings/security` → **Use secure DNS** → On → "With" → Cloudflare
  (`https://cloudflare-dns.com/dns-query`) or Google (`https://dns.google/dns-query`).
- Edge: `edge://settings/privacy` → **Security** → **Use secure DNS to specify how to lookup the
  network address for websites** → On → **Choose a service provider** (Cloudflare/Google/Quad9) or a
  custom provider URL.
- Interaction with Clash/sing-box fake-ip: a browser pinned to its own DoH provider bypasses the
  proxy core's DNS module. Two consistent designs, pick one:
  - **(A) Proxy core owns DNS** (matches the proxy guide): leave browser Secure DNS **off** so all
    lookups fall to Windows/core and go through the tunnel via fake-ip/`detour`.
  - **(B) Browser owns DNS**: enable Secure DNS — it rides whatever network path TCP/443 takes,
    which under TUN is the tunnel; leak tests should then show the DoH provider, not your ISP.
  Never leave one half configured so Windows says DoH while the browser silently uses another
  resolver — mixed providers produce ambiguous leak-test results.

---

## 3. PASS/FAIL self-test procedure (exact sites + the observable that decides PASS)

Run everything **inside the same browser/profile with the proxy connected**. Five minutes.

### Step 0 — Baseline (optional but makes FAIL unambiguous)

Disconnect the proxy → open https://www.browserleaks.com/ip → note your real ISP public IP and ISP
name. Reconnect the proxy.

### Step 1 — WebRTC: https://browserleaks.com/webrtc

The page creates `RTCPeerConnection`s against STUN servers and lists every ICE candidate.
Scroll to **WebRTC Local IP Address** and **WebRTC Public IP Address** (the srflx / "server
reflexive" candidates; also check the raw ICE candidate list).

| Observable | Verdict |
|---|---|
| **No `host` candidate with a real private IP** (only mDNS `...local` names or nothing) **AND no `srflx` candidate showing your real ISP IP** — the only public IP visible is the proxy/VPN exit IP, or there are no candidates at all | **PASS — WebRTC leak closed** |
| Any `srflx` candidate equals your real ISP public IP from Step 0 | **FAIL** — non-proxied UDP still escaping; extension/policy not actually active |
| Private IP `192.168.x.x` / `10.x.x.x` visible in a host candidate | **FAIL** (local-IP requirement) — set the extension/policy to `disable_non_proxied_udp` or `default_public_interface_only` |
| "No WebRTC candidates" / test errors because RTCPeerConnection is disabled | **PASS** (WebRTC Control / WebRTC Protect in block mode) |

Cross-check: https://ipleak.net — its "IP Address" + WebRTC section must both show the proxy exit
IP only.

### Step 2 — DNS: https://www.dnsleaktest.com

1. Click **Standard test** (6 queries), then **Extended test** (36 queries — use this for the
   verdict; the short one can miss intermittent leaks).
2. The result table lists every resolver that answered: **IP, DNS server name, provider/ASN,
   country**.

| Observable | Verdict |
|---|---|
| Every row's provider belongs to your intended path — Cloudflare (`Cloudflare, Inc.` / `1.1.1.x`), Google, Quad9, your DoH provider, or the proxy provider's ASN — and the country matches your proxy exit country | **PASS — DNS travels with the proxy** |
| **Even one row** shows your home ISP's name/ASN (compare with Step 0 baseline) | **FAIL — DNS leak confirmed** |
| A mix of your provider AND your ISP | **FAIL — partial/intermittent leak** (usually browser Secure DNS fighting the proxy core, IPv6 bypass, or `redir-host` mode instead of fake-ip) |
| Rows in a country different from the proxy exit | **FAIL** (DNS escaping to a local resolver) — re-check `respect-rules`/`detour`/`fake-ip` |

Cross-check with https://browserleaks.com/dns (resolves 50 random hostnames, 25 IPv4 + 25 IPv6) —
this catches IPv6-only leaks that dnsleaktest.com can miss. Any ISP row there = FAIL.

### Step 3 — DoH confirmation (proves encryption, not just the resolver path)

Open https://1.1.1.1/help → **"Using DNS over HTTPS (DoH): Yes"** = PASS for encrypted DNS.
"No" = your DoH setting isn't actually in effect (Windows dropdown or browser Secure DNS off /
overridden by policy). Optional: run `Get-DnsClientDohServerAddress` in PowerShell and confirm your
servers have a `DohTemplate` and `AutoUpgrade: True`.

### Final gate

**PASS overall** requires all three simultaneously:
1. browserleaks.com/webrtc shows **no real private IP and no real ISP public IP**;
2. dnsleaktest.com **Extended** test shows **zero ISP resolver rows** (only Cloudflare/your chosen
   resolver or proxy-provider ASN, country matching the exit);
3. 1.1.1.1/help reports **DoH: Yes**.

Re-run after every browser update, proxy-client update, adapter/DNS change, or profile switch —
each of these has been observed to reintroduce a leak.
