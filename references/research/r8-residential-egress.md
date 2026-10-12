# R8 — Residential-egress options for this machine

**The gap:** this machine egresses via Astrill VPN at exit IP 192.119.15.202 (Los Angeles US), ASN AS55081 "24 SHELLS" / Host World Net LLC — a hosting/datacenter ASN — while the site rates datacenter and shared VPN-pool IPs **high-risk** and residential/mobile-ISP IPs **low-risk**, and requires an exit in an Anthropic-supported country (US/UK/SG/JP at minimum; TW also required for fit here), with IP type, DNS hygiene and session stability all evaluated against those criteria ([VPN 与代理选择指南](https://fuck-claude.vercel.app/zh/guides/vpn-and-proxy-selection/), [环境净化与原生住宅 IP 指南](https://fuck-claude.vercel.app/zh/guides/environment-cleanup-and-ip-setup/)) — no local setting can change the egress ASN, so a third-party egress is required.

---

## Option 1 — Bright Data Residential Proxies

- **Category:** residential proxy service
- **Official website:** https://brightdata.com/proxy-types/residential-proxies
- **Supported exit countries (US/SG/JP/TW):** Yes — 195 countries; the residential FAQ's exclusion list (Iran, Iraq, Syria, Lebanon, Palestine, North Korea, Cuba, Sudan, Belarus, Russia) leaves US, SG, JP and TW all available, and the locations page lists US and Japan IP counts ([docs FAQ](https://docs.brightdata.com/proxy-networks/residential/faqs), [locations](https://brightdata.com/locations)).
- **Genuine residential/mobile ISP ASN:** Yes — "routes your traffic through 400M+ real residential IPs… a real residential device in your chosen target region," with carrier/ASN-level targeting ([docs introduction](https://docs.brightdata.com/proxy-networks/residential/introduction)). Note: residential-network access requires KYC approval by Bright Data's compliance team ([docs configuration](https://docs.brightdata.com/proxy-networks/residential/configure-your-proxy)).
- **DNS-leak protection:** Yes — explicit `-dns-remote` (resolve at the exit/proxy) vs `-dns-local` username parameter, so DNS can be forced to the exit path ([config options](https://docs.brightdata.com/proxy-networks/config-options)).
- **Sticky-session support:** Yes — `-session-<id>` keeps the same peer; session IP persists up to 5 minutes of idle time (keep-alive extends it) ([docs FAQ](https://docs.brightdata.com/proxy-networks/residential/faqs)).
- **Fit verdict:** Full fit — genuine residential ASN in all four target countries with remote-DNS and sticky sessions; the only friction is corporate KYC on the residential network.

## Option 2 — Oxylabs Residential Proxies

- **Category:** residential proxy service
- **Official website:** https://oxylabs.io/products/residential-proxy-pool
- **Supported exit countries (US/SG/JP/TW):** Yes — 195 countries, with dedicated country entry nodes explicitly listed for all four: `us-pr.oxylabs.io`, `sg-pr.oxylabs.io`, `jp-pr.oxylabs.io`, `tw-pr.oxylabs.io` ([select-country docs](https://developers.oxylabs.io/products/proxies/residential-proxies/location-settings/select-country)).
- **Genuine residential/mobile ISP ASN:** Yes — "real IP addresses provided by Internet Service Providers (ISPs) attached to physical devices," filterable by ASN ([developer docs](https://developers.oxylabs.io/products/proxies/residential-proxies), [product page](https://oxylabs.io/products/residential-proxy-pool)).
- **DNS-leak protection:** Yes — hostnames are resolved on the residential exit node, and UDP DNS queries (port 53) execute via residential IPs ([protocols docs](https://developers.oxylabs.io/products/proxies/residential-proxies/protocols)).
- **Sticky-session support:** Yes — `sessid` / sticky ports, default 10-minute sessions extendable with `sesstime` up to 1440 minutes (24 h), plus `sessid_oneip` for strict single-IP binding that fails rather than rotates ([session-control docs](https://developers.oxylabs.io/products/proxies/residential-proxies/session-control)).
- **Fit verdict:** Full fit — the strongest combination for this machine: ISP-sourced IPs in US/SG/JP/TW, exit-side DNS, and up to 24-hour sticky sessions for a stable account identity.

## Option 3 — CometVPN Native Residential VPN

- **Category:** VPN offering residential IPs
- **Official website:** https://cometvpn.com/features/residential-vpn/native/
- **Supported exit countries (US/SG/JP/TW):** Yes — server-location list includes United States, Singapore, Japan and Taiwan ([locations](https://cometvpn.com/locations/)); the Taiwan page specifically advertises "Native residential IPs… coming from Internet Service Providers" ([Taiwan page](https://cometvpn.com/locations/taiwan-vpn/)).
- **Genuine residential/mobile ISP ASN:** Claimed yes — IPs "sourced from household devices" / from ISPs rather than datacenter ranges ([native residential page](https://cometvpn.com/features/residential-vpn/native/), [dedicated IP page](https://cometvpn.com/features/dedicated-ip/)). These are the vendor's own claims; the ASN should be checked with ipinfo.io after the first connect before trusting it.
- **DNS-leak protection:** Yes — dedicated DNS-leak-protection feature with private/encrypted DNS ([feature page](https://cometvpn.com/features/dns-leak-protection/)).
- **Sticky-session support:** Yes — sticky IPs for up to 24 hours, or rotating per request; a dedicated (single-user) residential IP add-on is also offered ([native residential page](https://cometvpn.com/features/residential-vpn/native/), [dedicated IP page](https://cometvpn.com/features/dedicated-ip/)).
- **Fit verdict:** Fits the site's criteria on paper (residential IPs, all four countries, DNS-leak protection, 24 h sticky) — but every claim is self-published by a smaller provider, so treat ASN verification (ipinfo.io → should show a consumer ISP, not a hosting company) as mandatory before relying on it.

## Option 4 — Proxora Rotating Mobile (and residential)

- **Category:** mobile-carrier IP service
- **Official website:** https://proxora.io/proxies/rotating-mobile
- **Supported exit countries (US/SG/JP/TW):** Yes — dedicated location pages for all four each list a "Rotating Mobile" product: [United States](https://proxora.io/locations/united-states), [Singapore](https://proxora.io/locations/singapore), [Japan](https://proxora.io/locations/japan), [Taiwan](https://proxora.io/locations/taiwan); the network spans 200+ countries ([mobile page](https://proxora.io/proxies/rotating-mobile)).
- **Genuine residential/mobile ISP ASN:** Yes (mobile) — "3G/4G/5G/LTE addresses assigned by real carriers"; the Japan page names NTT Docomo, au (KDDI), SoftBank and Rakuten Mobile, and the Taiwan page names Chunghwa Telecom and Taiwan Mobile among 27 local ISPs/carriers. Caveat from the same pages: live ASNs rotate with pool availability, so carrier choice is a pool, not a fixed guarantee ([Japan page](https://proxora.io/locations/japan), [Taiwan page](https://proxora.io/locations/taiwan)).
- **DNS-leak protection:** Yes (configuration-based) — official help explicitly says to use `socks5h://` so DNS resolves through the proxy ("prevents DNS leaks"), and Proxora hosts its own DNS-leak test tool ([SOCKS5 setup](https://proxora.io/help/set-up-socks5), [DNS leak test](https://proxora.io/tools/dns-leak-test)).
- **Sticky-session support:** Yes — per-request rotation or sticky sessions from 3 to 240 minutes ([mobile page](https://proxora.io/proxies/rotating-mobile)).
- **Fit verdict:** Fits — carrier-ASN IPs in all four countries with documented DNS handling and sticky sessions; rated low-mid (not low) risk by the site because mobile IPs are CGNAT-shared and rotate, so keep sessions ≤4 h and don't switch countries mid-flow.

## Option 5 — NordVPN Dedicated IP

- **Category:** VPN offering dedicated IPs (not residential-ISP IPs)
- **Official website:** https://support.nordvpn.com/hc/en-us/articles/19507808024209-Dedicated-NordVPN-IP-addresses
- **Supported exit countries (US/SG/JP/TW):** US (11 cities), Singapore and Japan (Tokyo) yes; **Taiwan is not in the published region list** (Hong Kong is the closest listed location) ([dedicated-IP article](https://support.nordvpn.com/hc/en-us/articles/19507808024209-Dedicated-NordVPN-IP-addresses)).
- **Genuine residential/mobile ISP ASN:** No (by its own documentation) — the dedicated IP is "an IP address assigned to a specific server" of NordVPN's own server fleet, i.e. a VPN-server address; the article makes no claim of ISP/consumer-ASN sourcing. That is exactly the "IP 标记为住宅但实际是数据中心段 / VPN 池" failure mode the site warns about ([site criteria](https://fuck-claude.vercel.app/zh/guides/vpn-and-proxy-selection/)). Confirm with ipinfo.io before use if considered.
- **DNS-leak protection:** Yes — NordVPN apps use NordVPN's private DNS automatically, "which prevents DNS leaks during your VPN connection" ([DNS article](https://support.nordvpn.com/hc/en-us/articles/19587726859793-What-are-NordVPN-DNS-server-addresses)).
- **Sticky-session support:** Yes by design — the IP is static and dedicated to the account (changeable only once per subscription via support) ([dedicated-IP article](https://support.nordvpn.com/hc/en-us/articles/19507808024209-Dedicated-NordVPN-IP-addresses)).
- **Fit verdict:** Poor fit — stable and DNS-safe, but the ASN is a VPN server rather than a residential ISP (the exact high-risk class the site flags) and Taiwan is missing, so it does not clear the site's core criterion.

---

## Does Astrill itself offer residential/dedicated-IP options?

Verified on Astrill's own site: **Astrill offers a dedicated ("Private") IP add-on, but no residential-ISP IP option.**

- Astrill's dedicated-IP page says "Selected VPN servers allow allocation of dedicated IP (aka private IP). Such IP is used only by you and no other Astrill clients" — i.e. an exclusive IP **on Astrill's own VPN servers**, the same infrastructure class as the current AS55081 hosting exit ([astrill.com/network/dedicated-ips](https://www.astrill.com/network/dedicated-ips)).
- The pricing page lists it as an extra feature: "Private IP on specific server — Get dedicated IP on selected servers that is used only by you and nobody else. Private IP is static and stays the same for the duration of your membership" ([astrill.com/pricing](https://www.astrill.com/pricing)).
- The FAQ confirms: "Do you offer Private / dedicated IP? Yes, we offer Private IP as an add-on for additional charge per month on top of the basic plan" ([astrill.com/faq](https://www.astrill.com/faq)).
- Nothing on Astrill's site, features page or docs claims IPs sourced from consumer/residential ISPs — the features page describes "our vast network of super powerful dedicated servers" and its own "Private IP" concept ([astrill.com/features](https://www.astrill.com/features)).

**Conclusion:** buying Astrill's Private IP would remove sharing with other users but would not change the egress ASN class (Astrill-operated server, hosting-class), so it does not satisfy the site's residential-ASN criterion.

---

## Summary table — options ranked by fit

| Rank | Option | Category | US/SG/JP/TW | IP is genuine residential/mobile ASN? | DNS-leak protection | Sticky sessions | Fit vs site criteria |
|---|---|---|---|---|---|---|---|
| 1 | Oxylabs Residential Proxies | residential proxy service | ✅ all four (dedicated entry nodes) | ✅ ISP-sourced, ASN-targetable | ✅ exit-side/UDP DNS at exit | ✅ up to 24 h, `sessid_oneip` | **Best fit** — meets every criterion |
| 2 | Bright Data Residential Proxies | residential proxy service | ✅ all four (195 countries) | ✅ real residential devices, carrier/ASN targeting | ✅ `-dns-remote` | ✅ session-based (5 min idle window) | **Strong fit** — corporate KYC is the only friction |
| 3 | CometVPN Native Residential VPN | VPN offering residential IPs | ✅ all four (locations list) | ⚠️ claimed ISP-sourced; vendor-stated, needs ipinfo check | ✅ dedicated feature | ✅ up to 24 h / dedicated IP | Fits on paper; verify ASN after first connect |
| 4 | Proxora Rotating Mobile | mobile-carrier IP service | ✅ all four (per-country pages) | ✅ real carrier ASNs (Docomo/Chunghwa/Taiwan Mobile etc.) | ✅ `socks5h` remote DNS + own leak test | ✅ 3–240 min | Fits (mobile = low-mid risk); shorter sticky window |
| 5 | NordVPN Dedicated IP | VPN offering dedicated IPs | ⚠️ US/SG/JP only — no TW | ❌ VPN-server IP, no ISP-sourcing claim | ✅ private DNS in apps | ✅ static by design | Fails core ASN criterion + missing TW |
| — | Astrill Private IP (current provider) | VPN offering dedicated IPs | — (server-dependent) | ❌ Astrill's own servers; no residential option advertised | ✅ claims DNS/WebRTC leak fix | ✅ static per membership | Fails ASN criterion — does not fix the gap |
