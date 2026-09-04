# Deliverability + Legal Compliance — Research Notes

**Product:** Cold outreach Issue Board SaaS (Resend Phase 0 → Gmail Phase 1)  
**Audience:** Product/engineering (Korean founders sending US/EU B2B)  
**Research date:** 2026-09-04 (Asia/Seoul)  
**Status:** **Research notes only — not legal advice.** Confirm with counsel for each target jurisdiction before shipping public send. Marked **uncertain** where law varies by member state or secondary summaries were used.

Related: `02-stack-blueprint.md`, `04-master-roadmap.md`, `issue-board-ia.md`

---

## TL;DR for product

| Priority | Gate | Why |
|---|---|---|
| P0 | DNS wizard (SPF+DKIM+DMARC) must pass before first public send | Google/Yahoo/Microsoft bulk-sender auth |
| P0 | One-click List-Unsubscribe (RFC 8058) + body unsub + honor ≤48h (Gmail/Yahoo) / ≤10 business days (CAN-SPAM) | Provider + US law |
| P0 | Bounce/complaint suppression + auto-pause | Resend quotas + Gmail spam-rate rules |
| P0 | Daily/hourly caps + warm-up schedule | Resend warm-up docs + Workspace limits |
| P0 blocker | **Resend AUP prohibits cold outreach / unsolicited mail** | Must plan Phase 1 Gmail path or alternate ESP for true cold send |
| P0 | Agent cannot send without human Approve (HITL) | Roadmap + abuse risk |
| Legal UX | Separate “lawful basis” copy for US / UK-EU / CA / KR | Different consent vs LI regimes |

---

## 1. Product features that MUST exist before public send

These are **product gates**, not optional polish. Sources: Google Email sender guidelines + FAQ, Microsoft high-volume sender blog + NDR 550 5.7.515, Yahoo Sender Hub, Resend quotas/AUP, FTC CAN-SPAM guide.

### 1.1 DNS / authentication wizard (block send until green)

| Check | Requirement (research summary) | Product UX |
|---|---|---|
| **SPF** | All senders: SPF *or* DKIM. Bulk (≥~5k/day to Gmail personal): **both** SPF and DKIM. Microsoft high-volume: SPF **and** DKIM must pass. | Show TXT to add; poll dig; fail closed |
| **DKIM** | Bulk: required; Gmail recommends ≥1024-bit (prefer 2048). Align `d=` with From organizational domain. | Selector + CNAME/TXT from Resend; verify signature on test send |
| **DMARC** | Bulk Gmail/Yahoo/Microsoft: publish DMARC with at least `p=none`. Alignment: From domain aligns with SPF **or** DKIM org domain. | Default wizard to `p=none` + `rua=`; warn before `quarantine`/`reject` |
| **From alignment** | Bulk: RFC5322 From must align with SPF or DKIM domain (Gmail FAQ). Microsoft: DMARC validation must pass for high-volume. | Reject send if From domain ≠ verified sending domain |
| **PTR / FCrDNS** | Required (ESP owns shared IPs; dedicated IP needs PTR). | Informational for shared IP; checklist item for dedicated |
| **TLS** | Gmail: TLS required for transmission (ESP responsibility). | Status: “handled by Resend” |

**Citations**

- Google Email sender guidelines: https://support.google.com/mail/answer/81126  
- Google Email sender guidelines FAQ: https://support.google.com/mail/answer/14229414  
- Microsoft high-volume senders (May 2025 enforcement; 550 5.7.515): https://techcommunity.microsoft.com/blog/microsoftdefenderforoffice365blog/strengthening-email-ecosystem-outlook%e2%80%99s-new-requirements-for-high%e2%80%90volume-senders/4399730  
- Microsoft NDR 550 5.7.515: https://support.microsoft.com/en-us/outlook/fix-ndr-error-550-5-7-515-in-outlook-com  
- Yahoo Sender Hub best practices: https://senders.yahooinc.com/best-practices/

**Bulk-sender definition (Gmail):** ~5,000+ messages/day to *personal* `@gmail.com` / `@googlemail.com` from the same primary domain; once classified, **permanent**. Guidelines do **not** apply to mail *to* Google Workspace accounts (FAQ). Workspace *from* accounts still must meet guidelines when sending *to* personal Gmail.

### 1.2 Sending caps + warm-up (block or throttle)

| Cap | Suggested product default | Source |
|---|---|---|
| New domain warm-up | Day 1 ≤150 … Day 7 ≤2,000 (Resend new-domain table) | https://resend.com/docs/knowledge-base/warming-up |
| Existing domain → Resend | Day 1 ≤1,000/h100 … Day 7 ≤10,000/h2000 | same |
| Bounce during warm-up | Keep **&lt;4%**; slow plan if rising | Resend warm-up + quotas |
| Spam during warm-up | Keep **&lt;0.08%** (Resend); aim **&lt;0.1%** Gmail Postmaster | Resend + Google |
| Workspace mailbox (Phase 1) | Soft cap far below 2,000 msg/day (e.g. 20–50 cold/day/mailbox) | Google Workspace sending limits |
| Product daily default (roadmap) | 20–30/day until reputation proven | `04-master-roadmap.md` |

**Must ship:** per-workspace + per-domain + per-mailbox daily/hourly caps; queue with backoff; UI that shows remaining quota.

### 1.3 Unsubscribe (one-click + body + honor SLA)

| Layer | Spec | Honor window |
|---|---|---|
| **RFC 8058 one-click** | Headers: `List-Unsubscribe: <https://…>` **and** `List-Unsubscribe-Post: List-Unsubscribe=One-Click`. HTTPS POST only (no preference-page redirect for the one-click endpoint). Required for Gmail/Yahoo **marketing/subscribed** bulk mail. | Gmail FAQ: fulfill within **48 hours** (recommend); Yahoo: within **2 days** |
| **Visible body link** | Gmail bulk marketing: clearly visible unsub in body (can go to preferences *in addition to* headers). | Same |
| **CAN-SPAM (US commercial)** | Clear opt-out; process opt-outs for ≥30 days after send; honor within **10 business days**; no fee / no extra PII beyond email; postal address required; identify as ad; truthful From/subject. **Applies to B2B** — no B2B exception. | https://www.ftc.gov/business-guidance/resources/can-spam-act-compliance-guide-business |
| **Resend AUP** | Frictionless opt-out; honor within **7 days** (AUP mailing-list rules). | https://resend.com/legal/acceptable-use |

**Product must:** generate headers on every commercial/outreach touch; one-click endpoint writes suppression **immediately** (async OK for ESP, but DB suppress before next send); preferences page optional secondary.

### 1.4 Complaint pause

| Signal | Threshold (research) | Product action |
|---|---|---|
| Gmail user-reported spam (Postmaster) | Keep **&lt;0.10%**; never reach **0.30%**. ≥0.3% → mitigation ineligible until &lt;0.3% for **7 consecutive days**. | Soft warn at 0.05%; hard pause workspace/domain send at 0.1% or rising trend |
| Yahoo | Keep spam rate **&lt;0.3%** (Yahoo: rate based on mail delivered to inbox). | Same pause model |
| Resend spam/complaint | **&lt;0.08%** or account may pause/shut down | Map webhook `complained` → suppress + pause |
| Microsoft | Hygiene: complaints matter; monitor via SNDS/JMRP (**uncertain** exact public numeric threshold in Microsoft’s May 2025 auth blog — third-party sources often cite 0.3%/0.1%; verify via SNDS). | Pause on complaint webhooks + SNDS red if available |

**Citations:** Google FAQ spam-rate section; Yahoo Sender Hub; Resend account quotas https://resend.com/docs/knowledge-base/account-quotas-and-limits

### 1.5 Suppression lists (hard gate on Approve/Send)

Must suppress before any send path (Resend *and* Gmail):

1. **Hard bounce** addresses (permanent failure) — remove immediately  
2. **Complaint / spam report** addresses  
3. **Unsubscribe / opt-out / objection** (email-level + workspace-level)  
4. **Manual do-not-contact** (user or admin)  
5. **Role / trap heuristics** (optional MVP+): `abuse@`, `postmaster@`, known spamtrap patterns — **uncertain** exact lists; at least never re-add suppressed  
6. **Workspace global suppress** shared across Issues/Accounts  

**Google guidance:** don’t buy lists; don’t mail people who didn’t sign up; auto-unsub multi-bouncers.  
**Resend:** only send to opted-in; capture bounces/complaints via webhooks.

### 1.6 Content / identity hygiene (ship with compose)

- Accurate From display name (no “Re:” / “URGENT” spoof patterns — Gmail display-name guidelines)  
- Physical postal address in footer for US commercial (CAN-SPAM)  
- Clear sender identity; no `no-reply@` (Resend tip)  
- Plain-text + HTML; keep under ~102KB to avoid Gmail clip (Resend tip)  
- Links/images prefer same organizational domain as From (Resend)  
- Open tracking **off by default** for cold/transactional-adjacent (roadmap + Resend)  

### 1.7 Pre-public-send checklist (product gate UI)

```text
☐ Domain verified (SPF + DKIM pass on test message)
☐ DMARC published (p=none minimum) + From alignment OK
☐ Warm-up plan selected; daily/hourly caps configured
☐ List-Unsubscribe + List-Unsubscribe-Post wired; test POST
☐ Body unsub + postal address (US commercial) present in template
☐ Webhooks: bounce / complaint / delivered → suppress + metrics
☐ Complaint/bounce auto-pause thresholds configured
☐ Suppression checked on Approve
☐ HITL: no agent auto-send
☐ Lawful-basis / region disclosure completed for workspace
☐ ESP ToS acknowledged (see §2 Resend cold-outreach ban)
```

---

## 2. Resend-specific quality gates → UX warnings

### 2.1 Hard product risk: Resend prohibits cold outreach

**Primary source (2026-08-27 AUP):** https://resend.com/legal/acceptable-use.md

> **Spamming:** “You are prohibited from sending unsolicited messages of any kind, **including cold outreach**, purchased lists, or scraped contact data.”  
> **Mailing lists:** “All mail must be sent to recipients who have **explicitly opted in**…”

**Also:** ToS requires consents under CAN-SPAM, GDPR, etc. Consent KB: https://resend.com/docs/knowledge-base/what-counts-as-email-consent

| Implication | Product response (research note) |
|---|---|
| Phase 0 Resend path is **not** a compliant cold-email ESP under their AUP | Market as: (a) opted-in / warm / transactional-adjacent only on Resend, **or** (b) accelerate **Gmail OAuth send** for true cold, **or** (c) evaluate cold-friendly ESP later |
| UX must not imply “blast cold lists via Resend” | Onboarding copy: “Resend is for permissioned mail. Cold B2B send uses connected mailbox (Phase 1).” |
| Account termination risk | Detect high complaint/bounce; pause; show “ESP risk” |

### 2.2 Numeric gates (map 1:1 to UX)

| Metric | Resend gate | Suggested UX |
|---|---|---|
| **Bounce rate** | Must stay **&lt;4%**; above may **temporary pause** | Green &lt;1%; amber 1–3%; red ≥3% “Sending paused soon”; hard stop ≥4% |
| **Spam / complaint rate** | Must stay **&lt;0.08%**; above may pause/shutdown | Green &lt;0.03%; amber 0.03–0.06%; red ≥0.06%; hard stop ≥0.08% |
| **Warm-up spam target** | **&lt;0.08%** while ramping | Same meters on Accounts health |
| **Rate limit** | Default **10 req/s per team** (shared across API keys); `429` | Client throttle; batch API (≤100) counts as 1 request |
| **Free quota** | 100/day, 3,000/month (sent+received) | Block schedule when remaining &lt; N |
| **Paid overage** | Hard cap **5×** monthly quota then pause | Warn at 80% / 100% |
| **Data retention** | 30 days email content/events | Store webhooks in own DB for suppress history |
| **Test sends** | Don’t hard-bounce fake addresses | Use Resend test addresses only |

Sources: https://resend.com/docs/knowledge-base/account-quotas-and-limits · https://resend.com/docs/knowledge-base/warming-up · https://resend.com/blog/top-10-email-deliverability-tips

### 2.3 UX warning copy (draft — product, not legal)

| Trigger | Warning (EN) | Warning (KO sketch) |
|---|---|---|
| Domain unverified | “Verify SPF/DKIM before sending. Unauthenticated mail is rejected by Gmail/Outlook.” | “발송 전 SPF/DKIM을 확인하세요. 미인증 메일은 Gmail/Outlook에서 거부될 수 있습니다.” |
| No DMARC | “Add DMARC (p=none) now. Required for bulk senders and Microsoft high-volume domains.” | “DMARC(p=none)를 추가하세요. 대량 발송·Microsoft 고볼륨에 필요합니다.” |
| Bounce ≥3% | “Bounce rate is high. Pause, remove invalids, slow warm-up. Resend may pause above 4%.” | “반송률이 높습니다. 정리 후 워밍업을 늦추세요. 4% 초과 시 Resend가 중단할 수 있습니다.” |
| Complaint ≥0.05% | “Spam complaints rising. Pause campaigns and review targeting. Resend limit is 0.08%.” | “스팸 신고가 증가 중입니다. 타겟을 점검하세요. Resend 한도는 0.08%입니다.” |
| Cold list + Resend | “Resend’s Acceptable Use Policy prohibits cold outreach. Use a connected mailbox or only email opted-in contacts.” | “Resend 이용약관은 콜드 아웃리치를 금지합니다. 연결된 메일함 또는 수신동의 연락처만 사용하세요.” |
| Open tracking on | “Open tracking can hurt deliverability. Recommended off for first-touch outreach.” | “오픈 추적은 전달률에 불리할 수 있습니다. 첫 접촉은 끄는 것을 권장합니다.” |
| Sudden volume spike | “Volume jump detected vs last 7 days. Ramp gradually (Resend warm-up).” | “최근 7일 대비 발송량이 급증했습니다. 워밍업 일정에 맞춰 올리세요.” |
| Cap hit | “Daily cap reached (workspace policy). Remaining sends queued for tomorrow.” | “일일 한도에 도달했습니다. 나머지는 내일 대기열에 넣습니다.” |

### 2.4 Webhook → product mapping

| Resend event | Action |
|---|---|
| `bounced` (hard) | Suppress address; increment bounce meter; if rate high → pause |
| `complained` | Suppress; complaint meter; pause Issue + alert |
| `delivered` | Reputation OK signal |
| `opened` / `clicked` | Optional analytics; **do not** auto-trigger next email (§6) |
| Inbound / reply | Stop sequence for that person; move Issue → Replied |

---

## 3. Gmail / Workspace limits — UX copy

### 3.1 Provider bulk rules (personal Gmail / Yahoo / Outlook.com)

Use in Accounts → “Inbox provider rules” help panel:

**EN (short):**  
“If your domain sends ~5,000+ messages/day to personal Gmail, Yahoo, or Outlook.com, you must use SPF **and** DKIM, publish DMARC (at least p=none), keep spam complaints under 0.3% (aim under 0.1%), and support one-click unsubscribe on marketing mail. Gmail bulk status is permanent once earned. Non-compliance can mean delays, junk, or SMTP rejection (e.g. Microsoft 550 5.7.515).”

**KO (short):**  
“도메인이 개인 Gmail·Yahoo·Outlook.com으로 하루 약 5,000통 이상 보내면 SPF+DKIM, DMARC(최소 p=none), 스팸 신고율 0.3% 미만(목표는 0.1% 미만), 마케팅 메일 원클릭 수신거부가 필요합니다. Gmail 대량 발신자 분류는 한 번 적용되면 유지됩니다. 미준수 시 지연·스팸함·SMTP 거부가 발생할 수 있습니다.”

### 3.2 Google Workspace *sending* limits (Phase 1 Gmail OAuth)

Primary: https://support.google.com/a/answer/166852

| Limit type | Paid Workspace (typical) | Trial |
|---|---|---|
| Messages / user / day | **2,000** (mail merge 1,500) | **500** |
| Recipients / message (web) | 2,000 total (max **500 external**) | — |
| SMTP POP/IMAP recipients / message | **100** | — |
| Gmail API recipients / message | **500** | — |
| Total recipients / day | **10,000** (merge 1,500) | — |
| External recipients / day | **3,000** | — |
| Unique recipients / day | **3,000** (of which **2,000 external**; trial **500 external**) | — |
| Rolling window | **24 hours** (not calendar day) | same |
| On exceed | Can’t send up to **~24h**; account may suspend | same |

**Notes for UX**

- Limits **can change without notice** (Google).  
- Alternate addresses, delegates, vacation auto-replies **count**.  
- SMTP relay has **different** limits — link out if used.  
- Free consumer Gmail is tighter (~500/day web) — **do not** encourage consumer Gmail for outreach product.

**Suggested soft product caps (not Google’s hard limit):**  
New mailbox: 20–30 external/day → ramp to 50–100 only with clean metrics. Hard Google limit is a *ceiling*, not a *target*.

**UX copy when approaching Workspace limit:**

> “This Google mailbox can send at most 2,000 messages per rolling 24 hours (500 on trial). You’re at {n}/2,000. Exceeding Google’s limit blocks sending for up to 24 hours.”

> “이 Google 메일함은 롤링 24시간 기준 최대 2,000통(체험 500통)입니다. 현재 {n}/2,000. 한도 초과 시 최대 24시간 발송이 막힐 수 있습니다.”

### 3.3 Postmaster Tools (recommend in product)

- Register domain: Google Postmaster Tools (spam rate, auth, compliance).  
- Microsoft SNDS + JMRP for Outlook.com complaint feedback.  
- Yahoo CFL after DKIM.

---

## 4. Legitimate interest vs consent — EU / CA / KR UX notes

**Disclaimer:** Research notes summarizing public guidance. **Not legal advice.** EU ePrivacy is **nationally implemented** — Germany/Italy/Spain often stricter than UK/IE/FR for B2B email. Marked **uncertain** where member-state variance dominates.

### 4.1 Comparison matrix (high level)

| Region | Typical B2B cold email posture (research summary) | UX implication |
|---|---|---|
| **US** | **CAN-SPAM**: opt-**out** regime for commercial email; **no prior consent required**; truthful headers; ad identification; postal address; honor opt-out ≤10 business days. Applies to **B2B**. | Default: “US commercial — CAN-SPAM fields required.” Collect postal address in workspace settings. |
| **UK** | **PECR**: electronic mail marketing rules **do not** apply to **corporate subscribers**; still need identity + opt-out address. **Sole traders / some partnerships** = individual subscribers → consent or soft opt-in. **UK GDPR** still applies to personal data (named people) → usually **legitimate interests** + transparency + absolute right to object. | Ask: corporate vs sole trader (**uncertain** auto-detect). Show LI notice + unsub. Soft opt-in only if sale/negotiation criteria met. |
| **EU** | **GDPR** Art. 6(1)(f) legitimate interest often argued for B2B + Recital 47 (direct marketing example) **if** LIA documented. **ePrivacy Art. 13** consent default for natural persons; **member-state B2B exemptions vary** (**uncertain** — DE/IT/ES often consent-heavy). | Per-country policy flag later; MVP: force stronger consent mode for DE/AT/IT/ES or “counsel required.” Always: identity, purpose, opt-out, privacy info ≤1 month if data from third party. |
| **Canada (CASL)** | Generally **consent-based** (express or implied) for CEMs; identification + unsubscribe; honor unsub ~**10 days**; consent records. B2B **not** a free-for-all; exemptions/implied consent are **narrow** (existing business relationship, conspicuous publication + relevance, limited B2B exemption). **uncertain** without counsel for scraping LinkedIn etc. | Default CA mode: require recorded consent basis; store evidence; unsub ≤10 days. |
| **Korea (KR)** | **정보통신망법 §50**: 영리목적 광고성 정보 → **명시적 사전 동의** principle; limited exceptions (e.g. prior transaction same-type goods within presidential-decree window — often cited **6 months** in secondary guides). Must honor 수신거부; labeling **(광고)** etc. per enforcement rules; periodic consent re-check (commonly cited **every 2 years**). | If sending **to** persons in KR / from KR entity: prefer **consent mode** + `(광고)` templates. Cold B2B to KR emails is **high risk** without counsel. |

**Primary / strong secondary cites**

- FTC CAN-SPAM: https://www.ftc.gov/business-guidance/resources/can-spam-act-compliance-guide-business  
- ICO B2B marketing (UK PECR/UK GDPR): https://ico.org.uk/for-organisations/direct-marketing-and-privacy-and-electronic-communications/business-to-business-marketing/  
- GDPR legitimate interest concept: EUR-Lex GDPR Art. 6(1)(f), Recital 47 (**counsel for LIA**)  
- CASL overview (secondary; verify CRTC): industry summaries + Torys note on B2B exemption limits — treat as **uncertain** until counsel  
- KR: 정보통신망법 제50조 (law.go.kr); EasyLaw / KISA spam guidance summaries for labeling & 2-year reconfirm  

### 4.2 Product UX patterns (lawful basis picker)

On workspace or Issue audience settings:

```text
Sending region focus: [ US ] [ UK ] [ EU ] [ CA ] [ KR ] [ Mixed ]
Lawful basis for this Issue:
  ( ) Consent — I have documented opt-in
  ( ) Legitimate interest — B2B professional outreach (UK/EU where allowed)
  ( ) CAN-SPAM commercial — US opt-out compliance only
  ( ) CASL — express/implied consent on file
```

**Always show (first touch footer / privacy blurb):**

- Who you are + why writing (role-relevant)  
- How you got the address (source field on Contact)  
- Link to privacy notice  
- One-click / easy opt-out  
- For US: physical postal address + commercial identification as required  

**LI mode extras (UK/EU research note):**

- Store **Legitimate Interest Assessment** checklist (purpose / necessity / balancing) as workspace doc — product can provide a template, **not** certify compliance  
- Cap frequency; relevance to recipient’s job  
- Honor objection immediately → suppression  
- Prefer role-based relevance over spray purchased lists (purchased lists also fail Google + Resend rules)

**Consent mode extras:**

- Unchecked opt-in; purpose-specific; easy withdraw  
- Resend’s definition of consent aligns with GDPR-style active opt-in (their KB)

**KR mode extras (research):**

- `(광고)` prefix where required for 광고성 정보  
- Sender name, contact, easy 수신거부  
- Do not make unsub require login  
- Re-consent reminder job every ~2 years if storing KR consents (**confirm** decree details with counsel)

### 4.3 Uncertain — escalate to counsel before scaling

- Whether a specific EU member state allows LI-only B2B email to named professionals  
- CASL “conspicuous publication” / B2B exemption for scraped public emails  
- Whether Korean 정보통신망법 treats outbound from KR company to foreign B2B emails differently from domestic  
- Interaction of open/click tracking pixels with ePrivacy/cookie rules (ICO notes PECR cookie rules can apply to pixels)

---

## 5. Checklist — Korean founders sending to US / EU B2B

Use as founder ops checklist (product can render as Settings → Compliance).

### 5.1 Company / ops

- [ ] Decide primary send path: **permissioned via Resend** vs **cold via Gmail/Workspace** (Resend AUP blocks cold)  
- [ ] Register sending domain aged + website on same org domain  
- [ ] Separate subdomains: e.g. `mail.` outreach vs `notify.` transactional (Resend recommendation)  
- [ ] Publish SPF, DKIM, DMARC (`p=none` → plan `quarantine`)  
- [ ] Google Postmaster Tools + (if Outlook volume) SNDS/JMRP  
- [ ] Collect **US postal address** for CAN-SPAM footers  
- [ ] Privacy policy (KO/EN) describing outreach processing, sources, retention, rights  
- [ ] Document LIAs if claiming legitimate interest (UK/EU)  
- [ ] Retention: suppressions **kept** even when profiles deleted (do-not-contact)  
- [ ] No purchased/scraped lists; prefer first-party or compliant vendors with provenance  

### 5.2 Before first US send

- [ ] CAN-SPAM: accurate From; non-deceptive subject; ad identification; postal address; working opt-out; honor ≤10 business days  
- [ ] One-click headers still recommended (Gmail/Yahoo) even if under 5k/day  
- [ ] Soft daily caps; warm-up if new domain  

### 5.3 Before first UK / EU send

- [ ] Classify recipients: corporate vs sole trader / personal mailbox (**uncertain** automation)  
- [ ] UK: PECR corporate subscriber path + UK GDPR transparency + objection  
- [ ] EU: check target country ePrivacy; default to consent for DE/AT/IT/ES until counsel says otherwise (**uncertain**)  
- [ ] Provide privacy info at collection or within 1 month if obtained indirectly (GDPR transparency — ICO/EU practice)  
- [ ] Honor opt-out/objection immediately  

### 5.4 Canada (if any CA recipients)

- [ ] Record express or valid implied consent basis + date  
- [ ] CEM identification + unsub; honor ~10 days  
- [ ] Do not rely on “B2B = exempt” without counsel  

### 5.5 Korea-origin / KR recipients

- [ ] If 광고성 정보 to KR: prior consent + labeling + 수신거부 mechanics per 정보통신망법 §50  
- [ ] Don’t assume US/EU LI theory covers KR recipients  
- [ ] PIPA (개인정보보호법) may also apply to personal data processing — **uncertain** scope note; counsel  

### 5.6 Product feature checklist (founder → eng)

- [ ] DNS wizard gate  
- [ ] Caps + warm-up  
- [ ] Unsub + suppression  
- [ ] Complaint/bounce pause  
- [ ] HITL approve  
- [ ] Region / lawful-basis fields on workspace  
- [ ] Audit log: who approved send, template version, recipient source  

---

## 6. What NOT to automate

| Anti-pattern | Why | Product rule |
|---|---|---|
| **Agent sends without HITL** | Abuse, ToS, irreversible reputation/legal harm; roadmap already “draft-only until Approve” | MCP/`schedule_send` requires human confirm; no API key for agents to raw Resend |
| **Open-triggered sequences** (“if opened, send #2 in 1h”) | Opens are unreliable (Apple MPP, prefetch); looks like engagement bait; can spike complaints; Resend warns tracking hurts some mail | Sequences time-based or reply-based only; opens = analytics not triggers |
| **Auto-buy/enrich + instant send** | Purchased/scraped lists banned by Google guidelines + Resend AUP; high trap/complaint risk | Enrichment ≠ send; human review of list provenance |
| **Auto-restart after unsub/complaint** | Illegal/ToS; destroys reputation | Suppression is permanent unless explicit re-subscribe |
| **Hidden unsub / login-walled unsub** | Fails Gmail one-click spirit; KR law forbids obstructing 수신거부; CAN-SPAM limits friction | One-click endpoint works logged-out |
| **Fake engagement warm-up services** | Resend explicitly discourages artificial engagement warm-up | Ban in product docs; organic warm-up only |
| **Sudden 10× volume** | Greylisting / reputation crash (Google + Resend) | Enforce ramp % vs trailing 7-day volume |
| **Sending from root domain mixed traffic** | One spam incident burns corporate domain | Force subdomain for outreach |
| **Consumer @gmail.com as sending identity** | Spoofing/impersonation rules; weak limits | Workspace custom domain only |
| **Ignore Resend cold ban and “hope”** | Account shutdown without warning possible (AUP) | Explicit ESP mode switch in UI |

### 6.1 Safe automation (OK to ship)

- Draft generation, personalization suggestions  
- Schedule within caps after Approve  
- Stop-on-reply / bounce / complaint  
- Reminder to human when Waiting too long  
- DNS monitoring + health scores  
- Suppression sync from webhooks  

---

## 7. Source index (primary preferred)

| Topic | URL |
|---|---|
| Gmail sender guidelines | https://support.google.com/mail/answer/81126 |
| Gmail sender FAQ | https://support.google.com/mail/answer/14229414 |
| Google Workspace sending limits | https://support.google.com/a/answer/166852 |
| Microsoft high-volume senders | https://techcommunity.microsoft.com/blog/microsoftdefenderforoffice365blog/strengthening-email-ecosystem-outlook%e2%80%99s-new-requirements-for-high%e2%80%90volume-senders/4399730 |
| Microsoft 550 5.7.515 | https://support.microsoft.com/en-us/outlook/fix-ndr-error-550-5-7-515-in-outlook-com |
| Yahoo sender best practices | https://senders.yahooinc.com/best-practices/ |
| Resend warm-up | https://resend.com/docs/knowledge-base/warming-up |
| Resend quotas / bounce / spam | https://resend.com/docs/knowledge-base/account-quotas-and-limits |
| Resend AUP | https://resend.com/legal/acceptable-use |
| Resend consent KB | https://resend.com/docs/knowledge-base/what-counts-as-email-consent |
| Resend deliverability tips | https://resend.com/blog/top-10-email-deliverability-tips |
| FTC CAN-SPAM guide | https://www.ftc.gov/business-guidance/resources/can-spam-act-compliance-guide-business |
| ICO B2B marketing (UK) | https://ico.org.uk/for-organisations/direct-marketing-and-privacy-and-electronic-communications/business-to-business-marketing/ |
| KR 정보통신망법 §50 | https://www.law.go.kr (정보통신망 이용촉진 및 정보보호 등에 관한 법률 제50조) |

---

## 8. Open product decisions (for parent)

1. **ESP strategy:** Keep Resend for opted-in only and make Gmail the cold path — or switch Phase 0 ESP. AUP conflict is blocking for a “cold email” positioning.  
2. **Default region pack:** US-only MVP vs US+UK with EU consent-walled.  
3. **Complaint pause threshold:** Product-hard at Resend 0.08% vs softer Gmail 0.1% warn.  
4. **Whether to implement RFC 8058 for *all* outreach** (recommended yes) even under bulk threshold.

---

*End of research notes. Re-verify Google/Microsoft/Resend pages before launch; laws and enforcement change.*
