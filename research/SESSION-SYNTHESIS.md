# Cold Outreach Research Session — Synthesis

**Session date:** 2026-09-04 (Asia/Seoul)  
**Mode:** Research-only · implementation deferred · no credentials  
**Closed:** after 23:30 KST (routine self-deleted)

---

## One-line thesis

인프라(웜업·무제한 인박스)로 Instantly/Smartlead와 싸우지 말고, **목적(Issue) 단위 인지 + Memory + Gmail/Resend 원장 + MCP HITL**로 인디/에이전시 웨지를 잡는다. 단기: **무료 워크스페이스**, Instantly식 과금 SKU는 목표가 아님.

---

## Locked product bets

| Area | Decision |
|------|----------|
| Metaphor | Issue = purpose; Board Draft→Ready→Sending→Waiting→Replied→Snoozed/Closed |
| Surface | Web SaaS first (desktop later) |
| Stack | Next.js App Router · Vercel · Supabase (Auth Google OAuth + Postgres RLS) · shadcn · next-intl (ko/en) |
| Send Phase 0 | Resend = permissioned / transactional / verified-domain only |
| Send Phase 1 / cold | **Gmail OAuth** (Resend AUP blocks true cold/unsolicited) |
| Jobs | Inngest (+ thin Vercel Cron heartbeat) |
| Agents | MCP Phase 2; draft-only until Board Approve (HITL) |
| Monetization (now) | Free / OSS-first — no paid plans |
| Coding | Prefer Grok Build; else local Cursor CLI (**not** cloud agents) |

---

## What each doc covers

| File | Angle |
|------|--------|
| `00-session-brief.md` | Session brief, locked bets, early auth/i18n/UI notes |
| `01-market-viral-conventions.md` | Ops conventions, viral GTM, pricing norms, risks, differentiation |
| `02-stack-blueprint.md` | Fullstack blueprint (repo layout, Auth, Resend, Inngest, Gmail, MCP) |
| `03-ui-shadcn-i18n.md` | shadcn + Linear density + next-intl ko/en UI plan |
| `04-master-roadmap.md` | Integrated phases (Phase 0 demo → Gmail → MCP) |
| `05-pricing-packaging-gtm.md` | Competitor packaging teardown, SKU copy, launch calendar |
| `06-deliverability-compliance.md` | SPF/DKIM/DMARC, unsub, bounce, Resend AUP blocker, legal UX |
| `07-mcp-agent-ux.md` | Instantly/Smartlead/Resend MCP, Issue-shaped tools, HITL gates |
| `08-agent-app-mail-tracking.md` | How Codex/Cursor/Claude send+track; ledger gaps |
| `09-direction-agent-mail-free.md` | Direction v2: free agent→Board→Memory OS |
| `10-email-preview-html-css-pitfalls.md` | Client engines, clip/size, sandbox/CSP preview rules |
| `issue-board-ia.md` | Primary IA (Board/Inbox/Templates/Memory/Accounts) |
| `architecture-prototype.md` | Earlier architecture research (adapters, tracking, replies) |
| `market-competitors.md` | Market size signals, competitor landscape, whitespace |

---

## Highest-priority gates before public send

1. **DNS wizard** (SPF+DKIM+DMARC) fail-closed before first public send  
2. **One-click List-Unsubscribe** + bounce/complaint suppression  
3. **Daily/hourly caps + warm schedule** baked into product defaults  
4. **Resend ≠ cold path** — plan Gmail (or alternate ESP) for true cold  
5. **Agent cannot send without human Approve** (server-side, not host UI alone)  
6. **Preview honesty** — iframe ≠ Outlook/Gmail; size meter + plain-text toggle  

---

## Differentiation (vs Instantly / Smartlead / Lemlist)

- Competitors: campaign/mailbox ops + warmup farms + modular paid SKUs  
- Us: **Issue-shaped MCP** (`get_memory`, `draft_touch`, gated `schedule_send`) + Board Approve + unified Gmail/Resend ledger + Memory  
- Viral clip: “Cursor에게 이 사람에게 뭐 보냈는지 물어보기” — not “API 엔드포인트 38개”

---

## Gaps still open (next research, if needed)

- Competitive teardown **refresh** (vendor pages move)  
- Agency **multi-workspace** conventions  
- next-intl **message catalog** design deep dive  
- Supabase **RLS policy examples** (concrete SQL)  
- Inngest **reminder patterns** (concrete function shapes)  
- X/Threads **hook bank** (copy-ready KO+EN)  
- Korean market **cold-email norms** (local compliance/culture)

---

## Index (all markdown under `/workspace/cold-email-research/`)

```
00-session-brief.md
01-market-viral-conventions.md
02-stack-blueprint.md
03-ui-shadcn-i18n.md
04-master-roadmap.md
05-pricing-packaging-gtm.md
06-deliverability-compliance.md
07-mcp-agent-ux.md
08-agent-app-mail-tracking.md
09-direction-agent-mail-free.md
10-email-preview-html-css-pitfalls.md
SESSION-SYNTHESIS.md          ← this file
architecture-prototype.md
issue-board-ia.md
market-competitors.md
```

---

*Session closed by routine `cold-outreach-research-waves` after 23:30 KST on 2026-09-04.*
