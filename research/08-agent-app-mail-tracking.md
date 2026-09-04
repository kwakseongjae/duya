# Agent-App Email Send + Tracking (2026)

**Document:** `08-agent-app-mail-tracking.md`  
**Audience:** Product / eng for Issue-board cold-outreach (Korean founder; research-only)  
**Research date:** 2026-09-04 (KST)  
**Method:** WebSearch + WebFetch of official docs, MCP READMEs, and 2026 secondary guides. **No fabricated metrics** — tool counts and capabilities are attributed; conflicting marketing vs runtime inventories are called out.  
**Scope:** How **coding/agent apps** (OpenAI Codex, Cursor agents, Claude Code, MCP email tools) send and track mail — **not** Instantly/Smartlead sequencers (covered in `07-mcp-agent-ux.md`).  
**Product context:** Issue-clustered outreach; **Gmail = cold path**; **Resend = permissioned / transactional / verified-domain only**; MCP primary + web Board as HITL/Unibox; **free / OSS first — no paid plans for now**.

**Depends on:** `00-session-brief.md`, `07-mcp-agent-ux.md`, `issue-board-ia.md`, `architecture-prototype.md`

---

## Contents

1. [Executive takeaways](#1-executive-takeaways)
2. [How agent apps send email today](#2-how-agent-apps-send-email-today)
3. [What tracking they expose vs gaps](#3-what-tracking-they-expose-vs-gaps)
4. [Feature checklist (Issue-board, not Instantly)](#4-feature-checklist-issue-board-not-instantly)
5. [Architecture: MCP primary + web Board HITL/Unibox](#5-architecture-mcp-primary--web-board-hitlunibox)
6. [Free / OSS positioning](#6-free--oss-positioning)
7. [Source index](#7-source-index)

---

## 1. Executive takeaways

### KO summary

2026년 코딩 에이전트(Cursor / Claude Code / Codex)의 이메일 패턴은 **시퀀서 MCP가 아니라 ESP·메일박스 MCP**다. Resend는 원격 MCP + Cursor/Claude 플러그인 + Skills로 **송수신·도메인·웹훅·메트릭**을 에이전트에 노출한다. Gmail은 공식 커넥터/플러그인이 **드래프트 우선(송신 제한 또는 불명확)**이고, 자율 송신은 커뮤니티 MCP + OAuth/`gmail.send`다. 트래킹은 Resend 웹훅(`delivered` / `opened` / `clicked` / bounce…)이 풍부하고, **Gmail API에는 open/delivery 이벤트가 없다** — 회신·스레드 ID가 진짜 시그널. Issue-board 제품은 Instantly식 캠페인 미러가 아니라 **Issue·Touch·Memory 도구 + Board Approve + 단일 ledger**로 에이전트가 보내고 추적하게 해야 한다.

### EN deep dive

| 2026 pattern | Implication for Issue-board |
|--------------|----------------------------|
| Hosted remote MCP (Streamable HTTP) + OAuth *or* Bearer / `x-api-key` | Ship `/api/mcp` with workspace-scoped keys; prefer header/OAuth over key-in-URL |
| Vendor plugins (Resend `/add-plugin`, Cursor Gmail marketplace) | Bundle optional Skills later; day-1 = our Issue MCP, not raw Resend for cold |
| Official Gmail paths often **draft-only** or send capability **disputed vs `tools/list`** | Cold path = Gmail OAuth with **server-side draft → Board Approve → send**; never trust host confirm alone |
| ESP MCPs expose get / events / webhooks; Gmail MCPs expose `threadId` / search / reply | Unify into one Touch timeline: provider id + Gmail thread id + reply classification |
| Safety MCPs (allowlist, `confirm_draft`, send codes) are emerging | Encode gates in **our** MCP (`draft_touch` / `schedule_send` + confirm) — see `07` |
| AgentMail = per-agent inboxes (not founder Gmail cold) | Useful reference for thread-first APIs; **not** our cold identity story |
| Free/OSS MCP gateways (AGPL self-host + optional hosted) | Open Issue MCP + Board UI schema; keep multi-tenant host, OAuth secrets, deliverability ops hosted |

**Differentiation (vs Instantly MCP in `07`):** competitors expose *campaign/mailbox API mirrors*. Coding agents need *purpose Issues*: draft in Cursor → Approve on Board → track reply/thread without becoming a sequencer.

---

## 2. How agent apps send email today

Four transport patterns dominate. Products often combine two (e.g. Cursor plugin = remote MCP + Skills).

### 2.1 Pattern map

| Pattern | Auth | Typical clients | Send posture | Best for |
|---------|------|-----------------|--------------|----------|
| **A. Hosted ESP MCP** (Resend, SendLayer, Mailtrap, Nitrosend) | OAuth and/or API key header | Cursor, Claude Code, Codex, Copilot, Windsurf | Direct send / batch / schedule | Transactional and product email; **permissioned** Resend in our stack |
| **B. Official mailbox connector / plugin** (Claude.ai Gmail, Cursor Gmail) | Google OAuth via host product | Claude Code (synced from claude.ai), Cursor | Often **draft + label**; send may be absent or marketing-vs-runtime mismatch | Inbox triage, reply drafts, HITL |
| **C. Community / self-hosted Gmail (or IMAP) MCP** | Google Cloud OAuth desktop client or app password | Claude Code, Cursor, Codex via local process | Can include send_email; better servers gate with confirm / allowlist | Founder Gmail **cold path** when official connector will not send |
| **D. Agent-native inbox API** (AgentMail) | OAuth or custom header key | Claude, Cursor, Codex | Dedicated agent addresses; send/reply/forward/thread | Automations with agent identity — not personal cold Gmail |

Browser automation exists in ad-hoc setups but is not the 2026 first-class pattern in vendor docs reviewed here; prefer MCP + OAuth.

### 2.2 Resend path (permissioned)

Resend exposes a hosted remote MCP plus an open-source local package.
Clients: Cursor, Claude Code, Codex, Copilot, Windsurf (per vendor docs).
Auth: browser OAuth for interactive clients; token header for headless.
Plugin installs bundle MCP with Skills (React Email, deliverability guidance, inbound handling).
Tool groups span outbound mail, inbound mail, templates, contacts, broadcasts, domains, webhooks, and logs.
Docs state agents can retrieve delivery and engagement metrics.
For this product: treat Resend as a permissioned provider behind Board Approve, not as the cold Gmail identity.

### 2.3 Cursor agents

- Resend marketplace plugin: natural-language send and deliverability debug inside Cursor (changelog 2026; `/add-plugin resend`).
- Gmail marketplace plugin (Google Workspace plugins shipped 2026-08-03): marketplace copy is search, read, draft, manage.
- Important mismatch: Cursor launch language may say draft and send, while Google Gmail MCP reference observed 2026-08-05 listed create_draft without send. **Contract = live tools/list**, not marketing (IndieSeek security checklist).
- Third-party remote MCP in `~/.cursor/mcp.json` for multi-inbox aggregators when first-party multi-account is flaky.
- Safe posture in 2026 guides: Read → Stage draft → Human commit on Board or in Gmail UI. Host-side tool confirmation is UX, not a security boundary.

### 2.4 Claude Code

- Official Gmail connector (claude.ai → Claude Code): verified June 2026 inventory has search/read threads, drafts, labels — **no send tool** (CC for Everyone guide, session 2026-06-10).
- Resend official plugin / MCP: ESP send available.
- Community Gmail MCP: can send when scopes allow; prefer staged confirm (examples: claude-gmail-mcp; email-agent-mcp with allowlist; draftgate-mcp trust levels).
- Claude MCP Channels can push inbound events into a session (relevant for reply wakeups).
- Anthropic docs still exemplify creating Gmail **drafts** as the first-party workflow.

### 2.5 OpenAI Codex

- Config via `~/.codex/config.toml` remote URL + env-referenced token (or local command).
- Resend remote MCP addable via Codex CLI.
- SendLayer MCP tools include send-email, get-events, webhook CRUD — pattern for agent-visible delivery state.
- Mailtrap Codex guide: per-tool approval_mode (approve vs auto); default ask before send.
- Community Google Workspace MCP guides list gmail.send / createDraft / search.
- Community perfect-email-mcp maps one Gmail thread per Codex session.
- Host approval_mode complements — does not replace — server-side confirm on our MCP.

### 2.6 AgentMail (agent-native inboxes)

- Hosted MCP; OAuth for Claude-class clients; header key for Cursor-class clients.
- Docs state 24 tools (+ 2 org tools on OAuth): inboxes, threads, messages (send/reply/forward), drafts (including scheduled sendAt), attachments, auth_me.
- Positioning: programmatic agent inboxes with threading and realtime events — not founder Gmail cold identity.
- Reference value: first-class thread_id, draft to send_draft, reply_to_message.

### 2.7 Auth and install summary

- Plugin install often wraps remote MCP plus Skills.
- Remote MCP: browser OAuth or header-based auth.
- Local stdio MCP: environment-based auth.
- Official Gmail connector: host-managed OAuth; often draft-only.
- Community Gmail MCP: user GCP OAuth client; can send; needs HITL gates.
- Browser RPA: fragile; avoid as product path.

---

## 3. What tracking they expose vs gaps

### 3.1 Capability matrix (qualitative — no invented rates)

| Signal | Resend | Official Gmail | Community Gmail MCP | AgentMail |
|--------|--------|----------------|---------------------|-----------|
| Accepted / sent | Yes | Draft; send if tool exists | Message id on send | Message + thread ids |
| Delivered | Yes | No first-class API event | No | Provider-dependent |
| Bounce / fail / complaint | Yes (webhooks) | Via inbound mail | Search inbound | Labels / inbound |
| Opened / clicked | Yes if domain tracking on | Not in Gmail API | Not native | Not core pitch |
| Reply | Inbound + parse | Strong | Strong | Strong |
| Thread id | Limited vs mailbox | First-class threadId | Usually yes | First-class |

Resend official webhook catalog (docs): sent, delivered, delivery_delayed, bounced, complained, opened, clicked, failed, scheduled, received, suppressed. At-least-once; order not guaranteed.

### 3.2 Gmail cold-path gaps (critical)

1. No marketer-style open/click/delivery status in Gmail API — history tracks label changes, not opened-at timestamps (Google history.list docs; Stack Overflow consensus).
2. Image-proxy opens are unreliable for cold mail; industry guidance shifts to reply-rate (Yespo Gmail tracking warning coverage; pixel vs archive architecture notes). Aligns with session brief: transparent n + reply rate.
3. Official connectors often omit send to reduce spam-automation risk (Claude Code Gmail guides).
4. Draft threading bugs reported (standalone drafts vs in-thread); agents must return threadId / Gmail URL; Board shows canonical thread.
5. Multi-account bleed on community servers — bind Touch to from_account_id.

### 3.3 What agents see after send

| Agent question | Typical today | Gap to close |
|---------------|---------------|--------------|
| Did it send? | ESP get/events; Gmail Sent | Persist provider message id + RFC Message-ID on Touch |
| Did they open? | Resend if tracking on; Gmail unavailable/unreliable | Deprioritize opens on Gmail Issues |
| Did they reply? | Manual search / poll | Sync into Inbox row linked to Issue |
| What is the thread? | Gmail threadId; AgentMail get_thread; session maps | Store gmail_thread_id; expose get_thread |
| Follow up? | Chat guesswork | Memory + Waiting + last-touch time |

SendLayer Codex exposing get-events is the right pattern: agents should not scrape dashboards for delivery state.

---

## 4. Feature checklist (Issue-board, not Instantly)

Goal: Codex/Cursor/Claude Code can send + track via our MCP and Board without campaign sequences, unlimited warmup theater, or lead-DB credits.

### 4.1 Must-have (MVP)

| # | Feature | Agent surface | Board / Unibox |
|---|---------|---------------|----------------|
| 1 | Issue-shaped tools (list_issues, get_issue, draft_touch, schedule_send, get_memory) | MCP | Draft → Ready columns |
| 2 | Hard send gate (confirm=true + optional confirmation_token; ack recipient count) | MCP | Approve on Ready |
| 3 | Account binding (Gmail OAuth cold or Resend permissioned) | list_accounts | Accounts settings |
| 4 | Touch ledger (subject, body version, from, provider ids, timestamps) | get_issue / get_memory | Issue timeline |
| 5 | Delivery events (Resend path) | get_touch_events or status fields | sent / delivered / bounced badges |
| 6 | Reply detection (Gmail path primary) | list_inbox, get_thread | Inbox linked to Issue |
| 7 | Thread id persistence | Returned on send; get_thread | Deep link to Gmail |
| 8 | Suppression / bounce / complaint blocks Approve | Server enforce | Memory + badges |
| 9 | Audit log (who drafted, who approved, which agent/key) | list_audit_events | Settings |
| 10 | Idempotent send | idempotency_key or touch id | No double-send on retry |

### 4.2 Should-have (v1)

| # | Feature | Why |
|---|---------|-----|
| 11 | Scoped MCP keys: read / draft / send | Matches Mailtrap/Codex approval spirit + RLS |
| 12 | Draft preview URLs in tool result | HITL in chat or Board |
| 13 | Classify reply (interested / OOO / not now / bounce) | Status moves without sequencer rules engine |
| 14 | Per-account daily caps with warning | Deliverability without a warmup product |
| 15 | Webhook receiver for Resend + optional Gmail history sync | Agents sleep; Board stays live |
| 16 | Optional MCP App Approve card (Phase 2+) | See doc 07 MCP Apps note |

### 4.3 Explicit non-goals (avoid Instantly gravity)

- Campaign subsequences, A/B spintax farms, infinite inbox rotation UI
- Lead database / waterfall credits
- Autopilot reply agent that sends without Board
- Open-rate leaderboards as primary success metric (especially on Gmail)
- Exposing raw Resend full tool catalog as our product MCP

### 4.4 Tracking UX copy (founder-facing)

- Gmail Issues: Sent · Waiting · Replied · Bounced (no open badge, or open N/A on Gmail).
- Resend Issues: may show Delivered / Opened / Clicked when domain tracking is on — labeled as ESP metrics, not truth.
- Primary KPI: reply rate with transparent n (session brief).

---

## 5. Architecture: MCP primary + web Board HITL/Unibox

### 5.1 Roles

```
Cursor / Codex / Claude Code
        |  MCP (Streamable HTTP)
        v
Issue-board MCP (workspace JWT / MCP key)
        |  same Postgres + RLS
        +----> Web Board + Inbox (Unibox) Approve/edit
        |
        +----> Memory / Touches ledger
        |
        +----> Workers -> Gmail API and/or Resend after Approve
```

| Layer | Responsibility |
|-------|----------------|
| MCP | Primary agent interface: memory, draft, prepare/schedule with confirm, inbox, thread/events |
| Web Board | Primary human interface: kanban, preview, Approve/Reject, Unibox triage, account connect |
| Providers | Gmail = cold path OAuth; Resend = permissioned verified-domain + rich webhooks |
| Workers | Ingest Resend webhooks; sync Gmail history; classify replies; enforce caps |

Agents do not become Instantly: no start-campaign tool. They operate Issues and Touches; humans own irreversible send via Board or explicit MCP confirm echoing a Board-issued token.

### 5.2 Recommended send flows

**A. Agent drafts → Human Approves on Board (default)**
1. draft_touch → Touch draft
2. update_issue_status → Ready (or human moves card)
3. Human Board Approve → worker sends via Gmail/Resend
4. Events/replies on timeline; agent later get_memory / list_inbox

**B. Agent schedules with confirm (power user)**
1. preview_touch / prepare_send → confirmation_token
2. schedule_send(confirm=true, confirmation_token=..., ack_recipient_count=n)
3. Same worker path as Board Approve

**C. Never:** agent holds raw provider credentials and sends outside the ledger.

### 5.3 Unibox = HITL, not sequencer inbox

- Single stream: replies, bounces, complaints keyed by Issue/Touch
- Actions: classify, snooze Issue, close, draft reply
- Optional Slack later — Board first for OSS/free

### 5.4 Alignment with 07-mcp-agent-ux.md

Reuse tool names and confirm rules from sections 6–7 of doc 07. This doc adds the tracking and agent-app ecosystem lens.

---

## 6. Free / OSS positioning

**Constraint (this research wave):** no paid plans for now — free hosted and/or open-source wedge. Doc 05 sketched Indie/Pro ladders for later; treat paid SKUs as future.

### 6.1 What to open-source

| Artifact | License leaning | Why |
|----------|-----------------|-----|
| Issue MCP server (schemas + reference impl) | MIT or Apache-2.0 | Virality for Cursor/Codex users; matches Resend open MCP package pattern |
| AGENTS.md / skills for Issue-board workflows | MIT | Token-efficient agent behavior without shipping secrets |
| Board UI kit (shadcn Issue board + Inbox, ko/en) | MIT | Shareable differentiation; cold email as Issues demo |
| Webhook normalizer (Resend events → Touch timeline) | MIT | Commodity; builds trust |
| Docker-compose self-host | Same as app | Privacy-minded founders (Korea + EU narrative) |

### 6.2 What to keep hosted (even while free)

| Capability | Why not OSS-only |
|------------|------------------|
| Google OAuth client + token vault | Brand verification, redirect URIs, security responsibility |
| Multi-tenant RLS / workspace isolation | Easy to get wrong in forks |
| Deliverability defaults (caps, suppression, bounce handling) | Shared learning; abuse prevention |
| Production webhook endpoint + signing secrets | Needs stable public URL |
| Abuse / rate limits / ToS enforcement | Free tier still attracts spam actors |
| Optional managed MCP URL | OAuth UX parity with Resend/AgentMail |

### 6.3 Free-tier gates (while price is zero)

| Gate | Suggestion |
|------|------------|
| Workspaces / seats | 1 / 1–2 |
| Active Issues | Low (e.g. 3–10) |
| Sends / month | Hard cap (Gmail quotas + any Resend usage) |
| MCP | On for free (viral wedge); send scope needs connected account + daily cap |
| Resend | Opt-in; user BYO key or shared low-volume pool with strict caps |
| Data export | Allowed (OSS goodwill) |

### 6.4 OSS business analogies (2026 MCP market)

- Resend: open-source MCP package + hosted remote MCP + commercial ESP.
- AGPL gateways (PlugMCP, AnythingMCP): full self-host free; commercial license to lift copyleft — later option if cloud clones appear.
- Recommendation now: **MIT core MCP + Board** for distribution; hosted multi-tenant free app for OAuth/sync; revisit AGPL only if needed.

### 6.5 Positioning lines (KO / EN)

- KO: Cursor에서 Issue 초안 → 보드에서 승인 → Gmail/Resend 기록이 하나로.
- EN: Draft cold email as Issues in Codex/Cursor. Approve on the Board. Track replies — not open-rate theater.
- Explicit: Not Instantly. No sequences. Free MCP for coding agents.

---

## 7. Source index

### Resend / ESP MCP

- https://resend.com/docs/mcp-server
- https://www.npmjs.com/package/resend-mcp
- https://resend.com/changelog/cursor-plugin
- https://resend.com/cursor
- https://resend.com/claude-code
- https://resend.com/docs/webhooks/event-types
- https://resend.com/docs/webhooks/introduction
- https://sendlayer.com/blog/how-to-automate-email-sending-in-codex/
- https://docs.mailtrap.io/guides/ai-powered-integrations/codex
- https://nitrosend.com/email-mcp

### Cursor / Claude / Codex / Gmail

- https://code.claude.com/docs/en/mcp
- https://ccforeveryone.com/guides/connect-claude-code-to-gmail
- https://cursor.com/marketplace/cursor/gmail
- https://indieseek.co/blogs/cursor-google-workspace-plugins-security-checklist/
- https://snov.io/blog/how-to-send-emails-with-claude/
- https://github.com/pliablepixels/claude-gmail-mcp
- https://github.com/UseJunior/email-agent-mcp
- https://thinhdanggroup.github.io/google-workspace-mcp-for-codex/
- https://github.com/goyalayus/perfect-email-mcp
- https://www.usecarly.com/blog/how-to-connect-multiple-gmail-accounts-to-cursor/

### Agent inboxes and safety gates

- https://www.agentmail.to/
- https://www.agentmail.to/docs/integrations/mcp
- https://github.com/DRMM101/draftgate-mcp
- https://github.com/xenji/simple-email-mcp

### Tracking limitations

- https://developers.google.com/workspace/gmail/api/reference/rest/v1/users.history/list
- https://stackoverflow.com/questions/67959524/gmail-api-how-do-i-see-a-email-opening-date
- https://yespo.io/blog/did-google-just-end-open-tracking-gmail
- https://resources.mailertogo.com/faq/does-gmail-archive-count-as-read-email-open-tracking

### Related internal docs

- `07-mcp-agent-ux.md` — Instantly/Smartlead MCP + our tool catalog and confirm gates
- `00-session-brief.md` — Issue primary object; Gmail + Resend ledger; reply-rate KPI
- `05-pricing-packaging-gtm.md` — future paid SKUs (deferred while free/OSS)

---

*End of `08-agent-app-mail-tracking.md`. Research-only; no implementation in this wave.*
