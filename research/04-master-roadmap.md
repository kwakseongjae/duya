# Cold Outreach Issue Board — Master Roadmap (통합)

**Date:** 2026-09-04 KST  
**Inputs:** `01-market-viral-conventions.md` · `02-stack-blueprint.md` · `03-ui-shadcn-i18n.md` · `issue-board-ia.md` · `architecture-prototype.md`

---

## 1. Product thesis (한 줄)

인프라(웜업·무제한 인박스)로 Instantly/Smartlead와 싸우지 말고, **목적(Issue) 단위 인지 + Memory + Gmail/Resend 원장 + MCP HITL**로 인디/에이전시 웨지를 잡는다. 딜리버빌리티는 지루하지만 기본값을 제품에 굽는다.

---

## 2. Locked decisions

| Area | Decision |
|------|----------|
| Metaphor | Issue = purpose; Board columns Draft→Ready→Sending→Waiting→Replied→Snoozed/Closed |
| Surface | Web SaaS first (desktop shell later) |
| Framework | Next.js App Router on **Vercel** |
| DB/Auth | **Supabase** Postgres + RLS + **Google OAuth** via `@supabase/ssr` |
| Send Phase 0 | **Resend** (transactional) + webhooks + Reply-To inbound |
| Send Phase 1 | Gmail OAuth (separate client; restricted scopes → verification/CASA) |
| Jobs | **Inngest** (+ thin Vercel Cron heartbeat) |
| UI | **shadcn** new-york / Radix / neutral · Linear density · dark mode |
| i18n | **next-intl** · `/ko` · `/en` |
| Agents | MCP Phase 2; draft-only until Approve |
| Coding | Prefer Grok Build; else local Cursor CLI (not cloud) |

---

## 3. Phase plan (solo founder + agent)

### Phase 0 — ~10–12 days (shippable demo)
1. Scaffold `apps/web` + shadcn + next-intl shell (ko/en)
2. Supabase project: migrations (workspaces → issues → touches → events), RLS
3. Google login (product auth only) + middleware session refresh
4. Accounts: Resend API key + domain checklist UI (mock DNS verify OK)
5. Templates CRUD + version pin + desktop/mobile iframe preview
6. Board + Issue detail with seed data
7. Send path: Resend `emails.send` + schedule via Inngest
8. Webhooks: delivered/bounced/complained/opened(opt-in)/received → events timeline
9. No-reply reminder job; stop-on-reply when inbound matches
10. Inbox triage (Replies/Bounces/Complaints) stub
11. Memory read-only list from past touches
12. Landing + waitlist; deploy Vercel preview

**Ops defaults baked in:** daily cap 20–30, open tracking **off**, DMARC wizard `p=none`, agent cannot send without Approve.

### Phase 1 — ~2–4 weeks
- Gmail send OAuth (`gmail.send` first)
- Reply detection (Pub/Sub watch + history) when readonly scopes verified
- Multi-account Send-from on Issue
- Suppression / do-not-contact enforcement on Approve
- Agency multi-workspace polish
- Postmaster / bounce auto-pause

### Phase 2 — ongoing
- MCP Streamable HTTP (`list_issues`, `draft_touch`, `get_memory`, `schedule_send`+confirm)
- Winners embeddings / what-worked Memory
- Open confidence scoring (MPP-aware)
- Optional Tauri shell
- Better Auth only if leaving Supabase Auth

---

## 4. UI sprint (from 03) — can run parallel to Phase 0 days 1–7

| Day | Focus |
|-----|-------|
| 1 | App shell, Sidebar, next-intl, auth pages |
| 2 | Board kanban (dnd-kit) |
| 3 | Issue detail hub |
| 4 | Compose + Templates + preview iframes |
| 5 | Inbox + Memory |
| 6 | Accounts health + DNS checklist |
| 7 | QA a11y, empty states, ko/en parity |

---

## 5. Data model (MVP tables)

`workspaces` · `workspace_members` · `sending_accounts` · `templates` · `template_versions` · `issues` · `prospects` · `touches` · `events` · `suppressions`

RLS: `private.is_workspace_member(workspace_id)` · all tenant tables filtered by membership.  
Service role only in server admin client / webhook workers.

---

## 6. Viral / GTM sequence

1. Issue board demo GIF (metaphor)
2. Memory anti-duplicate clip
3. MCP + Approve clip
4. Deliverability checklist thread (cite Google docs)
5. Template swipe file by Issue type
6. Before/after **reply** rates with transparent n — never open-rate vanity

Packaging feel: Indie $29–49 · Pro $79–99 · Agency workspaces · AI credits optional.

---

## 7. Success metrics (MVP)

- Positive reply rate / Issue  
- % Issues → Replied or Closed with outcome  
- Duplicate-send rate ↓ (Memory)  
- Time-to-first-approved-send  
- Bounce / complaint health  
- MCP draft → approved send conversion  

---

## 8. Doc index

| File | Role |
|------|------|
| `00-session-brief.md` | Session bets |
| `01-market-viral-conventions.md` | Market, ops, viral, pricing |
| `02-stack-blueprint.md` | Auth, schema, Resend, Vercel, phases |
| `03-ui-shadcn-i18n.md` | shadcn + next-intl sprint |
| `issue-board-ia.md` | Screens & objects |
| `architecture-prototype.md` | Earlier provider/arch research |
| `market-competitors.md` | Competitor matrix |
| `04-master-roadmap.md` | **This file** |

---

## 9. Immediate next implementation step

When coding resumes (Cursor CLI login or Grok Build):

```text
1. create next-app apps/web (or /workspace/cold-outreach)
2. shadcn init (new-york, Radix, neutral)
3. next-intl [locale] shell
4. supabase migrations from 02-stack-blueprint schema
5. Google OAuth login page
6. Board page with seed Issues
```


## 10. Blocking note — Resend AUP vs cold email (2026-09-04)

Research in `06-deliverability-compliance.md`: Resend Acceptable Use (dated ~2026-08-27 in sources) **prohibits cold / unsolicited outreach**.

**Implication:** Do not market Phase 0 as “send cold via Resend.” Options:
1. **Gmail/Workspace connected mailbox = cold path** (bring earlier than planned); Resend = transactional / opted-in only
2. Resend path gated as **permissioned audiences only** with hard UX copy + audit
3. Swap cold ESP / SMTP mailbox infra for cold Issues

Update Phase 0 send story before coding starts.

## 11. Send-path decision (default, 2026-09-04)

User skipped the choice widget. Default locked from compliance research:

| Path | Use for |
|------|---------|
| **Gmail / Google Workspace OAuth** | Cold / unsolicited Issue sends (primary cold path) — pull earlier in Phase 0/1 |
| **Resend** | Permissioned / opted-in / transactional / product mail only; hard UX gate + audit |
| Other ESP | Optional later if Gmail caps block agency volume |

Phase 0 demo can still use Resend **seed/permissioned** prospects with clear “not for cold” labeling, while Gmail cold path is prioritized in the same milestone band.

## 12. Pricing/GTM snapshot (from 05)

- Indie ~$39 · Pro ~$89 · Agency ~$199 (workspace-based; don’t compete on unlimited inboxes)
- Avoid Instantly-style credit surprise; optional AI credits only
- Agency: include ~5 client workspaces transparently
- GTM: Issue metaphor → Memory → MCP Approve → deliverability honesty

## 13. MCP snapshot (from 07)

- Tools: `list_issues`, `get_issue`, `draft_touch`, `get_memory`, `list_templates`, `get_stats`, `schedule_send` (confirm/HITL required)
- Default agent mode = draft; Board Ready → human Approve
- Audit every MCP mutation; never open-triggered auto-send

## 14. Direction v2 (2026-09-04 evening) — free + agent-native

See `09-direction-agent-mail-free.md`.

- **No monetization / paid SKUs for now** (user stated).
- Product wedge = **Codex/Cursor-class agents** send via MCP + **Issue Board** HITL + durable tracking (not Resend MCP clone, not Instantly clone).
- Mail: Gmail = cold; Resend = permissioned only.
- Phase 0 raises Gmail connect priority; drop Stripe/pricing work.
- Tracking pipeline (events ledger) is the core differentiator vs raw provider MCP.
