# duya

**Issue-tracked outreach for coding agents** — send from Codex/Cursor, approve on a board, remember every touch.

한국어: 코덱스·커서에서 보내고, 보드에서 승인하고, 누구에게 뭘 보냈는지 기억하는 **에이전트용 아웃리치 OS** 리서치·설계 저장소입니다.

> Status: **research & design only** (implementation deferred). **No paid plans** for now — users bring their own Gmail / Resend.

---

## Why duya?

Cold-email tools (Instantly, Smartlead, Lemlist) win on warmup farms and volume. Agent apps (Codex, Cursor, Claude Code) can already **send** via Resend MCP — but they lack:

- purpose clustering (**Issues**, not only campaigns)
- a durable **touch ledger** (which account → whom → what → reply?)
- **HITL Approve** on a board
- **Memory** agents can query (`get_memory(person)`)
- honest deliverability defaults (open tracking off, stop-on-reply, no open-triggered sequences)

**duya** aims at that wedge: MCP-first for agents + web Issue Board for humans. Not an Instantly clone. Not a Resend MCP clone.

---

## Locked product bets

| Area | Decision |
|------|----------|
| Primary object | **Issue** = outreach purpose (Linear/GitHub metaphor) |
| Surfaces | Board · Inbox · Templates · Memory · Accounts · Settings |
| Cold send | **Gmail / Google Workspace** OAuth |
| Permissioned / transactional | **Resend** only (Resend AUP prohibits cold/unsolicited) |
| App auth | Supabase Auth + Google OAuth (`@supabase/ssr`) + RLS |
| Hosting | Next.js App Router on Vercel · Inngest jobs |
| UI | shadcn (new-york / Radix) · next-intl `/ko` `/en` |
| Agents | MCP: `draft_touch`, `schedule_send`+confirm, `get_memory`, `get_stats`… |
| Monetization | **None for now** |

Board columns: `Draft → Ready → Sending → Waiting → Replied → Snoozed/Closed`

---

## Docs map

Start here:

| Doc | What it is |
|-----|------------|
| [`research/09-direction-agent-mail-free.md`](./research/09-direction-agent-mail-free.md) | **Direction v2** — agent-native, free, mail paths |
| [`research/04-master-roadmap.md`](./research/04-master-roadmap.md) | Master roadmap + phase plan |
| [`research/issue-board-ia.md`](./research/issue-board-ia.md) | IA & screen notes |

Deep dives:

| Doc | Topic |
|-----|--------|
| [`research/01-market-viral-conventions.md`](./research/01-market-viral-conventions.md) | Market, ops conventions, viral GTM |
| [`research/market-competitors.md`](./research/market-competitors.md) | Competitor matrix |
| [`research/02-stack-blueprint.md`](./research/02-stack-blueprint.md) | Next · Supabase · Resend · Vercel · schema/RLS |
| [`research/architecture-prototype.md`](./research/architecture-prototype.md) | Provider architecture, SaaS vs desktop |
| [`research/03-ui-shadcn-i18n.md`](./research/03-ui-shadcn-i18n.md) | shadcn + ko/en sprint |
| [`research/05-pricing-packaging-gtm.md`](./research/05-pricing-packaging-gtm.md) | Pricing teardown (SKU ideas deferred — no paid plans now) |
| [`research/06-deliverability-compliance.md`](./research/06-deliverability-compliance.md) | DNS, caps, unsub, KR→US/EU notes, Resend AUP |
| [`research/07-mcp-agent-ux.md`](./research/07-mcp-agent-ux.md) | MCP tools, HITL, demo scripts |
| [`research/08-agent-app-mail-tracking.md`](./research/08-agent-app-mail-tracking.md) | How Codex/Cursor send & track today |
| [`research/10-email-preview-html-css-pitfalls.md`](./research/10-email-preview-html-css-pitfalls.md) | Email HTML/preview pitfalls |
| [`research/00-session-brief.md`](./research/00-session-brief.md) | Early session bets |

---

## Phase sketch (when coding starts)

**Phase 0** — free demo: Next shell + i18n · Supabase schema/RLS · Google app login · **Gmail cold connect** · Resend permissioned · Board/Issue · templates+preview · event ledger · MCP stub · Approve gate · Memory read  

**Phase 1** — Gmail reply sync · multi-account policy · suppression · auto-pause on complaint  

**Phase 2** — hosted MCP polish · winners Memory · optional desktop shell  

Explicit non-goals now: Stripe, unlimited inbox warmup network, autonomous send without HITL.

---

## One-liners

- EN: *Issue-tracked outreach for coding agents — send from Codex, approve on a board, remember every touch.*
- KO: *코덱스에서 보내고, 보드에서 승인하고, 누구에게 뭘 보냈는지 기억하는 아웃리치.*

---

## License

Research docs: MIT (see [`LICENSE`](./LICENSE)). Third-party product names belong to their owners. Compliance sections are **research notes**, not legal advice.

---

## Disclaimer

Cold email is constrained by Google/Microsoft sender rules, ESP AUPs (e.g. Resend), and local law (CAN-SPAM, GDPR/PECR, CASL, KR network act). Design assumes HITL, suppression, and provider-appropriate consent classes.
