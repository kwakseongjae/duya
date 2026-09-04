# Cold Email Ops · Viral GTM · Pricing · Risks · Differentiation (2026)

**Document:** `01-market-viral-conventions.md`  
**Audience:** Product / GTM for an Issue-like purpose-clustering cold-email SaaS (indie founders + agencies)  
**Research date:** 2026-09-04 (KST)  
**Method:** WebSearch + WebFetch of operator playbooks, vendor pricing pages, Google/Microsoft primary docs. **No fabricated metrics** — vendor claims and secondary blogs are labeled. Where numbers conflict across sources, both are cited.

**Bilingual note / 이중언어 안내:**  
Each major section opens with a short **KO summary** then the full **EN deep dive**. Tables and checklists are language-neutral where possible.

---

## Contents

1. [Industry conventions for cold email ops (2025–2026)](#1-industry-conventions-for-cold-email-ops-2025–2026)
2. [How top products structure UX — and why Issue-board is novel](#2-how-top-products-structure-ux--and-why-issue-board-is-novel)
3. [Viral / GTM points for indie founders & agencies (X · Threads)](#3-viral--gtm-points-for-indie-founders--agencies-x--threads)
4. [Packaging & pricing conventions](#4-packaging--pricing-conventions)
5. [Risks: spam, ToS, Google/Microsoft bulk rules](#5-risks-spam-tos-googlemicrosoft-bulk-rules)
6. [Differentiation matrix vs Instantly / Smartlead / Lemlist](#6-differentiation-matrix-vs-instantly--smartlead--lemlist)
7. [Implications for our MVP](#7-implications-for-our-mvp)
8. [Source index](#8-source-index)

---

## 1. Industry conventions for cold email ops (2025–2026)

### KO summary
2026년 콜드이메일 운영의 핵심은 **카피보다 인프라**다. 전용 발송 도메인 팬(fan), 메일박스당 일일 캡(대체로 25–40통), 3–4주 워밍업, SPF/DKIM/DMARC, 리스트 검증(바운스 &lt;2%), Unibox로 회신 분류, 시퀀스 4–7터치가 업계 관례다. Google/Yahoo(2024)와 Microsoft(고볼륨, ~5,000/일) 벌크 규칙이 바닥선을 정한다.

### EN deep dive

### 1.1 Mental model: three “limits” operators confuse

Operator writing in 2026 distinguishes ([LeadHaste](https://leadhaste.com/blog/cold-email-sending-limits-2026)):

| Limit type | What it is | Rough 2026 cold-email reality |
|------------|------------|-------------------------------|
| **Platform ceiling** | Google Workspace / M365 hard caps | Workspace often cited ~2,000/user/day; M365 much higher — **not** the cold-email safe zone |
| **Deliverability ceiling** | Volume where primary placement collapses | Common operator band: **~25–40 cold sends per warmed inbox / day** |
| **Engagement / complaint ceiling** | Quality signal, not a fixed count | Spam rate ideally **&lt;0.1%**, never **≥0.3%** (Google Postmaster) |

Hitting the platform ceiling while ignoring deliverability/engagement is how teams “torch” domains.

### 1.2 Warmup ramps (what “good” looks like in 2026)

Consensus across operator playbooks ([Yalc warmup](https://www.yalc.ai/blog/cold-email-warmup/), [Yalc deliverability](https://www.yalc.ai/blog/cold-email-deliverability/), [LeadHaste](https://leadhaste.com/blog/cold-email-sending-limits-2026), [Unify](https://www.unifygtm.com/explore/cold-email-2026-domain-setup-deliverability-sequences), Instantly sequence workflow):

| Phase | Typical daily volume per mailbox | Focus |
|-------|----------------------------------|--------|
| Week 1 | ~3–10 (warmup only) | Auth 100%; no real cold or near-zero cold |
| Week 2 | ~10–20 | Warmup dominant; Postmaster domain reputation appears |
| Week 3 | ~20–30 | Introduce tiny real cold (≤5/day) if bounce &lt;2% |
| Week 4 | ~25–40 total | Split warmup + cold; seed test ≥80% primary before scaling |
| Steady state | Cold ~20–30 (some guides allow up to ~40); maintenance warmup ~5–10 or ~10% of volume | Continuous light warmup; don’t spike |

**Shape matters more than absolute numbers:** ~40% week-over-week is widely described as the safe band; tripling volume week-to-week is called out as domain-killing ([Yalc warmup](https://www.yalc.ai/blog/cold-email-warmup/)).

**Vendor-cited placement gap (treat as vendor/protocol claim, not census):** InboxKit protocol data via Yalc: full warmup ~**88%** inbox placement vs ~**54%** when skipped/abbreviated.

**Duration:** 2–4 weeks common; some operators say 4–6 weeks for brand-new domains.

### 1.3 Daily caps & mailbox math

| Source | Stated per-inbox cold cap (warmed) |
|--------|-------------------------------------|
| LeadHaste table | Google/M365 warmed **25–30**/day |
| Yalc warmup | **~20–30**/day “operator ceiling” |
| Yalc deliverability | **~30–40**/day before placement tilts |
| Instantly campaign guidance | Max **~30** campaign emails/inbox/day |
| Unify managed default | **25**/day default, configurable up to **65** when engagement supports |

**Scaling formula used by agencies:**  
`mailboxes_needed ≈ daily_cold_target ÷ 25` (conservative)  
Example: 500/day → ~20 mailboxes; 2,000/day → ~80 ([Yalc](https://www.yalc.ai/blog/cold-email-warmup/), [LeadHaste](https://leadhaste.com/blog/cold-email-sending-limits-2026)).

### 1.4 Domain rotation / “domain fan”

Industry convention (near-universal among serious operators):

1. **Never** send cold from the primary brand domain (customer/support risk).
2. Register **lookalike / cousin domains** (`getbrand.com`, `trybrand.io`, etc.).
3. Typical small setup: **2–10 send domains**, **2–8 mailboxes per domain**.
4. Rotate volume across the pool (round-robin or **reputation-weighted** rotation — Smartlead/Instantly both market weighted/reputation-aware rotation).
5. If one domain’s seed placement drops (e.g. &lt;80% primary), **pause that domain**, run warmup-only recovery, re-ramp — don’t push harder ([Yalc deliverability](https://www.yalc.ai/blog/cold-email-deliverability/)).

Prefer **real Workspace / M365 mailboxes** for relational cold reputation; custom SMTP (SendGrid/SES-class) is widely described as better for transactional than cold relational sends ([Yalc warmup](https://www.yalc.ai/blog/cold-email-warmup/)).

### 1.5 Sequence length & spacing

From Instantly’s own sequence workflow + 2026 benchmark narrative ([Instantly sequence guide](https://instantly.ai/blog/email-sequence-workflow-step-by-step-process-for-building-deploying-sequences/), [Instantly 2026 benchmark](https://instantly.ai/cold-email-benchmark-report-2026)):

| Convention | Typical value |
|------------|---------------|
| Touch count | **4–7** emails |
| Spacing | **2–4 business days** |
| Window | ~**10–15 days** |
| Reply timing | Instantly reports **~58%** of replies on Step 1; **~42%** from follow-ups |
| First-touch length | Often **&lt;80 words** (benchmark guidance) |
| Avg reply rate (Instantly platform sample) | **3.43%**; top quartile **≥5.5%**; elite **≥10.7%** |

Content norms in deliverability playbooks: plain-text bias, ≤1 link (often 0 on email #1), open tracking **off** for cold, rotate subject variants ([Yalc deliverability](https://www.yalc.ai/blog/cold-email-deliverability/)). Signal-led openers (funding, hiring, tool switch) dominate 2026 copy advice ([Flowjam](https://www.flowjam.com/blog/cold-email-templates-for-b2b-saas-that-book-30-more-demos), [Sendspark](https://blog.sendspark.com/cold-email-best-practices)).

### 1.6 Reply handling & Unibox / Master Inbox

**What Unibox means in this category:** one UI that aggregates replies from all connected sending accounts so reps don’t tab through 10–50 Gmail tabs ([SalesTarget explainer](https://salestarget.ai/articles/what-is-a-unified-inbox-for-cold-email), Instantly Unibox, Smartlead Master Inbox).

**Expected reply taxonomy (table stakes):**

| Category | Typical automation |
|----------|-------------------|
| Interested / meeting | Stop sequence; surface to top; assign human / book |
| Not interested | Stop; suppress |
| Unsubscribe | Immediate global suppress |
| OOO | Pause until return date; resume |
| Bounce / auto-reply | Classify; don’t treat as positive intent |
| Wrong person | Suppress or route to alternate contact |

Critical product rule: **reply must pause the sequence in the same system** — disconnected inbox + sequencer causes “yes let’s talk” + Step 4 follow-up disasters ([SalesTarget](https://salestarget.ai/articles/what-is-a-unified-inbox-for-cold-email)).

Instantly Hypergrowth+ markets AI reply agent + booking; Smartlead markets NLP auto-categorize + Master Inbox; Lemlist leans multichannel inbox + more manual triage ([Growth Engineer comparison](https://growthengineer.ai/blog/instantly-vs-smartlead-vs-lemlist)).

### 1.7 List hygiene

Operator + Instantly guidance ([Instantly sequence](https://instantly.ai/blog/email-sequence-workflow-step-by-step-process-for-building-deploying-sequences/), [LeadHaste](https://leadhaste.com/blog/cold-email-sending-limits-2026)):

| Control | Common target |
|---------|----------------|
| Pre-send verification | NeverBounce / ZeroBounce / MillionVerifier-class |
| Bounce rate | **&lt;2%** widely cited; **&lt;1%** “healthy” target |
| Pause trigger | Often **&gt;5%** bounce → pause & scrub |
| Catch-alls / role addresses | High risk; many teams skip or down-rank |
| Exclusion sync | Customers, open opps, prior opt-outs before launch |
| List refresh | Periodic re-verify; don’t recycle dead lists |

### 1.8 SPF / DKIM / DMARC / BIMI

#### Authentication stack (required floor)

**Google Email sender guidelines** (primary): https://support.google.com/a/answer/81126

| Audience | Requirement (summary) |
|----------|------------------------|
| All senders to personal Gmail (from Feb 2024) | SPF **or** DKIM; valid PTR; TLS; spam rate &lt;0.3%; RFC 5322 |
| Bulk (≥5,000/day to personal Gmail) | SPF **and** DKIM; **DMARC** (min `p=none`); From alignment; one-click unsubscribe for marketing/subscribed mail |

Google also recommends keeping spam rates **below 0.1%** and never reaching **0.3%** ([FAQ](https://support.google.com/a/answer/14229414)). Google does **not** track open rates and cannot verify third-party open accuracy (same guidelines page) — product KPI implication: prefer replies.

**Microsoft high-volume to consumer Outlook** (primary NDR doc): https://support.microsoft.com/en-us/outlook/fix-ndr-error-550-5-7-515-in-outlook-com  
Threshold: **≥5,000** messages/day to Microsoft consumer mail with same From domain → must pass SPF **and** DKIM, publish DMARC, pass DMARC alignment — else **`550 5.7.515`**.

**Operator DMARC path (convention):**

1. Start `v=DMARC1; p=none; rua=mailto:…`
2. After clean reports → `p=quarantine`
3. Mature programs → `p=reject`
4. Prefer **2048-bit** DKIM (Workspace often defaults 1024; operators urge upgrade) ([Yalc warmup](https://www.yalc.ai/blog/cold-email-warmup/))

#### BIMI (optional brand layer — not cold-email table stakes)

Google Workspace BIMI help: https://support.google.com/a/answer/10911320  

BIMI needs:

- DMARC at **`p=quarantine` or `p=reject`** with **`pct=100`** (BIMI does **not** work with `p=none`)
- SVG Tiny PS logo + HTTPS hosting
- For Gmail: **VMC** (trademark → blue check) or **CMC** (logo without check, with usage history)
- BIMI DNS TXT at `default._bimi.…`

**Product implication:** BIMI is a **later brand/trust upsell** for primary or carefully enforced send domains — not MVP for cold lookalike domains that may be burned and replaced. Cold lookalikes rarely justify VMC cost/process.

### 1.9 Monitoring loop (convention)

| Cadence | Check |
|---------|--------|
| Daily (active campaigns) | Seed/inbox placement; bounce; spam rate in Postmaster |
| 2×/week early | Postmaster Tools + Microsoft SNDS |
| Weekly | Reply rate; DMARC aggregate; pause domains &lt;~80% primary placement |
| Always | Honor unsubscribes fast (Google: within **48 hours** for bulk marketing paths) |

---

## 2. How top products structure UX — and why Issue-board is novel

### KO summary
인컴번트 UX의 1급 객체는 **Campaign / Sequence / Cadence**다. 회신은 Unibox, 인프라는 Accounts/Warmup. “목적(purpose)을 Linear/GitHub Issue처럼 보드로 모은다”는 메타포는 조사 범위에서 **거의 없다** — 이게 우리 제품의 개념적 공백(whitespace)이다.

### EN deep dive

### 2.1 Incumbent object model (shared industry grammar)

| Product family | Primary object | Secondary | Reply surface | Infra surface |
|----------------|----------------|-----------|---------------|---------------|
| **Instantly** | Campaign containing Sequence steps | Leads / opportunities (CRM-lite) | Unibox | Email accounts + warmup + placement |
| **Smartlead** | Campaign + subsequences | Client/sub-accounts (agency) | Master Inbox | Unlimited accounts + SmartSenders/DNS helpers |
| **Lemlist** | Campaign / multichannel sequence | Creative assets (image/video) | Multichannel inbox | Lemwarm + limited senders per seat |
| **Enterprise SE** (Outreach, Salesloft) | **Cadence** | Opportunity / Salesforce objects | Team inbox patterns | Admin-heavy |

**Campaign vs Sequence vs Cadence (how operators use the words):**

- **Campaign** — the container: list + settings + analytics + which mailboxes send.
- **Sequence** — ordered email (or multichannel) steps with delays/conditions (“stop on reply”).
- **Cadence** — enterprise SE term for a multi-step, often multichannel playbook assigned to reps (Outreach/Salesloft language).

Instantly’s mental model is explicitly: import list → build sequence steps → connect inboxes → launch campaign; Unibox handles replies ([Instantly sequence workflow](https://instantly.ai/blog/email-sequence-workflow-step-by-step-process-for-building-deploying-sequences/), [LGM comparison UX notes](https://lagrowthmachine.com/instantly-vs-smartlead-vs-lgm/)).

Smartlead adds agency-shaped **client workspaces**, weighted rotation, API/webhooks, Master Inbox triage ([Smartlead site](https://www.smartlead.ai/), Growth Engineer).

Lemlist organizes around **creative personalization + multichannel steps** (email + LinkedIn + call), with per-seat sender pools ([Growth Engineer](https://growthengineer.ai/blog/instantly-vs-smartlead-vs-lemlist)).

### 2.2 What users actually stare at daily

1. **Campaign list** sorted by status/volume  
2. **Unibox** filtered to “Interested”  
3. **Account health / warmup scores**  
4. **Analytics** (reply, bounce — opens de-emphasized by sophisticated teams)

Missing from almost all: a **purpose-first board** that answers “what initiatives are live?” the way Linear answers “what work is in flight?”

### 2.3 Why Issue-board metaphor is novel

Our IA (see `issue-board-ia.md`) treats:

- **Issue** = purpose cluster (“Series A founders – partner offer”, “Agency webinar invite”)  
- **Touches** = sends/reminders inside the issue  
- **Sequence** = automation rules *owned by* the issue, not the other way around  
- Board columns: Draft → Ready → Sending → Waiting → Replied → Snoozed → Closed  

**Why this is different from Campaign folders/labels:**

| Dimension | Campaign UX | Issue-board UX |
|-----------|-------------|----------------|
| Noun users think in | Blast / funnel | Purpose / initiative / ticket |
| Lifecycle | Launch → finish campaign | Long-lived purpose with status changes |
| Cross-list memory | Weak across campaigns | Issue + Memory index designed for “what did we already say?” |
| Agent/MCP fit | Campaign IDs | Issue IDs map cleanly to agent tasks (`get_issue`, `list_open_purposes`) |
| Indie founder cognition | Spreadsheet of campaigns | Linear/GitHub habits already installed |

**Competitive whitespace (from prior market memo + this pass):** No major Instantly/Smartlead/Lemlist/Apollo/Outreach surface was found that presents outreach as **GitHub/Linear-style issues**. Closest metaphors are folders, tags, pipelines, or opportunities — still campaign-centric or CRM-centric.

**Risk of novelty:** Metaphor must still ship **commodity sequencer reliability** underneath, or users bounce to Instantly for “just send.” Novelty is the **organizing layer**, not a substitute for warmup/caps/Unibox.

---

## 3. Viral / GTM points for indie founders & agencies (X · Threads)

### KO summary
2026 인디/에이전시 GTM은 **시그널 기반 아웃바운드 + 빌드인퍼블릭 + MCP/오픈소스 드롭 + before/after 회신율**이 먹힌다. “언리미티드 메일박스”만으로는 차별화가 약하다. Issue 보드 데모, Memory(“이 사람한테 뭐 보냈지?”), Cursor/MCP 훅이 X·Threads 훅으로 강하다.

### EN deep dive

### 3.1 Audience & channel reality

- **X (Twitter):** agency operators, Clay+sequencer stack screenshots, deliverability threads, “I added 40 mailboxes” flexes, founder-led GTM.
- **Threads / LinkedIn-adjacent indie:** softer build-in-public, template drops, peer-to-peer “I emailed 80 founders” stories.
- Buyers are **tool-fatigued** on Instantly vs Smartlead debates; they engage on **new metaphors, agent workflows, honest deliverability, open tools**.

### 3.2 Hooks that work in 2026 (patterns, not guaranteed virality)

| Hook type | Why it lands | Example angle for *this* product |
|-----------|--------------|----------------------------------|
| **Metaphor demos** | Pattern interrupt vs campaign grids | “Linear for cold email” — drag Issue from Draft → Sending |
| **Before/after reply rates** | Concrete social proof; Instantly benchmark frames the baseline (~3.43% avg) | “Same list, purpose-clustered issues → X% positive replies” (**only with real numbers**) |
| **Infra transparency** | Trust in a spam-sensitive category | Show DNS checklist, caps, Postmaster screenshot (redacted) |
| **AI-slop contrast** | Filters + humans detect generic AI | “Signal opener + Issue purpose, not GPT spray” |
| **MCP / Cursor native** | Technical founders live in agents | “Ask Claude: what open Issues mention Acme?” |
| **Open-source drops** | GitHub stars → distribution | Local CLI/MCP that drafts into Issues (see coldforge/viralman pattern) |
| **Agency client board** | Agencies buy multi-tenant clarity | “One board per client purpose, not 14 campaign names” |
| **Memory / anti-duplicate** | Real pain across Clay+Instantly stacks | “Never pitch the same person twice across Issues” |
| **Dual-send story** | Gmail warmth → Resend scale | “Start on Gmail plugin; graduate volume without losing ledger” |
| **Template swipe files** | High save/share | 4–7 step purpose templates with variables |

### 3.3 Demo shapes that travel

1. **60-second Loom:** create Issue → attach 12 prospects → preview → approve send → reply lands in Inbox tagged Interested.  
2. **Split screen:** Instantly-style campaign list vs Issue kanban — same sends, clearer “why.”  
3. **Agent clip:** Cursor calls MCP `list_issues(status=waiting)` → drafts follow-ups → human Approve.  
4. **Memory clip:** query “everything sent to jane@acme” across Gmail + Resend identities.

Video-in-sequence as *outreach* tactic (not just GTM): Flowjam notes demo video often punches on later steps ([Flowjam](https://www.flowjam.com/blog/cold-email-templates-for-b2b-saas-that-book-30-more-demos)).

### 3.4 Template & content drops

High-share formats on X/Threads:

- “4-email Issue template for launching to waitlist”
- “Agency partner outreach Issue pack”
- “Breakup email that doesn’t burn the domain”
- Spintax / plain-text packs with **compliance footer** (physical address + opt-out)

Signal-based templates outperform pitch-first ([Flowjam](https://www.flowjam.com/blog/cold-email-templates-for-b2b-saas-that-book-30-more-demos), [Sendspark](https://blog.sendspark.com/cold-email-best-practices)).

### 3.5 Open-source & MCP angles (2026 zeitgeist)

Public examples of indie-friendly positioning:

- **coldforge** — local-first cold outreach toolkit + **MCP server**, no SaaS lock-in narrative: https://github.com/Makeph/coldforge  
- **viralman** — X/Reddit drafts + GitHub stargazer outreach: https://github.com/art8engine/viralman  
- **makerlens** — landing-page-aware personalized emails for indie SaaS: https://github.com/RRYanng/makerlens  

Incumbents already ship MCP (bar rising):

- Instantly official MCP: https://developer.instantly.ai/mcp/introduction  
- Smartlead MCP marketing (vendor claims of large tool counts — treat counts as unaudited): https://www.smartlead.ai/blog/what-is-cold-email-mcp-server  
- Resend official MCP (transactional): https://resend.com/mcp  

**Differentiation for us:** MCP tools centered on **Issues, Memory, dual-provider ledger, HITL approve** — not “yet another send_campaign.”

### 3.6 Before/after reply-rate storytelling (honest rules)

- Anchor against public baselines (Instantly 2026: avg **3.43%**, top quartile **≥5.5%**) — https://instantly.ai/cold-email-benchmark-report-2026  
- Always disclose: list size, ICP, sequence length, time window, positive vs total reply.  
- Prefer **positive reply rate** and **meetings booked** over opens (Google: opens unreliable).  
- Never invent lifts; if n is small, say so.

### 3.7 What *not* to spam on X in 2026

- “Unlimited mailboxes” as sole claim (commodity vs Instantly/Smartlead).  
- Fake inbox-placement percentages.  
- Encouraging ToS-violating Workspace abuse.  
- Autonomous AI SDR hype without HITL (crowded + trust-damaging when wrong).

---

## 4. Packaging & pricing conventions

### KO summary
SMB 콜드이메일 SaaS는 **워크스페이스 정액 + 언리미티드 메일박스**(Instantly/Smartlead) vs **좌석제 멀티채널**(Lemlist)로 갈린다. 크레딧은 리드 DB/AI에 붙는 경우가 많다. 무료는 시험/제한 티어; “퍼머넌트 프리”는 드물다. 인디 PLG는 **$30–100/mo** 밴드가 인지 앵커다.

### EN deep dive

### 4.1 Three dominant models

| Model | Who uses it | Pros | Cons |
|-------|-------------|------|------|
| **Flat workspace + volume tiers** | Instantly, Smartlead | Scales mailboxes without seat tax; agency-friendly | Hidden modules/credits; contact/send caps |
| **Per-seat** | Lemlist Multichannel, Apollo, Mixmax, enterprise SE | Aligns with SDR headcount; multichannel value | Expensive at team scale; mailbox add-ons |
| **Credits** | Instantly Credits, Lemlist lead credits, Clay | Meter AI/enrichment | Stacking invoices; expiry rules |

### 4.2 Snapshot prices (public pages / llms.txt — verify live before quoting customers)

**Instantly Outreach** ([pricing.md](https://instantly.ai/pricing.md), checked via fetch 2026-09-04):

| Plan | Monthly | Emails/mo | Uploaded contacts | Notes |
|------|---------|-----------|-------------------|-------|
| Growth | **$47** | 5,000 | 1,000 | Unlimited accounts & warmup |
| Hypergrowth | **$97** | 125,000 | 25,000 | Full Unibox reply, webhooks, etc. |
| Light Speed | **$358** | 500,000 | 100,000 | SISR / private infra |
| Trial | 14-day | 1,000 emails / 250 contacts | — | No card claimed |

Credits are a **separate** subscription (Nano $9 → Hyper Credits $197+); AI agents consume credits (e.g. Reply Agent 5 credits/reply — vendor table).

**Smartlead** ([pricing](https://www.smartlead.ai/pricing), [llms.txt](https://www.smartlead.ai/llms.txt), help center Aug 2026):

| Plan | Monthly | Annual equiv. | Sends/mo (llms.txt) | Verified prospects |
|------|---------|---------------|---------------------|--------------------|
| Base | **$39** | **$32.50** | 6,000 | 2,000 |
| Pro | **$94** | **$78.30** | 90,000 | 30,000 |
| Unlimited Smart | **$174** | **$144.50** | 150,000 | 50,000 |
| Unlimited Prime | **$379** | **$315** | 510,000 | 170,000 |

All plans: **unlimited email accounts + unlimited warmup**. Whitelabel/client add-ons often cited ~**$29/client** on Pro ([FAQ](https://www.smartlead.ai/faq)).

**Lemlist** (secondary sources disagree slightly; Lemlist vs Instantly page):

- Email plan often cited ~**$69/mo** (or ~**$55** annual)  
- Multichannel ~**$99–109/user/mo**  
- Extra mailboxes historically ~**$9/mailbox/mo** on some plans  
- Lead credits separate (~$0.01/credit on some pages)

Always re-check https://www.lemlist.com before locking GTM pricing claims.

### 4.3 Free tier conventions

| Pattern | Examples |
|---------|----------|
| Time-boxed trial, no permanent free | Instantly 14-day; Smartlead free trial |
| Free enrichment/finder with paid sequences | Hunter freemium; Apollo free tier |
| Dev transactional free | Resend free tier (not cold sequencer) |

**Implication:** Permanent free cold-sending is rare (abuse/spam risk). Offer **trial + generous draft/Issue UX free**, gate **send volume**.

### 4.4 What buyers expect in each price band

| Band | Expectation |
|------|-------------|
| ~$30–50 | Unlimited or many accounts, warmup, basic sequences, CSV import |
| ~$90–180 | Unibox depth, API/webhooks, higher volume, light AI |
| ~$300–400+ | Agency white-label, private infra, CSM |
| Per-seat $70–110 | Multichannel + creative personalization |

### 4.5 Packaging ideas consistent with conventions (for us)

- **Workspace flat** (not per-seat) to compete with Instantly/Smartlead mentally.  
- Meter **sends** or **active prospects**, not mailboxes (align with Smartlead).  
- Optional **credits** for AI draft / enrichment / agent actions.  
- Agency: client workspaces or Issue boards per client — avoid Smartlead’s surprise $29/client tax if possible, or make it transparent.  
- Free: Issue board + templates + Memory preview; paid: send + Unibox + MCP write.

---

## 5. Risks: spam, ToS, Google/Microsoft bulk rules

### KO summary
법적·기술적 리스크는 제품 설계에 직접 들어가야 한다. Google 스팸율 0.3% 천장(권장 0.1%), DMARC/정렬, 원클릭 구독해지, Microsoft 550 5.7.515, CAN-SPAM/GDPR/CASL, Workspace AUP, Resend 등 ESP의 cold 남용 정책이 핵심이다.

### EN deep dive

### 5.1 Spam complaint economics

Google ([sender guidelines](https://support.google.com/a/answer/81126), [FAQ](https://support.google.com/a/answer/14229414)):

- Keep Postmaster spam rate **&lt;0.3%**; aim **&lt;0.1%**.  
- ≥0.3%: delivery mitigations unavailable; filtering worsens.  
- Rate is **user-reported spam / delivered** (provider-side) — ESP dashboards may not match.

Operator arithmetic (Getlead / Warmy-style explainers): at 1,000 inbox-delivered messages, **3 complaints = 0.3%**. Small sends are fragile.

**Product controls:** easy opt-out, relevance gates, sequence length caps, auto-pause on complaint spikes, educate users not to buy lists.

### 5.2 Google bulk rules (hard floor)

Primary: https://support.google.com/a/answer/81126  

Bulk (≥5k/day to personal Gmail): SPF+DKIM+DMARC, alignment, one-click unsubscribe for marketing/subscribed, spam &lt;0.3%, TLS, PTR, RFC 5322.  
Even below bulk threshold, spam rate and authentication affect reputation.

Workspace senders also face Google Workspace **Spam and abuse / AUP** for Workspace-originated mail — product should rate-limit and educate that automation can risk **account suspension**, not only spam folder.

### 5.3 Microsoft consumer bulk rules

Primary: https://support.microsoft.com/en-us/outlook/fix-ndr-error-550-5-7-515-in-outlook-com  

- Trigger: **≥5,000**/day to Outlook.com/Hotmail/Live with same From domain.  
- Need SPF **and** DKIM pass, DMARC published, DMARC alignment.  
- Failure: **`550 5.7.515 Access denied`**.

### 5.4 Legal regimes (product UX implications)

| Regime | Implication |
|--------|-------------|
| **CAN-SPAM (US)** | Accurate headers/subjects; physical postal address; functioning opt-out; honor opt-outs |
| **GDPR / UK GDPR** | Lawful basis (often legitimate interest for relevant B2B); purpose limitation; DSAR; DPA; easy opt-out |
| **CASL (Canada)** | Consent-heavy — geo warnings / blocks for true cold |
| **Other** | AU Spam Act etc. — region warnings |

Cold email ≠ illegal in US B2B if CAN-SPAM followed ([LeadHaste FAQ framing](https://leadhaste.com/blog/cold-email-sending-limits-2026)) — but **spam filters and ToS** still kill programs that are “legal but hated.”

### 5.5 Provider / ESP ToS & AUP risks

| Surface | Risk |
|---------|------|
| Google Workspace / M365 user accounts | Mass cold via API/SMTP may violate acceptable use; suspension / sending limits |
| Shared ESP IPs (if used) | Neighbor spam poisons shared reputation |
| Resend / transactional APIs | Optimized for permissioned mail; cold spray can violate AUP and burn shared pools — dedicated domains + clear policy required |
| LinkedIn automation (Lemlist-class) | Separate ToS risk; agencies often avoid client LinkedIn automation |

### 5.6 Product risk register (MVP-relevant)

1. Users send from primary domain → nuke brand mail.  
2. No DMARC → silent filtering / rejection.  
3. Open tracking default on → worse placement + privacy backlash.  
4. Agent auto-send without HITL → spam + brand damage.  
5. Missing global suppression across Issues → repeated outreach → complaints.  
6. Fabricated deliverability claims in marketing → trust & legal risk.

---

## 6. Differentiation matrix vs Instantly / Smartlead / Lemlist

### KO summary
세 강자는 **볼륨 인프라(Instantly/Smartlead)** 와 **멀티채널 크리에이티브(Lemlist)** 를 차지한다. 우리 비전의 공백은 **Issue형 목적 클러스터링 + (Gmail↔Resend) 이중 발송 원장 + Memory + MCP를 핵심 UX로**. 언리미티드 워밍업만으로는 안 이긴다.

### EN deep dive

### 6.1 Side-by-side (2026)

| Dimension | Instantly | Smartlead | Lemlist | **Our vision** |
|-----------|-----------|-----------|---------|----------------|
| Primary metaphor | Campaign / Sequence | Campaign / Master Inbox | Multichannel campaign | **Issue board (purpose)** |
| Best for | Founders, all-in-one volume | Agencies, API, mailbox farms | SDR creative multichannel | Indies + agencies who think in Linear |
| Pricing shape | Flat + **credits modules** | Flat workspace, unlimited mailboxes | Seat / plan + mailbox extras | Flat workspace (recommended) |
| Entry price (public) | ~$47 Growth | ~$39 Base | ~$69 Email / ~$109 MC seat | Target **~$29–79** indie PLG |
| Mailboxes | Unlimited (plans) | Unlimited (all plans) | Limited per seat + $ extras | Dual identity (Gmail + Resend), not “farm first” |
| Warmup | Large network (vendor claims vary wildly — e.g. 200k–4.2M) | Unlimited included | Lemwarm | Partner or light; don’t claim mega-network day one |
| Unibox | Strong UX | Master Inbox at scale | Multichannel inbox | **Inbox** tied to Issues |
| AI / agents | Sales/Reply Agent (credits) | SmartAgents / NLP | lemAgent / creative AI | **HITL agents + Memory MCP** |
| MCP | Official | Official (vendor-marketed) | Not prominent in sources | **Core**, Issue/Memory tools |
| Lead DB | Modular credits / bundles | Higher plans / SmartProspect | Credits | Optional later; Clay-friendly |
| Multichannel LI/call | No native | Limited / integrations | **Native strength** | Out of MVP scope |
| Dual Gmail + Resend ledger | Connected mailboxes (API) | Connected mailboxes | Connected mailboxes | **Explicit product pillar** |
| Cross-campaign memory | Campaign analytics | Campaign analytics | Campaign analytics | **First-class Memory** |
| Issue-like clustering | **No** | **No** | **No** | **Yes — wedge** |

Sources for competitor cells: Instantly pricing.md, Smartlead llms.txt/pricing, Growth Engineer May 2026 comparison, Lemlist versus pages, market-competitors.md.

### 6.2 Win / lose themes

**We win when buyers care about:**

- Purpose clarity across many small initiatives  
- Not double-emailing the same human  
- Agent-native workflows (Cursor/Claude)  
- Starting simple on Gmail, scaling via Resend without amnesia  
- Delightful desktop/preview craft  

**We lose (initially) when buyers need:**

- 200-mailbox farms tomorrow  
- Largest warmup network claims  
- Native LinkedIn sequence steps  
- Built-in 100M+ lead database  

**Strategic rule:** Don’t lead GTM with “unlimited mailboxes + warmup.” Lead with **Issue + Memory + agent safety**, while matching **table-stakes** sequencer hygiene.

### 6.3 Positioning one-liners (candidates)

- EN: “Linear for cold email — purpose-clustered outreach with send memory, built for AI agents.”  
- KO: “콜드이메일을 이슈로 관리하세요. 목적별 보드 + 발송 기억 + 에이전트 승인.”

---

## 7. Implications for our MVP

### KO summary
MVP는 (1) Issue 보드 + 시퀀스/리마인드, (2) Inbox 회신 분류·시퀀스 중지, (3) SPF/DKIM/DMARC 체크리스트 + 일일 캡 + 억제 목록, (4) 템플릿/프리뷰, (5) Memory의 최소 형태, (6) MCP 읽기 + 승인형 쓰기, (7) Gmail 우선 발송(Resend는 곧이어). 워밍업 거대 네트워크·리드 DB·링크드인은 후순위. 가격은 워크스페이스 정액 + 발송/활성 리드 미터링.

### EN — concrete MVP implications

### 7.1 Must-ship (table stakes + wedge)

| # | Capability | Why |
|---|------------|-----|
| 1 | **Issue board** with statuses (Draft→Closed) | Primary differentiator / GTM demo |
| 2 | Sequence / reminder rules **owned by Issue** | Commodity need under novel UX |
| 3 | **Inbox** with intent tags + auto-pause | Prevent follow-up disasters |
| 4 | Global **suppression** (unsub, bounce, not-interested) across Issues | Complaint & duplicate risk |
| 5 | DNS auth checklist (SPF/DKIM/DMARC) + send caps | Google/MS reality |
| 6 | List verify guidance / bounce &lt;2% warnings | Deliverability |
| 7 | Templates + desktop/mobile preview | Indie craft expectation |
| 8 | **Memory v0**: per-recipient history of touches + template ids | Whitespace; MCP fuel |
| 9 | **MCP read** tools + **write with Approve** | 2026 agent GTM; safety |
| 10 | Compliance footer fields (address, opt-out) | CAN-SPAM / complaint reduction |

### 7.2 Explicit non-goals for MVP

- Competing on warmup network size claims  
- Native LinkedIn/call cadences (Lemlist’s moat)  
- Built-in mega B2B database  
- BIMI / VMC flows  
- Autonomous AI SDR that sends without HITL  
- Open-rate as primary KPI (default tracking **off**)

### 7.3 Ops defaults to bake into product (opinionated)

| Setting | Suggested default |
|---------|-------------------|
| Per-identity daily cold cap | **20–30** (user-configurable, hard warn &gt;40) |
| New domain | Force warmup checklist; block high volume until N days |
| Sequence length UX | Suggest **4–7** steps, 2–4 day gaps |
| Open tracking | **Off** by default |
| DMARC | Setup wizard starting at `p=none` + rua |
| Primary domain send | Hard warning / block for “cold” Issues |
| Agent send | Draft → human Ready/Approve |

### 7.4 Packaging recommendation

- **Trial:** full Issue UX, limited sends  
- **Indie:** ~$29–49/mo flat workspace — enough sends for founder GTM  
- **Pro:** ~$79–99/mo — Unibox depth, MCP write, higher volume, Memory insights  
- **Agency:** client boards / workspaces — transparent pricing (avoid surprise per-client fees)  
- **Credits (optional):** AI rewrite / enrichment only  

### 7.5 GTM sequence for launch content

1. Metaphor demo (Issue board)  
2. Memory anti-duplicate clip  
3. MCP + Approve clip  
4. Honest deliverability checklist thread (cite Google docs)  
5. Template swipe file tied to Issue types  
6. Before/after only with real instrumentation  

### 7.6 Success metrics for MVP (product, not vanity)

- Positive reply rate / Issue  
- % Issues that reach Replied or Closed with outcome tag  
- Duplicate-send rate (should fall with Memory)  
- Time-to-first-approved-send for new workspace  
- Spam complaint / bounce rates (health)  
- MCP tool call → approved send conversion (agent loop quality)

### 7.7 One-paragraph product thesis

In 2026 the cold-email category is an **infrastructure arms race** dominated by Instantly and Smartlead on mailbox economics and Lemlist on multichannel creativity. The remaining human problem is **cognitive**: founders and agencies juggle many *purposes*, forget what was said, and bolt agents onto campaign IDs that weren’t designed for agents. An Issue-board SaaS that ships boring-but-correct deliverability controls, a Unibox that respects sequences, a cross-provider send ledger, and MCP-native Memory can win a wedge **without** out-warming Instantly’s network — if GTM tells that story clearly and MVP doesn’t ship metaphor without send reliability.

---

## 8. Source index

### Primary / official
- Google Email sender guidelines: https://support.google.com/a/answer/81126  
- Google sender guidelines FAQ: https://support.google.com/a/answer/14229414  
- Google BIMI setup: https://support.google.com/a/answer/10911320  
- Microsoft NDR 550 5.7.515: https://support.microsoft.com/en-us/outlook/fix-ndr-error-550-5-7-515-in-outlook-com  
- Instantly pricing.md: https://instantly.ai/pricing.md  
- Instantly 2026 benchmark: https://instantly.ai/cold-email-benchmark-report-2026  
- Instantly sequence workflow: https://instantly.ai/blog/email-sequence-workflow-step-by-step-process-for-building-deploying-sequences/  
- Instantly MCP: https://developer.instantly.ai/mcp/introduction  
- Smartlead pricing: https://www.smartlead.ai/pricing  
- Smartlead llms.txt: https://www.smartlead.ai/llms.txt  
- Smartlead pricing help (updated Aug 2026): https://helpcenter.smartlead.ai/en/articles/439-smartlead-pricing-plans  
- Resend MCP: https://resend.com/mcp  

### Operator playbooks & comparisons
- LeadHaste sending limits 2026: https://leadhaste.com/blog/cold-email-sending-limits-2026  
- Yalc warmup 2026: https://www.yalc.ai/blog/cold-email-warmup/  
- Yalc deliverability 2026: https://www.yalc.ai/blog/cold-email-deliverability/  
- Unify cold email 2026: https://www.unifygtm.com/explore/cold-email-2026-domain-setup-deliverability-sequences  
- Growth Engineer Instantly vs Smartlead vs Lemlist: https://growthengineer.ai/blog/instantly-vs-smartlead-vs-lemlist  
- GTM HQ comparison: https://gtmhq.io/best-cold-email-software-instantly-smartlead-lemlist/  
- Cleanlist Instantly pricing guide (2026-09-01): https://www.cleanlist.ai/blog/2026-09-01-instantly-pricing-guide  
- SalesTarget Unibox: https://salestarget.ai/articles/what-is-a-unified-inbox-for-cold-email  
- mxio bulk sender requirements: https://mxio.io/learn/guides/bulk-sender-requirements  
- Courier sender requirements summary: https://www.courier.com/blog/email-sender-requirements  
- PowerDMARC bulk rules 2026: https://powerdmarc.com/bulk-email-sender-requirements/  
- BIMI requirements overview: https://withsignet.com/blog/bimi-requirements-2026  

### GTM / templates / OSS
- Flowjam cold email templates 2026: https://www.flowjam.com/blog/cold-email-templates-for-b2b-saas-that-book-30-more-demos  
- Sendspark best practices: https://blog.sendspark.com/cold-email-best-practices  
- coldforge: https://github.com/Makeph/coldforge  
- viralman: https://github.com/art8engine/viralman  
- makerlens: https://github.com/RRYanng/makerlens  
- Lemlist vs Instantly: https://www.lemlist.com/versus/lemlist-vs-instantly  

### Internal companion docs
- `/workspace/cold-email-research/market-competitors.md`  
- `/workspace/cold-email-research/issue-board-ia.md`  

---

*End of `01-market-viral-conventions.md`. Generated 2026-09-04 (KST). Metrics not found in sources were left unstated; vendor marketing figures labeled as such.*
