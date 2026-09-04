# MCP + AI Agent UX for Outbound / Cold Email (2026)

**Document:** `07-mcp-agent-ux.md`  
**Audience:** Product / eng for Issue-board cold-outreach SaaS  
**Research date:** 2026-09-04 (KST)  
**Method:** Official docs + vendor blogs via WebFetch/WebSearch. **No fabricated tool counts** — every number is attributed; conflicting vendor claims are called out.  
**Depends on:** `issue-board-ia.md`, `architecture-prototype.md` §6, `00-session-brief.md`, `02-stack-blueprint.md`

---

## Contents

1. [Executive takeaways](#1-executive-takeaways)
2. [Instantly MCP](#2-instantly-mcp)
3. [Smartlead MCP — claims vs constraints](#3-smartlead-mcp--claims-vs-constraints)
4. [Resend MCP](#4-resend-mcp)
5. [Claude / Cursor agent workflows for sales email](#5-claude--cursor-agent-workflows-for-sales-email)
6. [Our MCP tool surface ↔ Issue Board IA](#6-our-mcp-tool-surface--issue-board-ia)
7. [Safety gates: draft vs schedule_send confirm](#7-safety-gates-draft-vs-schedule_send-confirm)
8. [Human-in-the-loop patterns](#8-human-in-the-loop-patterns)
9. [Audit logs](#9-audit-logs)
10. [Demo scripts for viral clips](#10-demo-scripts-for-viral-clips)
11. [Source index](#11-source-index)

---

## 1. Executive takeaways

### KO summary
2026년 콜드이메일 MCP는 **대시보드 대신 대화로 캠페인/회신/발송을 조작**하는 레이어다. Instantly·Smartlead는 “풀 API를 도구로 노출” 경쟁, Resend는 Cursor/Claude용 **트랜잭션·브로드캐스트** MCP + Skills. 우리는 **Issue Board IA에 맞춘 작은 도구 표면** + **draft 기본 / schedule_send 확인 게이트**로 차별화한다. 도구 개수 자랑보다 **HITL·감사로그·바이럴 데모**가 제품 이야기.

### EN deep dive

| Pattern in market (2026) | Implication for us |
|--------------------------|--------------------|
| Hosted remote MCP (Streamable HTTP / SSE) + API key or OAuth | Prefer remote `/api/mcp` with workspace JWT / scoped MCP keys — not “paste API key in URL” as primary |
| Tool catalogs mirror full product APIs (Instantly auto-gen from V2; Smartlead “116+” marketing claim) | **Do not** ship a raw IMAP/API mirror. Ship **Issue-shaped** tools that match Board IA |
| Reply agents: HITL vs Autopilot (Instantly) | Default agent mode = **draft + propose**; Approve on Board `Ready` |
| Resend = developer transactional MCP in Cursor/Claude | Use Resend as **sending backend**, not as the product MCP story |
| MCP Apps (Jan 2026) enable inline Approve/Edit/Reject cards | Phase 2+: optional MCP App approval card for `schedule_send` |
| Cursor often pauses before destructive tool calls | Still enforce **server-side** confirm + quotas — host UI is not enough |

**Differentiation bet:** competitors expose *campaign/mailbox ops*; we expose *purpose Issues* (`get_memory`, `draft_touch`, gated `schedule_send`) — the viral clip is “ask Cursor what we already sent this person,” not “list 38 API endpoints.”

---

## 2. Instantly MCP

### Official surface

| Fact | Source |
|------|--------|
| Hosted endpoint | `https://mcp.instantly.ai/mcp` — [developer intro](https://developer.instantly.ai/mcp/introduction) |
| Scope | **Full Instantly V2 API** as MCP tools; tools auto-generated from the same API spec as the reference — [tools page](https://developer.instantly.ai/mcp/tools) |
| Naming | `{action}_{resource}` e.g. `list_campaigns`, `create_lead`, `get_campaign_analytics` |
| Auth (recommended) | `Authorization: YOUR_API_KEY` or `Bearer …`; also `x-instantly-api-key` — [auth docs](https://developer.instantly.ai/mcp/authentication) |
| Auth (fallback) | URL path `…/mcp/YOUR_API_KEY` — Instantly warns this may appear in logs |
| Clients called out | Claude, Cursor — [intro](https://developer.instantly.ai/mcp/introduction) |

**Official categories** (developer tools page; not a fixed count): Accounts, Campaigns, Subsequences, Leads, Email, Analytics, Webhooks, Enrichment, Workspace, Custom tags, Blocklist, OAuth, API keys, Background jobs.

### Marketing tool-count claims (do not treat as official)

| Claim | Where | Note |
|-------|-------|------|
| **38 tools across 6 categories** | Instantly blog, Feb 24, 2026 — [Automate Sales Operations…](https://instantly.ai/blog/how-to-automate-sales-operations-with-instantly-mcp-server/) | Categories listed: Accounts, Campaigns, Leads, Emails, Analytics, Background jobs |
| **31 tools across five categories** | Instantly blog — [MCP Server for Sales](https://instantly.ai/blog/mcp-server-sales/) | Conflicts with “38” claim |
| Official docs | [Available tools](https://developer.instantly.ai/mcp/tools) | **No fixed integer** — “every API endpoint” |

**Product rule:** cite Instantly as “full V2 API as MCP tools” unless quoting a specific blog with date. Do not invent a third count.

### Setup UX (Claude Desktop pattern)

From Instantly’s Feb 2026 ops blog:

```json
{
  "mcpServers": {
    "instantly": {
      "url": "https://mcp.instantly.ai/mcp",
      "headers": {
        "Authorization": "PASTE_YOUR_API_KEY_HERE"
      }
    }
  }
}
```

API keys: Settings → Integrations → API (Growth+ plans per Instantly marketing).

### Agent UX patterns Instantly markets

1. **Conversational ops:** “pause campaigns under 2% reply rate,” batch sentiment tagging of Unibox replies, bounce hygiene prompts.
2. **Draft-then-review:** prompts that “Save each draft in Unibox for my review” before send.
3. **Separate product: AI Reply Agent** — not the MCP server itself:
   - **HITL:** approve every draft (Unibox and/or Slack)
   - **Autopilot:** send when confident; escalate when unclear
   - Docs: [Help Center — AI Reply Agent](https://help.instantly.ai/en/articles/11774076-ai-reply-agent), [HITL vs Autopilot guide](https://instantly.ai/blog/ai-reply-agent-guide/)

**Takeaway for us:** Instantly’s *marketing* story is speed + agency Unibox triage. Their *MCP* story is “Claude can drive the whole API.” Safety lives partly in Reply Agent modes, not as a first-class `confirm` parameter on every send tool in public MCP docs.

---

## 3. Smartlead MCP — claims vs constraints

### Official help center (ground truth for setup)

| Fact | Source |
|------|--------|
| Purpose | Claude interacts with Smartlead data: campaign insights, deliverability diagnostics, lead/account/performance — [Help Center article 300](https://helpcenter.smartlead.ai/en/articles/300-smartlead-mcp-server) |
| Transport | **SSE only** for now; help center says HTTPS transport is **not** supported |
| Client | FAQ: MCP integrations supported **only in Claude Desktop** for now; web “later” |
| Setup | `npx mcp-remote` → `https://mcp.smartlead.ai/sse?user_api_key=YOUR_API_KEY` |
| Example tools shown | `get_campaign_status`, `fetch_lead_data`, `check_deliverability` |
| Security note in FAQ | Keys “stay local in Claude config” — but the **key is in the URL query string** (weaker than Instantly/Resend header/OAuth patterns) |

### Marketing claims (labeled)

| Claim | Source | How we use it |
|-------|--------|---------------|
| **116+ tools** across 6 categories | Smartlead blog [What Is a Cold Email MCP Server?](https://www.smartlead.ai/blog/what-is-cold-email-mcp-server) (updated Aug 2026); setup guide [smartlead-mcp-setup-guide](https://www.smartlead.ai/blog/smartlead-mcp-setup-guide) | Attribute as **vendor claim**. Help center does **not** publish a verified inventory count. |
| Categories | Campaign management, lead lifecycle, email account management, deliverability diagnostics, analytics, webhook automation | Useful IA for competitive feature matrix |
| “Most comprehensive” vs Instantly’s 38 | Same Smartlead blog | Competitive framing — not independent verification |
| Independent write-up of constraints | [RevenueFlow — Smartlead MCP](https://www.revenueflow.com/blog/smartlead-mcp) | Reinforces SSE-only, Claude Desktop-first, key-in-URL |

### UX patterns Smartlead markets

- Deliverability audit in natural language (“bounce rate >5% last 7 days”)
- Pre-launch configuration checks
- Reply-gap audits (replied but still in sequence)
- Morning health prompts for SDR / sales ops

**Takeaway for us:** Smartlead wins on **ops depth messaging** (warmup, spam risk, webhooks). Constraints (SSE, Claude Desktop, key-in-URL) are product liabilities we should avoid. Our story: Streamable HTTP + OAuth/header auth + Issue-shaped tools + confirm gates.

---

## 4. Resend MCP

### Official surface

| Fact | Source |
|------|--------|
| Remote URL | `https://mcp.resend.com/mcp` — [MCP docs](https://www.resend.com/docs/mcp-server) |
| Auth (remote) | Browser **OAuth** on connect; or `Authorization: Bearer re_…` for headless/CI |
| Local package | `npx -y resend-mcp` (stdio or `--http`) |
| Changelog | **10 tool groups** covering Resend APIs — [changelog/mcp](https://resend.com/changelog/mcp) |
| Cursor marketing | “**85+** precision tools” for sending + templates — [resend.com/cursor](https://resend.com/cursor) (vendor page claim) |
| Cursor plugin | `/add-plugin resend` bundles MCP + Skills — marketplace listing [cursor.com/marketplace/mcp/resend](https://cursor.com/marketplace/mcp/resend) |

### Capability groups (docs — no fabricated per-tool totals)

From [Resend MCP docs](https://www.resend.com/docs/mcp-server):

- **Emails:** send, list, get, cancel, update, batch; HTML/text; attachments; schedule; tags; topics  
- **Received emails:** list/read inbound; download attachments  
- **Templates, Contacts, Broadcasts, Automations, Events**  
- **Domains, Segments, Topics, Contact properties, API keys, Webhooks, Logs**  
- **Editor:** connect to visual editor drafts  

Changelog’s original “10 tool groups” list: emails, contacts, broadcasts, domains, webhooks, segments, topics, contact properties, API keys, received emails — docs now also describe templates, automations, events, logs, editor (product expanded; cite docs page for current set).

### Cursor / Claude workflow shape

Resend’s GTM is **developer email inside the coding agent**:

1. Install plugin or remote MCP  
2. Install **Skills** (`npx skills add resend/resend-skills`) for React Email + best practices  
3. Natural-language prompts: welcome email, password reset template, weekly digest, order shipped  

This is **transactional / product email**, not cold Unibox ops. For our stack: Resend is the **provider** under Accounts; our MCP talks Issues/Touches, then the app calls Resend (or Gmail) — agents should not need raw Resend tools for day-1 founder workflows unless debugging deliverability.

---

## 5. Claude / Cursor agent workflows for sales email

### Observed end-to-end patterns (2026)

```
User intent (NL)
    → Agent plans multi-tool chain
        → Read tools (list/get/stats/memory)     [low risk]
        → Draft tools (create Issue, draft_touch) [medium; reversible]
        → Mutating ops (pause, suppress)          [medium-high]
        → Send / schedule                         [irreversible → HITL]
    → Human sees preview + Approve
    → Provider send + webhook timeline
```

### Claude Desktop / Claude.ai

- Custom connectors / `claude_desktop_config.json` for remote MCP  
- Instantly & Smartlead both optimize for Claude first  
- Instantly Reply Agent: Slack approval channel as second surface  
- MCP Apps (spec **2026-01-26**): interactive cards (Approve / Edit / Reject) inside chat — [MCP Apps overview](https://modelcontextprotocol.io/extensions/apps/overview.md); implementation writeups: [Thomas Wiegold — approval gate](https://thomas-wiegold.com/blog/how-to-build-mcp-ui/), [Keldrik/mcp-approval-gate](https://github.com/Keldrik/mcp-approval-gate)

### Cursor

- Settings → MCP → global servers (`url` or `command`+`env`)  
- Resend remote or local; Instantly header auth works similarly  
- Agent UI typically surfaces tool args; **hosts may ask before destructive calls** ([Brixus365 email MCP article](https://www.brixus365.com/blog/email-automation-mcp-claude-cursor-cline)) — treat as UX nicety, not security boundary  
- Best viral fit for *our* product: founder already in Cursor → `get_memory("Jane @ Acme")` without opening a SaaS tab

### Workflow archetypes we should support

| Archetype | Example prompt | Tools |
|-----------|----------------|-------|
| **Memory lookup** | “What did we send to the VP Sales at Notion?” | `get_memory`, `list_issues` |
| **Purpose draft** | “Open an Issue for Series B fintech intros; draft first touch for these 5” | `create_issue`, `add_prospects`, `draft_touch`, `list_templates` |
| **Ready for approve** | “Move Issue X to Ready and summarize who gets what” | `get_issue`, `preview_touch`, status → Ready |
| **Gated send** | “Schedule Issue X Tue 09:00 KST” | `schedule_send` + confirm |
| **Inbox triage** | “Classify last 24h replies; pause complainers” | `list_inbox`, `classify_reply`, `pause_issue` |
| **Health** | “Any account near bounce budget?” | `list_accounts`, `get_account_health` |

---

## 6. Our MCP tool surface ↔ Issue Board IA

Maps 1:1 to [issue-board-ia.md](./issue-board-ia.md) primary nav and statuses. Prefer **few sharp tools** over Instantly/Smartlead-style API mirrors.

### Tool catalog (proposed MVP → v1)

| Tool | Risk | Maps to IA | Notes |
|------|------|------------|-------|
| `list_issues` | Read | **Board** | Filters: status, owner, tag, account |
| `get_issue` | Read | **Issue detail** | Prospects, next touch, sequence rules, stats summary |
| `create_issue` | Write | Board quick-create | Purpose one-liner required |
| `update_issue_status` | Write | Board columns | Enforced transitions (e.g. cannot skip Ready→Sending without schedule path) |
| `list_prospects` / `add_prospects` | Read/Write | Issue → Prospects | Dedupe by email; respect suppression |
| `list_templates` / `get_template` | Read | **Templates** | Version id immutable on send |
| `draft_touch` | Write (draft) | Compose | **Never sends.** Creates Touch `status=draft` |
| `revise_touch` | Write (draft) | Compose | Edit subject/body/from account |
| `preview_touch` | Read | Compose preview | Returns text + desktop/mobile preview URLs |
| `schedule_send` | **Send** | Ready → Sending | **Requires confirm gate** (below) |
| `cancel_scheduled` | Write | Sending / Waiting | Soft cancel before provider fire |
| `pause_issue` / `close_issue` | Write | Snoozed / Closed | |
| `get_memory` | Read | **Memory** | Person/company → past Issues, touches, winners, suppression |
| `list_inbox` / `get_thread` | Read | **Inbox** | Replies/bounces/complaints linked to Issue |
| `classify_reply` | Write | Inbox | Interested / not / OOO / bounce / complaint → status side-effects |
| `get_stats` | Read | Issue Stats | Reply-rate with **n ≥ threshold** for badges |
| `list_accounts` / `get_account_health` | Read | **Accounts** | Caps remaining, bounce/complaint |
| `list_audit_events` | Read | Settings / compliance | See §9 |

Aligns with stack blueprint Phase 2 note: `list_issues`, `get_issue`, `draft_touch`, `schedule_send` (HITL), `get_memory`, `get_stats`, `list_templates`.

### Status machine the agent must respect

```
Draft → Ready → Sending → Waiting → Replied
                 ↘ Snoozed
                 ↘ Closed
```

- Agent may freely create Drafts and propose Ready.  
- **Sending** only via `schedule_send` / approved send path (not raw `update_issue_status`).  
- Suppression / do-not-contact blocks Approve (IA rule).

### Authz

- Same workspace membership + RLS as the web app (`02-stack-blueprint.md`).  
- MCP keys: scoped `read` | `draft` | `send` (send implies draft+read).  
- Never a privileged service role for user-initiated agent calls.

---

## 7. Safety gates: draft vs schedule_send confirm

### Design principle

| Action class | Default | Gate |
|--------------|---------|------|
| Read (list/get/memory/stats) | Allowed | None |
| Draft / revise / preview | Allowed | Soft: rate-limit; log |
| Pause / suppress / close | Allowed with write scope | Log + undo where possible |
| **Schedule / send** | **Denied until confirm** | Hard server gate |

### `draft_touch` contract

```json
{
  "name": "draft_touch",
  "description": "Create or update a draft Touch on an Issue. Does not send or schedule.",
  "inputSchema": {
    "issue_id": "uuid",
    "prospect_id": "uuid",
    "template_version_id": "uuid?",
    "subject": "string?",
    "body_text": "string?",
    "from_account_id": "uuid?",
    "reasoning": "string?"
  }
}
```

Returns: `touch_id`, preview payload, `status: "draft"`. Side effect: Issue may stay Draft or move Ready only if user/agent explicitly requests Ready (separate tool).

### `schedule_send` contract (confirm required)

```json
{
  "name": "schedule_send",
  "description": "Schedule approved Touches for an Issue. Irreversible once provider accepts. Requires confirm=true and optional confirmation_token from preview.",
  "inputSchema": {
    "issue_id": "uuid",
    "touch_ids": ["uuid"],
    "send_at": "iso8601?",
    "confirm": "boolean",
    "confirmation_token": "string?",
    "ack_recipient_count": "number?"
  }
}
```

**Server rules (non-negotiable):**

1. Reject if `confirm !== true`.  
2. Reject if any prospect is suppressed / unsubscribed / hard-bounced.  
3. Reject if Issue not in `Ready` (or explicit override role).  
4. Reject if `ack_recipient_count` ≠ actual recipient count (anti-silent-broaden).  
5. Prefer two-step: `preview_touch` / `prepare_send` → returns `confirmation_token` (TTL ~10–15 min) → `schedule_send` must echo token.  
6. Enforce per-account daily/hourly caps server-side.  
7. Emit audit event `send.scheduled` with actor = MCP principal.

Optional Phase 2: MCP App card (Approve / Edit / Reject) instead of bare `confirm` boolean — pattern from [mcp-approval-gate](https://github.com/Keldrik/mcp-approval-gate) and Instantly Slack HITL.

### Anti-patterns to avoid

- Single `send_email` tool that agents call “helpfully” without Issue context  
- Putting API keys in SSE query strings (Smartlead pattern) as our primary auth  
- Trusting Cursor/Claude “Approve tool call” UI alone  
- Autopilot cold sends on day one (Instantly Autopilot is reply-side and plan-gated; we stay HITL for outbound)

---

## 8. Human-in-the-loop patterns

### Pattern A — Board Approve (primary UX)

1. Agent runs `draft_touch` (+ maybe `update_issue_status` → Ready).  
2. Founder opens **Board** → Issue in Ready → **Approve send**.  
3. UI shows desktop/mobile preview, account chips, recipient list, suppressions.  
4. Approve calls same internal path as `schedule_send` (shared service).

*This is the IA default:* “Default agent mode: draft + propose, human Approve on Board/Ready” (`issue-board-ia.md`).

### Pattern B — In-chat confirm (MCP client)

1. Agent proposes schedule; host shows tool args.  
2. User clicks allow **or** agent must pass `confirm: true` only after user typed “yes, schedule.”  
3. Still enforce token + counts server-side.

### Pattern C — MCP App approval card (Phase 2+)

1. Agent calls `request_approval` with subject, to, body preview.  
2. Inline card: Approve / Edit / Reject.  
3. On Approve, server returns effective payload; only then `schedule_send`.  
4. Spec backdrop: MCP Apps extension (2026-01-26).

### Pattern D — Slack / async review (Instantly-like)

- Useful for agencies; not MVP.  
- Mirror Instantly Reply Agent: draft → Slack → approve → send.  
- For us: more natural on **replies** than first cold touch.

### Pattern E — Autopilot (explicitly late)

- Only after HITL quality is proven; never default.  
- Scope narrowly (e.g. reminder touches only, same template, no new ICP).  
- Instantly gates Autopilot by plan and confidence — we should gate by workspace policy flag + audit.

### Escalation cues (always human)

- Complaint / unsubscribe language  
- Legal / pricing negotiation  
- First send from a new domain/account  
- Batch size above workspace threshold (e.g. >25 prospects)

---

## 9. Audit logs

### Why

Outbound + agents = compliance, blame assignment, and demo trust. Instantly scopes API keys; Resend exposes request **Logs** via MCP; confirm.dev-style approval tools keep immutable decisions. We need first-class `audit_events`.

### Schema sketch

| Field | Purpose |
|-------|---------|
| `id`, `workspace_id`, `created_at` | Tenant timeline |
| `actor_type` | `user` \| `mcp_key` \| `system` \| `webhook` |
| `actor_id` | User id / MCP key id |
| `action` | e.g. `touch.drafted`, `send.scheduled`, `send.cancelled`, `issue.status_changed`, `reply.classified`, `suppression.added` |
| `resource_type` / `resource_id` | Issue, Touch, Account, … |
| `request_id` | Correlate MCP `tools/call` |
| `tool_name` | If via MCP |
| `payload_redacted` | Diff / args with secrets stripped |
| `ip` / `user_agent` | Optional for key use |
| `outcome` | `ok` \| `denied` \| `error` |
| `denial_reason` | e.g. `confirm_missing`, `suppressed_recipient` |

### Product surfaces

- Settings → **Audit** (filter by Issue, actor, action)  
- MCP: `list_audit_events` (read scope)  
- Export CSV for agencies  
- Retention: e.g. 90 days MVP; longer on paid

### Mandatory logged transitions

Every `schedule_send`, `cancel_scheduled`, Approve on Board, suppression add, MCP key create/revoke, account connect/disconnect.

---

## 10. Demo scripts for viral clips

Target length: **20–45s**. Hook from session brief: “Cold email as Issues,” `get_memory` in Cursor, Gmail+Resend one ledger, transparent reply-rate n.

### Clip 1 — “Cold email as Issues” (Board → Approve → Reply)

| Beat | On screen | VO / caption |
|------|-----------|--------------|
| 0–5s | Empty Board → create Issue “Warm intros · Series B fintech” | “Campaigns are blasts. This is an Issue.” |
| 5–15s | Add 3 prospects; agent `draft_touch`; preview desktop/mobile | “Agent drafts. Nothing sends.” |
| 15–25s | Drag/click **Ready → Approve**; status **Sending** | “You approve. Then it sends.” |
| 25–40s | Inbox reply → Issue **Replied** | “Reply lands on the Issue — not a mystery Unibox.” |

**CTA:** “Issue-shaped outbound. MCP optional.”

### Clip 2 — Cursor `get_memory` (MCP viral)

| Beat | On screen | VO / caption |
|------|-----------|--------------|
| 0–5s | Cursor chat | “Don’t open the CRM.” |
| 5–20s | Prompt: *What did we already send to Mira at Acme?* → tool call `get_memory` | Live tool chip |
| 20–35s | Answer: prior Issue, subjects, winner snippet, “not suppressed” | “Memory is a tool, not a tab.” |
| 35–45s | Optional: `draft_touch` follow-up → stops before send | “Draft yes. Send only with confirm.” |

**CTA:** Connect MCP key → same Board data.

### Clip 3 — Safety gate ASMR

| Beat | On screen | VO / caption |
|------|-----------|--------------|
| 0–10s | Agent tries `schedule_send` without `confirm` → **denied** toast / tool error | “Agents don’t get a send button for free.” |
| 10–25s | Preview card: 12 recipients, from accounts, suppressions 0 | Count must match |
| 25–40s | `confirm: true` + token → Scheduled | “HITL is the product.” |

### Clip 4 — Gmail + Resend one ledger

| Beat | On screen | VO / caption |
|------|-----------|--------------|
| 0–15s | Accounts: Gmail chip + Resend domain | “One Issue, two pipes.” |
| 15–35s | Timeline: Touch via Gmail, reminder via Resend, same Issue | Single stats strip with **reply n=** |

### Prompt pack (for viewers to copy)

```
List my Issues in Ready and summarize who is waiting for approval.
```

```
get_memory for <name> at <company> — what Issues and subjects?
```

```
Draft a first touch for Issue <id> using template <name>; do not send.
```

```
Schedule Issue <id> for Tuesday 09:00 Asia/Seoul — ask me to confirm with recipient count.
```

```
Show bounce/complaint cues in Inbox from the last 48 hours and pause those Issues.
```

---

## 11. Source index

### Instantly
- [MCP Introduction](https://developer.instantly.ai/mcp/introduction)  
- [Available tools](https://developer.instantly.ai/mcp/tools)  
- [Authentication](https://developer.instantly.ai/mcp/authentication)  
- [Automate Sales Operations with Instantly MCP (38 tools claim)](https://instantly.ai/blog/how-to-automate-sales-operations-with-instantly-mcp-server/)  
- [MCP Server for Sales (31 tools claim)](https://instantly.ai/blog/mcp-server-sales-automation/)  
- [AI Reply Agent Help Center](https://help.instantly.ai/en/articles/11774076-ai-reply-agent)  
- [AI Reply Agent HITL guide](https://instantly.ai/blog/ai-reply-agent-guide/)  

### Smartlead
- [Help Center — Smartlead MCP Server](https://helpcenter.smartlead.ai/en/articles/300-smartlead-mcp-server)  
- [What Is a Cold Email MCP Server? (116+ claim)](https://www.smartlead.ai/blog/what-is-cold-email-mcp-server)  
- [Smartlead MCP setup guide](https://www.smartlead.ai/blog/smartlead-mcp-setup-guide)  
- [RevenueFlow analysis (constraints)](https://www.revenueflow.com/blog/smartlead-mcp)  

### Resend
- [MCP Server docs](https://www.resend.com/docs/mcp-server)  
- [Official Resend MCP changelog (10 tool groups)](https://resend.com/changelog/mcp)  
- [Send emails with Cursor (85+ claim)](https://resend.com/cursor)  
- [resend.com/mcp](https://resend.com/mcp)  
- [Cursor marketplace — resend](https://cursor.com/marketplace/mcp/resend)  

### Protocol / HITL
- [MCP Apps overview](https://modelcontextprotocol.io/extensions/apps/overview.md)  
- [MCP Tools spec (2026-07-28)](https://modelcontextprotocol.io/specification/2026-07-28/server/tools)  
- [Approval gate walkthrough](https://thomas-wiegold.com/blog/how-to-build-mcp-ui/)  
- [Keldrik/mcp-approval-gate](https://github.com/Keldrik/mcp-approval-gate)  
- [confirm-ai/confirm-mcp](https://github.com/confirm-ai/confirm-mcp)  
- [Email automation MCP (Claude/Cursor/Cline)](https://www.brixus365.com/blog/email-automation-mcp-claude-cursor-cline)  

### Internal
- `issue-board-ia.md` — Board IA + agent sketch  
- `architecture-prototype.md` §6 — earlier tool list + confirm gates  
- `02-stack-blueprint.md` — Phase 2 MCP  
- `00-session-brief.md` — viral angles  

---

## Appendix — Competitive snapshot (sourced claims only)

| Vendor | Hosted MCP | Transport (docs) | Auth pattern | Tool count (sourced) | Cold-email fit |
|--------|------------|------------------|--------------|----------------------|----------------|
| **Instantly** | `mcp.instantly.ai/mcp` | Remote HTTP (client config `url`) | Header API key (preferred) | Official: full V2 API; blogs: 38 or 31 | High — campaigns, Unibox, analytics |
| **Smartlead** | `mcp.smartlead.ai/sse` | **SSE only** (help center) | API key in **URL query** | Vendor blog: **116+**; help center: examples only | High — deliverability diagnostics emphasized |
| **Resend** | `mcp.resend.com/mcp` | Streamable HTTP + local stdio/HTTP | OAuth or Bearer | Changelog: **10 tool groups**; Cursor page: **85+** tools | Medium for cold — strong for transactional + Cursor DX |
| **Us (proposed)** | `/api/mcp` | Streamable HTTP | Workspace MCP key (scoped) + user JWT | **Small intentional set** (table §6) — not an API mirror | **Issue / Memory / gated send** |

---

*End of `07-mcp-agent-ux.md`.*
