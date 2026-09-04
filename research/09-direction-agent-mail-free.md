# Direction v2 — Mail connect · send automation · agent-app tracking (free)

**Date:** 2026-09-04 KST  
**Constraints:** No paid plans / monetization for now · Research synthesis · Implementation deferred  
**Builds on:** `04-master-roadmap.md`, `06-deliverability-compliance.md`, `07-mcp-agent-ux.md`, send-path lock (Gmail cold / Resend permissioned)

---

## 1. North star (updated)

**Codex / Cursor / Claude Code에서 “보내기”만 하는 게 아니라, Issue로 묶이고, 계정·수신자·회신·리마인드까지 한 장부에 남는 아웃리치 OS.**

인디 창업자가:
1. 코딩 에이전트(Codex류)에서 초안·발송 제안
2. 웹 Board에서 Approve / Pause / Close
3. Inbox·Memory에서 회신·중복·성과를 다시 에이전트에 넘김

을 **무료 워크스페이스**로 돌리는 게 단기 목표. Instantly식 웜업 팜·과금 SKU는 목표가 아님.

---

## 2. What agent apps do today (gap)

### 2.1 Typical stack (2026)
| Capability | Codex / Cursor / Claude today | Gap |
|------------|-------------------------------|-----|
| Send HTML/text | Resend MCP (`codex mcp add resend`, Cursor MCP JSON) | Easy send, weak purpose/Issue model |
| Domains / DNS | Resend MCP domain tools | No product UX for warmup/caps |
| List/get email | Resend MCP | No cross-provider ledger |
| Real-time open/bounce/reply | **Usually missing in-agent** — needs webhooks + your DB | Biggest gap |
| Thread / Issue clustering | None | Our wedge |
| HITL Approve | Chat confirm only | No Board Ready state |
| Memory “what did we tell X?” | Ad-hoc chat history | No durable Memory |
| Gmail cold send | Gmail connectors / separate OAuth | Not unified with Resend |
| Compliance gates | Weak | Must be product-enforced |

Sources: https://resend.com/changelog/mcp · https://resend.mintlify.dev/docs/mcp-server · Instantly/Smartlead MCP (sequencer APIs, not Issue boards)

### 2.2 Implication
**Don’t rebuild Resend MCP.** Wrap/complement it:
- Our MCP = Issue / Memory / Approve / Stats / Schedule-with-policy
- Provider MCP or adapter = raw send where allowed
- Web app = Unibox + Board + audit (agents can’t see webhooks alone)

---

## 3. Mail integration direction

### 3.1 Dual path (locked)
| Account type | Allowed Issue kinds | How connected |
|--------------|---------------------|---------------|
| **Gmail / Workspace** | Cold / unsolicited purpose Issues | OAuth (send first; readonly later for replies) |
| **Resend** | Permissioned / opted-in / transactional / product | API key + domain; hard “not for cold” gate |

### 3.2 Connect UX (Accounts)
- Connect Google (product login ≠ Gmail send — separate OAuth clients)
- Connect Resend (API key, domain verify checklist)
- Per account: daily cap, provider badge, health (bounce/complaint), freeze switch
- Issue Send-from picker only lists accounts **allowed for that Issue’s consent class**

### 3.3 Tracking pipeline (the product core)
```
Agent or UI draft_touch
  → policy check (consent class, suppression, cap, HITL)
  → provider.send (Gmail API | Resend API)
  → store touch + provider message ids + template_version
  → webhooks / PubSub / inbound Reply-To → events
  → Board/Inbox/Memory + MCP get_stats / get_memory
```

Must track (MVP): queued · sent · delivered · bounced · complained · reply · reminder_sent · cancelled  
Optional weak: open/click (off by default; never drive automation)

### 3.4 Automation (sequences) — conservative
- Issue-attached sequence: steps + wait business days + **stop on reply**
- Reminders only when status=Waiting and no reply event
- **Never** open-triggered next step
- Agent may `draft_touch` / propose schedule; **Approve** required for first cold send of an Issue (and optionally every step until user trusts)

---

## 4. Feature set for “Codex-like app send + track”

Prioritized for free agent-native users:

### P0 — Agent can trust the ledger
1. MCP: `list_issues`, `get_issue`, `draft_touch`, `schedule_send` (confirm), `get_memory(person)`, `get_stats(issue)`
2. Webhook/event store so delivery/bounce/reply appear without reopening chat
3. Board Approve / Pause / Close
4. Account policy (cold vs permissioned)
5. Suppression / do-not-contact

### P1 — Feels like a product not a wrapper
6. Inbox triage linked to Issues  
7. Templates + version pin + desktop/mobile preview  
8. Reminder automation  
9. Gmail reply sync  
10. Audit log of MCP + human actions  

### P2 — Nice for viral / power users
11. Worktree-friendly CLI (`npx` / hosted MCP URL like Resend)  
12. Skills/prompts pack for Codex (“open Issue, draft, wait for Approve”)  
13. Export Memory for other agents  
14. Optional local-first mode later  

### Explicit non-goals (now)
- Paid SKUs / usage billing  
- Unlimited mailbox warmup network  
- Autonomous AI SDR that sends without HITL  
- Competing with Instantly on volume infra  

---

## 5. Free positioning (no monetization)

| Free forever (near-term) | Why |
|--------------------------|-----|
| Personal / small team workspace | Founder dogfood + agent users |
| MCP endpoint + web Board | Distribution via Codex/Cursor |
| KO/EN UI | Audience on X/Threads KR+global |
| Open docs + prompt pack | Viral without paywall |

**Cost to user:** their own Gmail/Workspace + (optional) Resend account — we don’t sell email volume.

**Optional later (not now):** hosted fair-use limits only to protect infra; still no “Pro $89” push until you say otherwise.

**OSS wedge ideas (research, not committed):** MCP tool schemas + AGENTS.md skill open; hosted Board closed or open later.

---

## 6. Revised Phase 0 story (still research; when coding resumes)

1. Web Board + Auth (Google login for app)  
2. Issue/Prospect/Touch/Event schema  
3. **Gmail connect for cold** (priority raised vs old Resend-first cold)  
4. Resend connect for permissioned only  
5. MCP stub for Codex/Cursor (`codex mcp add …` / Cursor mcp.json)  
6. Webhooks → timeline  
7. Approve gate + Memory read  

Skip: pricing pages, Stripe, credit meters.

---

## 7. One-liner options (free / agent-native)

- EN: “Issue-tracked outreach for coding agents — send from Codex, approve on a board, remember every touch.”  
- KO: “코덱스에서 보내고, 보드에서 승인하고, 누구에게 뭘 보냈는지 기억하는 아웃리치.”  

Avoid: “cheaper Instantly”, “unlimited mailboxes”, “AI SDR that books meetings alone”.

---

## 8. Open questions (non-blocking)

1. Hosted MCP URL vs user self-host only while free  
2. How early to require Google verification for `gmail.readonly`  
3. Whether Phase 0 demo uses only permissioned Resend if Gmail OAuth not ready  


## 9. Addendum from `08-agent-app-mail-tracking.md`

- Four agent send patterns: ESP MCP · Gmail connector (often draft) · community Gmail MCP+HITL · AgentMail inboxes.
- Gmail cold path: optimize for **replies + threadId**, not opens/delivery webhooks.
- Free/OSS split: open Issue MCP schemas + Board UI + skills; keep OAuth vault, RLS, webhook ingest, abuse caps hosted.
