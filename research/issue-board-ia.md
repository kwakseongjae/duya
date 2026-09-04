# Cold Outreach — Issue Board IA & Core Screens

Date: 2026-09-04 · Audience: indie founders / small agencies

## Metaphor

Treat each outreach **purpose** as an **Issue** (Linear/GitHub style), not only as a blast campaign.

- Issue = “왜 이 메일을 보내는가” (목적 단위)
- Touches = 그 이슈 안의 개별 발송/리마인드
- Board = 목적들의 상태 흐름

Campaign/Sequence는 Issue 안의 **자동화 규칙**으로 두고, 사용자가 매일 보는 1급 객체는 Issue.

## Primary nav

| Nav | Role |
|-----|------|
| **Board** | Issue 칸반 / 리스트. 홈 |
| **Inbox** | 회신·바운스·불만 통합 인박스 |
| **Templates** | 템플릿 라이브러리 + 프리뷰 |
| **Memory** | 누구에게 뭘 보냈고 뭐가 먹혔는지 |
| **Accounts** | Gmail / Resend 발송 계정 |
| **Settings** | 워크스페이스, 컴플라이언스, MCP 키 |

Secondary (Issue 상세 안): Timeline · Prospects · Sequence · Preview · Stats

## Core objects

```
Workspace
  └─ SendingAccount (gmail | resend)
  └─ Template (versions)
  └─ Issue (purpose cluster)
       ├─ status, tags, owner
       ├─ Sequence / reminder rules
       ├─ Prospect[] (to / company / status)
       ├─ Touch / Message[] (from account, template_version, provider ids)
       └─ Event[] (sent, delivered, open?, click, reply, bounce, reminder)
  └─ MemoryIndex (embeddings / winners / suppressions)
```

### Issue statuses (board columns)

1. **Draft** — 목적·리스트·템플릿 준비 중  
2. **Ready** — 발송 승인 대기 (HITL)  
3. **Sending** — 예약/발송 중  
4. **Waiting** — 회신 대기 (리마인드 카운트다운)  
5. **Replied** — 사람 회신 있음 (긍정/중립/부정 태그)  
6. **Snoozed** — 일시 중지  
7. **Closed** — 완료 / 포기 / 구독해지

Optional swimlanes: owner, ICP tag, provider mix.

## Key screens (MVP)

### 1. Board
- Columns by status; cards show title, #prospects, reply rate, next reminder, sending accounts chip
- Filters: account, tag, owner, date
- Quick create Issue

### 2. Issue detail (hub)
- Header: title, status, purpose one-liner
- Left: Prospect table (who / last touch / opens? / reply)
- Center: Timeline (touches + events)
- Right: Sequence & reminder rules; Send-from accounts
- Top actions: Approve send · Pause · Close · Open preview

### 3. Compose / Preview
- Split: Desktop email chrome | Mobile preview
- Template variables filled from prospect
- From account picker (Gmail vs Resend)
- Send now / Schedule / Save as draft touch

### 4. Templates
- Grid of templates; performance badges (reply rate when n≥threshold)
- Editor + desktop/mobile preview
- Version history

### 5. Memory
- Search person/company → past Issues & touches
- “Winners” for segment (subject/body snippets)
- Suppression / do-not-contact

### 6. Accounts
- Connected Gmail OAuth + Resend API domains
- Daily cap, warmup note, SPF/DKIM/DMARC checklist
- Per-account health (bounce/complaint)

### 7. Inbox
- Replies threaded to Issues; one-click move status → Replied
- Bounce/complaint auto-pause cues

## Agent / MCP surface (maps to screens)

| Tool (sketch) | Maps to |
|---------------|---------|
| `list_issues` / `get_issue` | Board / Detail |
| `draft_touch` | Compose (draft only) |
| `schedule_send` | Ready→Sending (confirm gate) |
| `get_memory(person)` | Memory |
| `get_stats(issue)` | Issue stats |
| `list_templates` | Templates |

Default agent mode: **draft + propose**, human Approve on Board/Ready.

## Non-goals for first IA

- LinkedIn/call multichannel columns
- Unlimited mailbox farm ops UI (Instantly clone)
- Open-rate leaderboards as primary

## Success of this IA

Founder can answer in <10s: “이 목적(Issue)으로, 어느 계정에서, 누구에게, 뭐 보냈고, 다음에 리마인드가 언제며, 회신은?”

## Screen notes — Memory & Inbox (added)

### Memory
- Goal: answer “이 사람/회사에 뭘 보냈고, 뭐가 먹혔나?” in one place; MCP `get_memory(person)` surface.
- Layout: search + result list | person detail (past Issues, touches with provider chips, Winners snippets, suppression).
- Winners only show reply-rate badges when sample size ≥ threshold (avoid vanity opens).
- Do-not-contact / suppression is first-class and blocks Approve send.

### Inbox
- Goal: operational triage for replies, bounces, complaints — not a full email client.
- Filters: All · Replies · Bounces · Complaints.
- List → thread linked to Issue; actions: Mark replied (moves Issue → Replied), Pause sequence, Open issue.
- Bounce/complaint rows cue auto-pause / list hygiene.

## Screen notes — Accounts & Templates (added)

### Accounts
- Connect Gmail (OAuth) and Resend (API + verified domain).
- Per account: daily/hourly cap, health (bounce/complaint), SPF/DKIM/DMARC checklist.
- Resend: tracking subdomain toggle (default off for cold 1:1 unless opted in).
- Used by Issue Send-from picker and Compose From dropdown.

### Templates
- Library grid with reply-rate badges only when n ≥ threshold.
- Editor + version history; desktop/mobile preview thumbnails.
- Variables `{{first_name}}` etc.; Issue compose pulls a template version (immutable on send).
