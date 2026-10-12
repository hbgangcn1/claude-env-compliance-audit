# R5 — Third-party options to hide OS-installed Chinese fonts from canvas font-fingerprint probes

Research date: 2026-10-12 · Scope: web research only · Machine: Windows (OS-installed fonts 微软雅黑/msyh, 等线/DengXian, simhei, simsun, STKAITI cannot be removed by browser-local settings)

## 1. The gap (one line)

The site scores **"installed Chinese fonts" at weight 14**, detected **via canvas width probing** for fonts like 微软雅黑 — and because the fonts are OS-installed, no local browser setting can remove that signal ([detector homepage](https://fuck-claude.vercel.app/zh/), [environment-cleanup guide](https://fuck-claude.vercel.app/zh/guides/environment-cleanup-and-ip-setup/)).

## 2. How the probe works, and the three ways software defeats it

The probe draws text with a candidate font name on a `<canvas>` and compares rendered widths (or the `TextMetrics` readout) against a fallback — a font that exists on the machine produces a different width than one that does not. There are exactly three defensible mechanisms, and candidates differ on which they actually implement:

| Mechanism | What it does | What it defeats |
|---|---|---|
| **Font-list masking** | Spoofs the font *enumeration/list* APIs so the profile reports a curated, OS-appropriate list instead of the real installed set | CSS/JS font-enumeration scripts; claimed by vendors to cover render-based probes too (see GoLogin's wording) |
| **Canvas noise / readout faking** | Perturbs canvas pixel read-back (`getImageData`, `toDataURL`) or fakes `TextMetrics`/measurement results | Pixel-based width probes directly; `measureText`-based probes only if text metrics are spoofed (CanvasBlocker and Chameleon do this; no anti-detect vendor documents it) |
| **Font substitution / absence** | The font genuinely is not available to the browser, so rendering falls back | Everything — but on Windows this means uninstalling OS fonts, with UI breakage (section 3) |

**Important honesty note:** none of the anti-detect browsers below perform font *substitution* — real glyph rendering inside their browsers still uses the machine's installed fonts. Their documented defenses are font-list masking plus canvas noise. Only the extensions in the free route explicitly document spoofing `TextMetrics` (the exact readout a width probe consumes).

---

## 3. Candidate options

### 3.1 AdsPower

- **Category:** Anti-detect browser (fingerprint browser)
- **Official URL:** https://www.adspower.com/
- **HOW it defeats canvas font enumeration on Windows:**
  - *Font-list masking:* per-profile `fonts` parameter lets you supply a custom font list (e.g. `["Arial","Calibri","CambRIA"]`) or `"all"`, so sites are shown a chosen list rather than the machine's real one.
  - *Canvas noise:* the `canvas` parameter defaults to `1 = add noise` (vs `0 = real machine canvas`), which perturbs pixel read-back of canvas-drawn text.
  - *Not documented:* font substitution or `TextMetrics` spoofing — a pure `measureText()` width probe is not covered by any AdsPower doc found.
- **Evidence (official):**
  - Local API fingerprint config: https://localapi-doc-en.adspower.com/docs/Awy6Dg — "fonts … Font of the browser. Allows to customize font, format: string array"; "canvas … 1: Add noise (Default setting); 0: Use the current computer default Canvas"
  - Open-source config reference: https://github.com/AdsPower/adspower-browser/blob/main/skills/adspower-browser/references/fingerprint-config.md — same `fonts` / `canvas` parameters
  - Profile creation with fingerprint section: https://help.adspower.com/docs/creating_browser_profiles
- **Free vs paid:** Free plan **forever, 2 profiles**, plus a 7-day free trial of paid features — https://www.adspower.com/pricing
- **Fit verdict:** Best-fit paid-class option for a single user: the free tier covers one or two dedicated profiles, and both required mechanisms (font list + canvas noise) exist and default sensibly. Verify with the site's own detector after setup, since `measureText` spoofing is undocumented.

### 3.2 GoLogin

- **Category:** Anti-detect browser (fingerprint browser)
- **Official URL:** https://gologin.com/
- **HOW it defeats canvas font enumeration on Windows:**
  - *Font-list masking (explicit):* "When on, Gologin presents the profile's own curated font list to websites instead of the fonts actually installed on your machine" — the list is editable per profile.
  - GoLogin's docs explicitly frame the threat as *render-based* detection ("Websites can detect fonts by measuring how text renders on the page") and state "Gologin can mask the fonts list to generate realistic combinations that match the selected operating system" — i.e. the vendor claims this masking covers the exact class of probe the site uses.
  - *Canvas:* configurable canvas signal (including an "Off/Block/Add Noise" choice; GoLogin's own troubleshooting advises "Add Noise" vs "Off" depending on whether cross-device consistency matters).
  - *Not documented:* font substitution or `TextMetrics` spoofing.
- **Evidence (official):**
  - Profile fingerprint settings: https://support.gologin.com/en/articles/14810056-profile-fingerprint-settings
  - Advanced profile settings (font-list masking toggle, curated editable list): https://support.gologin.com/en/articles/14854407-advanced-profile-settings
- **Free vs paid:** Free plan: **3 profiles** with all benefits except profile sharing / team members — https://gologin.com/pricing (paid from $24/mo for 10 profiles)
- **Fit verdict:** Strong fit. GoLogin is the only vendor whose official docs directly claim font masking covers render-measurement detection, and its free tier (3 profiles) exceeds a single-account need. Caveat unchanged: masking claims are vendor-stated, so re-test against the detector.

### 3.3 Multilogin

- **Category:** Fingerprint browser (anti-detect browser)
- **Official URL:** https://multilogin.com/
- **HOW it defeats canvas font enumeration on Windows:**
  - *Font-list masking (default on):* "Font data … Masked (default): Multilogin exposes an optimized font setup based on the browser profile's OS." Docs also acknowledge probes "measure how text is displayed."
  - *Canvas noise (optional):* "Canvas graphics … Noise: Multilogin adds persistent noise to Canvas read-outs" — but **the default is "Real"**, so canvas noise must be opted into manually; "Disabled" is a legacy, discouraged option.
  - *Not documented:* font substitution or `TextMetrics` spoofing.
- **Evidence (official):**
  - Fingerprint section (Font data + Canvas graphics): https://multilogin.com/help/fingerprints/software-settings?kb_language=en_US
  - Fingerprint FAQ (what is/isn't masked): https://multilogin.com/help/en_US/fingerprints-faq
- **Free vs paid:** Now has a **free plan with 5 profiles** (no credit card); paid Pro from **$7.08/mo billed annually** — https://multilogin.com/pricing
- **Fit verdict:** Capable but defaults work against this specific gap (canvas = Real out of the box; font masking is on). Team/orientation and pricing make it overkill for one personal account, but the free 5-profile plan makes it a legitimate trial option.

### 3.4 Dolphin {anty}

- **Category:** Anti-detect browser (fingerprint browser)
- **Official URL:** https://dolphin-anty.com/
- **HOW it defeats canvas font enumeration on Windows:**
  - *Font-list masking:* "Fonts" is listed among 20+ spoofed parameters; the product blog describes font spoofing that "generates unique font sets for each [profile]" rather than reporting the machine's real set.
  - *Canvas noise:* "Canvas" is likewise a spoofed parameter with a noise option.
  - *Documented limitation (official):* Dolphin's own setup guidance warns that fonts are "one of the main parameters that gives away the fact that the print has been spoofed" — a Windows profile claiming macOS will still fail font checks because "the difference will be visible not only in the font set itself, but also in their design." Its recommendation: **match the profile OS to your real OS** (or use a Linux second OS to smooth font differences) — i.e. Dolphin relies on consistent font masking, not on removing Chinese fonts.
  - *Not documented:* font substitution or `TextMetrics` spoofing.
- **Evidence (official):**
  - Spoofed-parameter list (OS, Canvas, Fonts, …): https://docs.dolphin-anty.com/en/working-with-fingerprints/digital-fingerprint-and-profile-settings-in-dolphin-anty
  - Setup guidance, Fonts + Canvas sections: https://docs.dolphin-anty.com/en/getting-started/guidelines-for-setting-up-a-profile-dolphin-anty
  - Font spoofing changelog: https://dolphin-anty.com/blog/en/dolphin-anty-updates-2/
- **Free vs paid:** Free plan **5 browser profiles** (Starter from $10/mo for 20) — https://dolphin-anty.com/en/pricing
- **Fit verdict:** Solid budget pick with the most generous free tier; its own docs are unusually candid that font masking must match the claimed OS and that over-noising canvas makes a profile stand out — follow that guidance rather than maxing every slider. (Maskfog, the alternative allowed by the brief, was not needed once Dolphin's documentation was verified.)

### 3.5 Free route: dedicated clean Chrome/Edge profile + a canvas/font anti-fingerprint extension

- **Category:** Clean-profile technique + browser extension
- **Official URLs:**
  - Site's own recommendation: https://fuck-claude.vercel.app/zh/ (step 3: "隔离中文与厂商字体 … 使用干净的浏览器 Profile 或防指纹浏览器") and https://fuck-claude.vercel.app/zh/guides/environment-cleanup-and-ip-setup/ (table: "已装 CJK 字体 → 尽量少或 Profile 隔离")
  - CanvasBlocker: https://canvasblocker.kkapsner.de/ · repo: https://github.com/kkapsner/CanvasBlocker
  - Chameleon (Firefox): https://github.com/00x127/Chameleon · store: https://addons.mozilla.org/addon/chameleonn/
  - Fingerprint Protector (Chrome/Edge): https://chromewebstore.google.com/detail/fingerprint-protector/lhnhapbalajnchknjmjlcpoiehjegbdm
- **HOW it defeats canvas font enumeration on Windows:**
  - A clean profile **alone does not hide the fonts** — it isolates cookies/language/other signals from your daily Chinese-configured browser; the font signal itself must be closed by the extension.
  - **CanvasBlocker (Firefox)** — *canvas noise + text-metrics faking*: official README lists `canvas 2d`, **`TextMetrics`**, `DOMRect`, `SVG`, `webGL`, `audio` as protected APIs, with the default "fake" mode so "values obtained by the APIs are altered so that a consistent fingerprinting is not possible." Faking `TextMetrics` is exactly what a canvas **width** probe consumes, and faking `canvas 2d` read-back defeats pixel-based probes. Firefox-only (not usable in Chrome/Edge).
  - **Chameleon — 00x127 (Firefox)** — *per-origin deterministic noise*: README lists "TextMetrics, getBoundingClientRect, …" under layout fingerprinting and "Canvas 2D readback (toDataURL, toBlob, getImageData)" under visual fingerprinting, with a stable per-origin identity so repeated probes return the same (fake) values. Font surface is named among the blocked surfaces.
  - **Fingerprint Protector (Chrome/Edge)** — store page lists modules for **Canvas, fonts, ClientRects**, screen, timezone, etc., with a "stable, seed-based virtual identity" so spoofed values stay consistent across tabs — the free extension route for Chrome/Edge users (CanvasBlocker and Chameleon 00x127 are Firefox-only).
  - **Warning — do not use "Chameleon" by ghostwords for this:** that Chrome extension is **ARCHIVED**, pre-alpha, and its own README says "Chameleon does not yet protect against [canvas fingerprinting]" and lists font enumeration as *detection only, protection ✗* — https://github.com/ghostwords/chameleon
- **Evidence (official):** GitHub READMEs cited above; Firefox store page for Chameleon (https://addons.mozilla.org/addon/chameleonn/); Chrome Web Store page for Fingerprint Protector; CanvasBlocker project site https://canvasblocker.kkapsner.de/
- **Free vs paid:** **Free / open source** (CanvasBlocker MPL-2.0, Chameleon 00x127 MIT) or **free store extension** (Fingerprint Protector — no paid tier stated on its store page).
- **Fit verdict:** **Best fit for this user**, and it is what the source site itself says: its FAQ states that for a single account an individual user may not need an anti-detect browser — an independent Chrome/Firefox profile with aligned timezone/language, WebRTC disabled and stable residential IP is "enough for most individual users." The extension layer adds the one thing a clean profile cannot: corrupted `TextMetrics`/canvas read-backs so the width probe sees noise instead of 微软雅黑's metrics. Cost: $0; only caveat is Firefox-only for the two best-documented tools, so Chrome/Edge users should take Fingerprint Protector or use Firefox for the Claude session.

---

## 4. Is deleting/hiding Windows fonts viable?

**Site's stance (verified):** No. The homepage's remediation list says to *isolate* the fonts — "使用干净的 Browser Profile 或防指纹浏览器，让 canvas 字体探测找不到任何明显的中文痕迹" (use a clean profile or anti-detect browser so canvas probes find no obvious Chinese traces) — and the guide's checklist says for installed CJK fonts: "尽量少或 Profile 隔离" (keep as few as possible **or** isolate by Profile). The site never instructs uninstalling system fonts ([homepage](https://fuck-claude.vercel.app/zh/), [guide](https://fuck-claude.vercel.app/zh/guides/environment-cleanup-and-ip-setup/)).

**Windows reality (verified):**
- Windows Settings → Personalization → Fonts does offer **Hide** and **Uninstall** per font ([Microsoft: Manage Fonts in Windows](https://support.microsoft.com/en-us/windows/experience/personalization/manage-fonts-in-windows)). Microsoft documents **Hide** purely as list organization ("hide them … without permanently removing the fonts") — there is **no Microsoft claim that a hidden font becomes invisible to web content or canvas probes**; the font files remain installed ([Microsoft typography FAQ](https://learn.microsoft.com/en-us/typography/fonts/font-faq) likewise treats "hidden fonts" as fonts that don't show in menus but are still present).
- Many system fonts are protected and simply cannot be removed: "Some content is protected by the system and indeed cannot be deleted" ([Microsoft Q&A](https://learn.microsoft.com/en-us/answers/questions/3925550/cannot-be-deleted-because-it-is-a-protected-system)); fonts in active use are also file-locked while the UI/renderer is using them.
- **UI breakage risk:** msyh/simsun/simhei/DengXian are the default rendering fonts for Chinese text in Windows UI, Explorer, Edge/Chrome page fallback, and Office. Uninstalling them means tofu boxes or arbitrary fallback wherever Chinese renders — including this machine's everyday Chinese use — and Windows Update/component repair can restore them. Fonts are **per-machine, not per-browser-profile**, which is exactly why local settings cannot fix the gap.

**Verdict:** Not viable. Removing OS CJK fonts would trade a weight-14 fingerprint signal for a broken Chinese-language desktop, some fonts cannot be deleted at all, and "hidden" fonts are not documented as hidden from canvas probes. Follow the site's recommendation: profile isolation (plus, if desired, an extension that poisons the measurement readouts).

---

## 5. Summary table — ranked by fit (single-account personal use, Shenzhen)

| Rank | Option | Category | Mechanism vs canvas width probe | Free vs paid | Why it ranks here |
|---|---|---|---|---|---|
| 1 | **Clean Chrome/Edge/Firefox profile + extension** (Fingerprint Protector on Chrome/Edge; CanvasBlocker / Chameleon 00x127 on Firefox) | Clean-profile technique + browser extension | Canvas read-back noise **+ TextMetrics faking** (directly corrupts the width measurement) | **Free** (open source / free store listing) | Matches the source site's own FAQ for single-account users; $0; the only documented defense against `measureText`-based probes |
| 2 | **AdsPower** | Anti-detect browser | Font-list masking + canvas noise (default on) | Free plan: **2 profiles** | Both mechanisms present and documented; small free tier is still enough for one account |
| 3 | **GoLogin** | Anti-detect browser | Font-list masking (explicitly claims render-based coverage) + canvas modes | Free plan: **3 profiles** | Only vendor officially claiming its font masking addresses "how text renders" detection |
| 4 | **Dolphin {anty}** | Anti-detect browser | Font spoofing + canvas noise; official guidance = match profile OS to real OS | Free plan: **5 profiles** | Most generous free tier and candid docs, but advises *against* heavy canvas noise |
| 5 | **Multilogin** | Fingerprint browser | Font data Masked by default; canvas noise available but **defaults to Real** | Free plan: 5 profiles; paid from $7.08/mo | Requires manual opt-in for the canvas layer that matters here; team-oriented |
| — | Deleting/hiding Windows fonts | System-level (not recommended) | Font absence/substitution — would fully defeat the probe | n/a | Protected/in-use fonts can't be removed; breaks Chinese UI everywhere; site recommends profile isolation instead |

**Fair representation of the site's position:** for one personal account, route #1 (clean profile + aligned settings + an extension for the font signal) is the site's own recommendation; anti-detect browsers (#2–#5) are positioned by the site for multi-account or corporate-laptop scenarios where system region settings can't be changed.

---

### Source URLs cited

1. https://fuck-claude.vercel.app/zh/
2. https://fuck-claude.vercel.app/zh/guides/environment-cleanup-and-ip-setup/
3. https://www.adspower.com/
4. https://localapi-doc-en.adspower.com/docs/Awy6Dg
5. https://github.com/AdsPower/adspower-browser/blob/main/skills/adspower-browser/references/fingerprint-config.md
6. https://help.adspower.com/docs/creating_browser_profiles
7. https://www.adspower.com/pricing
8. https://gologin.com/
9. https://support.gologin.com/en/articles/14810056-profile-fingerprint-settings
10. https://support.gologin.com/en/articles/14854407-advanced-profile-settings
11. https://gologin.com/pricing
12. https://multilogin.com/
13. https://multilogin.com/help/fingerprints/software-settings?kb_language=en_US
14. https://multilogin.com/help/en_US/fingerprints-faq
15. https://multilogin.com/pricing
16. https://dolphin-anty.com/
17. https://docs.dolphin-anty.com/en/working-with-fingerprints/digital-fingerprint-and-profile-settings-in-dolphin-anty
18. https://docs.dolphin-anty.com/en/getting-started/guidelines-for-setting-up-a-profile-dolphin-anty
19. https://dolphin-anty.com/blog/en/dolphin-anty-updates-2/
20. https://dolphin-anty.com/en/pricing
21. https://canvasblocker.kkapsner.de/
22. https://github.com/kkapsner/CanvasBlocker
23. https://github.com/00x127/Chameleon
24. https://addons.mozilla.org/addon/chameleonn/
25. https://chromewebstore.google.com/detail/fingerprint-protector/lhnhapbalajnchknjmjlcpoiehjegbdm
26. https://github.com/ghostwords/chameleon
27. https://support.microsoft.com/en-us/windows/experience/personalization/manage-fonts-in-windows
28. https://learn.microsoft.com/en-us/typography/fonts/font-faq
29. https://learn.microsoft.com/en-us/answers/questions/3925550/cannot-be-deleted-because-it-is-a-protected-system
