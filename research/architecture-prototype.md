# Cold-Email Tracking + Automation Product — Architecture Research (2026)

**Research date:** 2026-09-04 (Asia/Seoul)  
**Scope:** Multi-account sending (Gmail OAuth/API + Resend), campaign clustering, per-message tracking, templates, stats/memory, MCP server, AI drafting/sequencing  
**Method:** Primary docs via WebSearch + WebFetch; limits cited or marked uncertain. No fabricated API quotas.

---

## Executive recommendation (TL;DR)

**Build a Next.js SaaS MVP first**, with a provider-adapter layer for **Resend (Phase 0)** then **Gmail API (Phase 1)**. Defer Tauri/Electron desktop until you need local-token privacy or offline compose.

| Priority | Choice |
|---|---|
| Surface | **Web SaaS (Next.js App Router)** |
| Data | **Postgres (Supabase or Neon) + RLS** |
| Jobs | **Inngest or Trigger.dev** (webhooks, reminders, watch renewal) |
| Send path | Adapter: `GmailProvider` \| `ResendProvider` |
| Tracking | Own open/click endpoints + **confidence scoring** (MPP-aware); treat opens as weak signals |
| Replies | Gmail: Pub/Sub `users.watch` + `history.list`; Resend: inbound `email.received` on reply subdomain |
| Agent surface | **MCP Streamable HTTP** tools over the same domain model |
| First 1–2 weeks | Campaigns + Resend send + webhooks + templates + reminder jobs + basic stats |

Cold email is deliverability- and compliance-constrained. Prefer **Workspace mailboxes + warm domains + low volume** over blasting. Open rates alone are unreliable in 2026—optimize for **replies, clicks, and booked outcomes**.

SEE_DISK_FILE_FOR_FULL_CONTENT_PLACEHOLDER