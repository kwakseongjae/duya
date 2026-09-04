# Email Preview · HTML/CSS Pitfalls (2026)

**Date:** 2026-09-04 KST  
**Angle:** Under-covered gap — `03-ui-shadcn-i18n.md` covers sandboxed iframe UX; this doc covers **what HTML will actually look like in real clients**, why Chrome preview lies, and product rules for Compose / Templates / Inbox.  
**Constraints:** Research only · no implementation · no fabricated metrics  

---

## TL;DR for product

1. **In-app preview is a security + draft QA surface, not a Litmus replacement.** Chromium iframe ≠ classic Outlook Word engine ≠ Gmail sanitizer.
2. **Cold templates should be boring HTML:** tables / `Row`+`Column`, inline critical CSS, max-width ~600px, PNG/JPEG only, body under ~80 KB raw HTML before ESP rewrite.
3. **Show size + plain-text in Compose** (clip risk, a11y, AI summaries). Soft-warn when HTML approaches Gmail’s ~102 KB clip threshold.
4. **Inbox HTML is hostile:** sanitize + sandbox + CSP; remote images opt-in; never `dangerouslySetInnerHTML` in the parent app.
5. **Engine coverage for seed tests:** classic Outlook Windows, Apple Mail iPhone, Gmail web, Gmail iOS, new Outlook preview pane — one pass per engine beats twenty random screenshots.

---

## 1. Why the Issue-board preview will lie

| What the user sees | What actually renders |
|---|---|
| App dark theme + sandboxed iframe (forced light canvas) | Recipient client theme + client dark-mode inversion |
| Blink/WebKit in Chrome/Safari desktop | Word (classic Outlook), Gmail sanitizer, WebView2 post-process (new Outlook) |
| Full `srcDoc` HTML | Gmail may strip entire `<style>` blocks; GANGA (Gmail app + non-Google IMAP) may drop embedded CSS entirely |
| Unbounded draft length | Gmail clips ~102 KB raw HTML; clipped “View entire message” can drop `<style>` again |

**Product implication:** Label preview as “브라우저 미리보기 (실제 클라이언트와 다를 수 있음)” and keep a “Plain text” toggle + byte-size meter. Optional later: Litmus / Mailgun Inspect export — not MVP.

Sources: https://email-dev.com/email-rendering-issues/ · https://making.close.com/posts/rendering-untrusted-html-email-safely/ · https://www.caniemail.com/

---

## 2. Five engines (not forty clients)

From Email Developer’s 2026 map — debug by **engine**, not app name:

| Engine | Where | Breaks | Survival rule |
|---|---|---|---|
| Word / MSO | Classic Outlook for Windows | No flex/grid/`border-radius`; CSS bg-images need VML; padding on divs unreliable | Tables, HTML `width`/`height` on images, MSO conditionals |
| WebKit | Apple Mail macOS/iOS, Outlook for Mac | Little | Full media queries / `prefers-color-scheme` mostly here |
| Blink / Chromium | Outlook.com, OWA | Own dark-mode logic | Test separately from Apple Mail |
| WebView2 | New Outlook for Windows | Chromium + post-load CSS rewrite; preview-pane-only bugs reported | Even `border-radius`; avoid relying on MQ alone |
| Gmail hybrid | Gmail web / iOS / Android | Aggressive sanitizer; size clip | Small HTML, boring CSS, inline critical styles |

**Outlook is not one client.** Emailens (July 2026 snapshot, caniemail data synced Sep 2 2026): Outlook for Windows ~19% full feature support vs Apple Mail ~94%; **91 features** work in Outlook on the web but fail on Outlook for Windows. Always ask “which Outlook?”

Sources: https://email-dev.com/email-rendering-issues/ · https://emailens.dev/email-css/report · https://www.caniemail.com/

---

## 3. CSS / HTML do’s and don’ts (cold + transactional)

### 3.1 Prefer

- Table / React Email `Row`+`Column` layout; container `width: 100%` + `max-width: 600px`
- **Inline** styles for anything that must survive if `<style>` is stripped
- HTML `width` and `height` attributes on every `<img>` (classic Outlook ignores CSS sizing)
- `display: block` on images; even pixel heights for sliced heroes
- Bulletproof CTA: padded table cell + text link (not an image button)
- Body copy ≥ 16px; `-webkit-text-size-adjust: 100%` on body
- Absolute `https://` image URLs; PNG/JPEG (avoid SVG/WEBP in templates)
- Meaningful live HTML text near the top (AI summaries ignore alt / image-baked text)
- Parallel plain-text MIME part

### 3.2 Avoid or treat as progressive enhancement

- Flexbox / Grid as primary layout (`display:flex` may survive; sub-properties often stripped — tables still win)
- CSS Grid (widely unsupported in major clients)
- External stylesheets / `@import`
- Nested at-rules (`@font-face` inside `@media`) — can nuke Gmail’s whole `<style>` block
- Single `<style>` block > **8,192 characters** (Gmail drops the whole block)
- One CSS parse error → Gmail may discard the **entire** style block
- `rem` units without conversion (React Email: use `pixelBasedPreset`)
- Tailwind responsive/`dark:` prefixes as if they were web — limited or unsupported in email
- Pure `#000` / `#FFF` (dark-mode inversion uglier); prefer near-black / near-white
- Critical content only in background images or hero JPG logos (white-box / blocked-image failure)

### 3.3 Media queries

- Apple Mail: most reliable full support
- Gmail / modern Outlooks: partial
- Classic Outlook Windows: ignores (fixed width)
- **Rule:** base layout must work **without** MQ; MQ only stacks columns / grows tap targets

Sources: https://www.caniemail.com/ · https://emailens.dev/email-css/report · https://designmodo.com/html-css-emails/ · https://github.com/resend/react-email/blob/main/skills/react-email/references/STYLING.md · https://raw.githubusercontent.com/resend/resend-skills/refs/heads/main/skills/react-email/SKILL.md

---

## 4. Gmail clipping (must be a product warning)

| Surface | Approx. raw HTML threshold | Notes |
|---|---|---|
| Gmail desktop / web | ~**102 KB** | Images external don’t count; markup + tracking URLs do |
| Gmail iOS (cited EO A 2014 test) | ~**20 KB**, inconsistent | Treat as high-risk; keep cold copy short |
| Practical pre-ESP target | ~**80 KB** | ESP link rewrite + pixels often add 20–30 KB |

**Secondary damage when clipped:**

- “View entire message” may strip `<style>` → ugly full view
- Tracking pixel at bottom may never load → fake “open rate” drop

**Product UX:**

- Show `HTML size: N KB` under Compose / Template editor
- Soft warn ≥ 80 KB; hard warn ≥ 95 KB before Approve
- Prefer short cold emails anyway (compliance + reply rates); don’t fight clip with bloat

Sources: https://email-dev.com/email-rendering-issues/ · https://email-dev.com/gmail-rendering-issues/ · https://knak.com/blog/common-email-rendering-issues-and-how-to-fix-them/ · https://github.com/resend/react-email/issues/1588

---

## 5. Dark mode (quiet breakage)

Three behaviours (not one “dark mode”):

1. **No inversion** — light design stays light  
2. **Partial inversion** — Outlook.com / new Outlook / some Gmail  
3. **Full forced inversion** — Gmail Android, some Apple Mail configs  

`prefers-color-scheme` and `color-scheme` / `supported-color-schemes` meta: useful mainly for **Apple Mail** (+ partial Outlook). **Gmail does not support** custom dark overrides (caniemail: not supported for Gmail/Yahoo for years).

Fixes that hold up:

- Transparent PNG logos (+ thin outline), not JPG white boxes  
- Avoid pure black/white  
- Test one forced-inversion + one partial client  
- Do **not** solve with “single giant image” emails (a11y, blocked images, empty AI summaries)

Sources: https://email-dev.com/email-rendering-issues/ · https://www.caniemail.com/

---

## 6. Preview security (extends `03` §5.2)

Industry pattern (Close, AgentMail, etc.):

```html
<iframe
  srcdoc="…wrapped sanitized document…"
  sandbox="allow-popups allow-popups-to-escape-sandbox allow-same-origin"
  referrerpolicy="no-referrer"
></iframe>
```

| Control | Role |
|---|---|
| `sandbox` **without** `allow-scripts` | No JS even if sanitizer misses |
| `allow-same-origin` alone | Needed to measure content height; **never** pair with `allow-scripts` |
| `allow-popups` (+ escape) | Links open real tabs via `<base target="_blank">` |
| CSP meta in `srcDoc` | e.g. `default-src 'none'; img-src https: data:; style-src 'unsafe-inline'; script-src 'none'` |
| DOMPurify / server sanitize | Strip scripts, handlers, dangerous tags before wrap |
| Remote images | Default block / proxy; “Load images” opt-in (tracking + privacy) |

**Compose vs Inbox:**

- Compose: your templates — still sandbox; sanitize less aggressively but size-check  
- Inbox / inbound snippets: treat as hostile — full sanitize + remote-image gate  

Sources: https://making.close.com/posts/rendering-untrusted-html-email-safely/ · https://engineering.agentmail.to/blog/rendering-email-safely-preventing-phishing-attacks

---

## 7. React Email + Resend (stack-aligned)

Locked stack already points at Resend. When templates are authored:

- Use React Email components (`Html`, `Head`, `Body`, `Container`, `Section`, `Row`, `Column`, `Text`, `Button`, `Img`, `Preview`)
- Tailwind: **`pixelBasedPreset`** (no rem)
- No flex/grid; no `sm:`/`dark:` reliance
- Keep rendered HTML under 102 KB; Prefer short cold bodies
- Always ship plain-text alternative
- After ESP rewrite, re-test from **sent** message, not local file

Sources: https://raw.githubusercontent.com/resend/resend-skills/refs/heads/main/skills/react-email/SKILL.md · https://github.com/resend/react-email/blob/main/skills/react-email/references/STYLING.md

---

## 8. Product checklist — Issue board Compose / Templates

Ship as UI gates / copy (research → later build):

- [ ] Desktop 600 / Mobile 375 device chrome around iframe (already in `03`)
- [ ] Forced light iframe canvas in dark app
- [ ] Plain-text toggle beside Preview
- [ ] HTML byte meter + 80/95 KB warnings
- [ ] “실제 Gmail·Outlook과 다를 수 있음” helper text
- [ ] Approve path requires subject + body non-empty + (optional) plain-text present
- [ ] Template lint (soft): missing `alt`, missing img width/height, SVG/WEBP detect, external stylesheet detect
- [ ] Seed-send helper (Phase 1): send test to user’s own Gmail / Outlook — not full Litmus
- [ ] Inbox: remote images off by default

---

## 9. Testing ladder (cheap → paid)

1. Open HTML in Chrome (structure only)  
2. Seed send: Gmail + Outlook + iPhone  
3. New Outlook: check **preview pane vs separate window** (post-process bug class)  
4. Dark mode: one forced + one partial  
5. Measure HTML size **after** Resend/ESP rewrite  
6. Optional paid screenshots: Mailgun Inspect / Litmus — pricing moves often; verify current plans before buying  

Do **not** treat in-app iframe as step 3–6.

Source: https://email-dev.com/email-rendering-issues/

---

## 10. Implications for 성재 MVP

| Decision | Recommendation |
|---|---|
| Template authoring | React Email + table/`Row`/`Column`; keep cold HTML tiny |
| Preview | Sandboxed iframe + plain-text + size meter (from `03` + this doc) |
| Cross-client QA | Seed inboxes first; screenshot SaaS later |
| Agency HTML paste | Sanitize hard; warn “Outlook Word / Gmail clip risk” |
| Tracking pixels | Prefer reply detection over open pixels; if pixel used, don’t put only at bottom of huge HTML |
| Dark mode polish | Transparent logos + near-neutral colors; don’t over-invest in Gmail dark overrides |

---

## 11. Source index

| URL | Use |
|---|---|
| https://www.caniemail.com/ | CSS/HTML support matrix (primary) |
| https://emailens.dev/email-css/report | 2026 state-of-email-CSS synthesis (caniemail-derived; Jul 2026 / sync Sep 2 2026) |
| https://email-dev.com/email-rendering-issues/ | Engine map, Outlook dual-client, Gmail clip, dark mode, test ladder |
| https://email-dev.com/gmail-rendering-issues/ | Gmail sanitizer + clip detail |
| https://email-dev.com/background-images-in-html-email/ | VML / bg-image bulletproofing |
| https://knak.com/blog/common-email-rendering-issues-and-how-to-fix-them/ | Clip + dark mode narrative |
| https://www.codexical.com/posts/2026-07-22-email-css-client-fragmentation | Dual-Outlook / permanent fragmentation thesis |
| https://designmodo.com/html-css-emails/ | Tables vs flex/grid in email |
| https://www.litmus.com/blog/understanding-media-queries-in-html-email | MQ support caveats (Litmus; verify dates) |
| https://making.close.com/posts/rendering-untrusted-html-email-safely/ | iframe sandbox + srcdoc + CSP pattern |
| https://engineering.agentmail.to/blog/rendering-email-safely-preventing-phishing-attacks | Sandbox + CSP for phishing defense |
| https://raw.githubusercontent.com/resend/resend-skills/refs/heads/main/skills/react-email/SKILL.md | React Email / Resend authoring rules |
| https://github.com/resend/react-email/blob/main/skills/react-email/references/STYLING.md | Styling don’ts (rem, flex, MQ) |
| https://github.com/resend/react-email/issues/1588 | Large-body style loss ↔ Gmail size behaviour |

**Uncertainty:** Exact Gmail iOS clip character count rests on older Email on Acid experiments; treat as “much lower than desktop, keep short.” Litmus/open-rate market shares cited by secondary blogs — use ESP’s own client breakdown for product decisions, not global averages.

---

## 12. Open questions (non-blocking)

1. Phase 0: paste-HTML templates vs React Email-only? (Recommend React Email primary; paste as advanced + lint.)  
2. Proxy remote images in Inbox (privacy) vs load-on-click only?  
3. When (if ever) to budget Litmus/Mailgun Inspect vs founder seed devices?
