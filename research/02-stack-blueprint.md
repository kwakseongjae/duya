# Fullstack Implementation Blueprint

**Product:** Cold outreach Issue Board (see `issue-board-ia.md`)  
**Stack:** Next.js App Router + Vercel + Supabase (Auth Google OAuth, Postgres RLS, optional Edge Functions) + Resend + Google login + MCP later  
**Research date:** 2026-09-04 (Asia/Seoul)  
**Method:** Primary docs via WebFetch/WebSearch; limits cited or marked **uncertain**. No fabricated API quotas.

Related: `architecture-prototype.md`, `issue-board-ia.md`, `market-competitors.md`

---

## TL;DR decisions

| Decision | Choice | Why |
|---|---|---|
| Auth (MVP) | **Supabase Auth + Google OAuth** via `@supabase/ssr` | Native `auth.uid()` → Postgres RLS; one platform for Auth+DB; SSR cookies documented for App Router |
| Not Better Auth (MVP) | Defer | Better Auth shines when authz lives in TypeScript; with Supabase Postgres + RLS as the tenancy boundary, Supabase Auth is the smaller coherent choice |
| Send (Phase 0) | **Resend** transactional API | Fast path; webhooks + inbound GA; no Google restricted-scope verification |
| Jobs | **Inngest** on Vercel (primary); Vercel Cron only for thin heartbeats | Durable steps, retries, cron with TZ; avoids Hobby cron frequency limits for reminder fan-out |
| Gmail | Phase 1 | Scope verification / CASA for restricted scopes — weeks, not days |
| MCP | Phase 2 | Same domain model + RLS; Streamable HTTP later |
| Edge Functions | Optional | Prefer Next.js Route Handlers on Vercel for webhooks/cron; use Supabase Edge only if you want Deno co-located with DB secrets |

---

## 1. Recommended repo structure (`apps/web`)

Monorepo-ready layout.

Start with `apps/web` only; add `packages/*` later.

Suggested layout (indent tree):

```text
repo/
  apps/
    web/                                 # Next.js App Router (Vercel)
      app/
        (marketing)/                     # public landing
        (auth)/
          login/page.tsx
          auth/callback/route.ts         # PKCE code exchange
        (app)/                           # authenticated shell
          layout.tsx                     # Board Inbox Templates Memory Accounts Settings
          board/page.tsx
          issues/[id]/page.tsx
          inbox/page.tsx
          templates/
          memory/page.tsx
          accounts/page.tsx
          settings/page.tsx
        api/
          webhooks/resend/route.ts
          inngest/route.ts
          cron/heartbeat/route.ts
          mcp/route.ts
        layout.tsx
        middleware.ts
      components/
      lib/
        supabase/
          client.ts
          server.ts
          middleware.ts
          admin.ts
        resend/
          client.ts
          verify-webhooks.ts
        providers/
        inngest/
          client.ts
          functions/
        domain/
      emails/
      package.json
      vercel.json
  packages/
    db/
    mcp-tools/
    tsconfig/
  supabase/
    config.toml
    migrations/
    tests/
    functions/
  README.md
```

**Notes**

- Keep provider adapters (`lib/providers/*`) behind sendTouch()/handleInbound().
- Privileged admin DB client only in `lib/supabase/admin.ts` (server only).
- Align routes with IA (`issue-board-ia.md`).
- Resend ingest route: `app/api/` + provider hook path (section 4).

---
## 2. Auth: pick + concrete setup

### 2.1 Pick for MVP: Supabase Auth (Google) — not Better Auth

**Rationale (Next.js App Router + SSR cookies + this product)**

1. **RLS is the tenancy model.** Workspace membership policies need `auth.uid()` from a Supabase-minted JWT. Better Auth on Supabase Postgres requires compatible JWTs or abandoning RLS — extra glue for MVP.
   Cite: https://www.iloveblogs.blog/post/better-auth-vs-supabase-auth-2026 and https://makerkit.dev/blog/tutorials/better-auth-vs-clerk
2. **SSR cookies are first-class.** Official path is `@supabase/ssr` with browser + server clients and middleware/proxy token refresh.
   Cite: https://supabase.com/docs/guides/auth/server-side/nextjs
3. **Google login is built-in.** Provider config + PKCE callback documented for App Router.
   Cite: https://supabase.com/docs/guides/auth/social-login/auth-google
4. **Better Auth** remains strong later if authz moves to TypeScript or you leave Supabase Auth billing — not MVP.

**MVP Google login = product sign-in only** (openid/email/profile). Gmail send/read OAuth is a separate Google Cloud client + scopes (Phase 1).

### 2.2 Packages

```bash
pnpm add @supabase/supabase-js @supabase/ssr
```

Source: https://supabase.com/docs/guides/auth/server-side/nextjs

### 2.3 Environment variables

```bash
# Public
NEXT_PUBLIC_SUPABASE_URL=https://<project-ref>.supabase.co
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY=sb_publishable_...
# older guides: NEXT_PUBLIC_SUPABASE_ANON_KEY=...

# Server-only — NEVER NEXT_PUBLIC_
SUPABASE_SERVICE_ROLE_KEY=...

NEXT_PUBLIC_SITE_URL=http://localhost:3000
# Google client id/secret live in Supabase Dashboard Auth Google provider
```

Sources: SSR env docs + https://supabase.com/docs/guides/api/api-keys

### 2.4 Redirect URLs

**Google Cloud — OAuth Web client** (https://supabase.com/docs/guides/auth/social-login/auth-google):

| Field | Value |
|---|---|
| Authorized JavaScript origins | `http://localhost:3000`, `https://app.yourdomain.com` |
| Authorized redirect URIs | `https://<project-ref>.supabase.co/auth/v1/callback`; local: `http://127.0.0.1:54321/auth/v1/callback` |

**Supabase Auth URL config:**

| Field | Value |
|---|---|
| Site URL | Production app origin |
| Redirect allowlist | `http://localhost:3000/auth/callback`, `https://app.yourdomain.com/auth/callback`, optional `https://*-yourteam.vercel.app/auth/callback` |

App callback at `app/auth/callback/route.ts`: Google -> Supabase -> your `/auth/callback?code=`.

### 2.5 Google provider setup (product login)

1. Create OAuth client (Web) in Google Auth Platform / Cloud Console.
2. Scopes: `openid`, userinfo.email, userinfo.profile — avoid sensitive/restricted scopes here.
3. Paste Client ID + Secret into Supabase → Authentication → Providers → Google.
4. Optional: custom Auth domain so consent shows auth.yourdomain.com not *.supabase.co.

### 2.6 Session patterns (@supabase/ssr)

Three contexts: browser client, server client, middleware/proxy refresh.

- Browser — `createBrowserClient`
- Server — `createServerClient` with cookies getAll/setAll from next/headers
- Middleware — refresh session; apply cache headers so CDNs do not leak sessions

Cite: https://supabase.com/docs/guides/auth/server-side/nextjs

| Method | Use |
|---|---|
| getClaims() | Protect pages/data — validates JWT |
| getUser() | Fresh user record (network) |
| getSession() | Raw tokens only — do not trust embedded user for authz |

Never trust getSession() alone in middleware for authorization.

OAuth sign-in (PKCE):

```ts
await supabase.auth.signInWithOAuth({
  provider: "google",
  options: { redirectTo: `${origin}/auth/callback?next=/board` },
})
```

Callback (from Supabase auth-google Next.js sample):

```ts
export async function GET(request: Request) {
  const { searchParams, origin } = new URL(request.url)
  const code = searchParams.get("code")
  let next = searchParams.get("next") ?? "/board"
  if (!next.startsWith("/")) next = "/board" // open-redirect guard
  if (code) {
    const supabase = await createClient()
    const { error } = await supabase.auth.exchangeCodeForSession(code)
    if (!error) {
      const forwardedHost = request.headers.get("x-forwarded-host")
      if (process.env.NODE_ENV === "development")
        return NextResponse.redirect(`${origin}${next}`)
      if (forwardedHost)
        return NextResponse.redirect(`https://${forwardedHost}${next}`)
      return NextResponse.redirect(`${origin}${next}`)
    }
  }
  return NextResponse.redirect(`${origin}/auth/auth-code-error`)
}
```

**Cookie write pitfall:** supabase-js >= 2.91.0 deferred SIGNED_IN in exchangeCodeForSession; cookies may not persist in serverless before response ends. See https://github.com/supabase/supabase-js/issues/2037 — pin versions or await microtask; retest on Vercel after upgrades.

Provider tokens (optional): queryParams access_type=offline + prompt=consent. MVP product login usually skips this; Gmail connect is separate.

---
## 3. Schema sketch (Issue Board IA) + RLS outline

Aligned to issue-board-ia.md: Workspace -> SendingAccount, Template(+versions), Issue -> Prospects, Touches, Events; plus Suppressions / Memory.

### 3.1 Enums (sketch)

```sql
create type sending_provider as enum ('resend', 'gmail');
create type issue_status as enum (
  'draft', 'ready', 'sending', 'waiting', 'replied', 'snoozed', 'closed'
);
create type prospect_status as enum (
  'pending', 'queued', 'sent', 'delivered', 'replied', 'bounced', 'suppressed', 'skipped'
);
create type touch_status as enum (
  'draft', 'scheduled', 'sending', 'sent', 'failed', 'canceled'
);
create type event_type as enum (
  'sent', 'delivered', 'delivery_delayed', 'opened', 'clicked',
  'bounced', 'complained', 'failed', 'suppressed', 'received',
  'reminder', 'status_change', 'note'
);
create type member_role as enum ('owner', 'admin', 'member');
```


### 3.2 Tables (sketch)

```sql
create table workspaces (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  slug text unique not null,
  created_at timestamptz not null default now()
);

create table workspace_members (
  workspace_id uuid not null references workspaces(id) on delete cascade,
  user_id uuid not null references auth.users(id) on delete cascade,
  role member_role not null default 'member',
  created_at timestamptz not null default now(),
  primary key (workspace_id, user_id)
);

create table sending_accounts (
  id uuid primary key default gen_random_uuid(),
  workspace_id uuid not null references workspaces(id) on delete cascade,
  provider sending_provider not null,
  label text not null,
  from_email text not null,
  config jsonb not null default '{}',
  daily_cap int,
  hourly_cap int,
  tracking_enabled boolean not null default false,
  health jsonb not null default '{}',
  created_at timestamptz not null default now()
);

create table templates (
  id uuid primary key default gen_random_uuid(),
  workspace_id uuid not null references workspaces(id) on delete cascade,
  name text not null,
  created_at timestamptz not null default now()
);

create table template_versions (
  id uuid primary key default gen_random_uuid(),
  template_id uuid not null references templates(id) on delete cascade,
  version int not null,
  subject text not null,
  body_html text,
  body_text text,
  variables jsonb not null default '[]',
  created_by uuid references auth.users(id),
  created_at timestamptz not null default now(),
  unique (template_id, version)
);

create table issues (
  id uuid primary key default gen_random_uuid(),
  workspace_id uuid not null references workspaces(id) on delete cascade,
  title text not null,
  purpose text,
  status issue_status not null default 'draft',
  owner_id uuid references auth.users(id),
  tags text[] not null default '{}',
  sequence_rules jsonb not null default '{}',
  default_account_id uuid references sending_accounts(id),
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);

create table prospects (
  id uuid primary key default gen_random_uuid(),
  workspace_id uuid not null references workspaces(id) on delete cascade,
  issue_id uuid not null references issues(id) on delete cascade,
  email text not null,
  first_name text,
  last_name text,
  company text,
  status prospect_status not null default 'pending',
  meta jsonb not null default '{}',
  created_at timestamptz not null default now(),
  unique (issue_id, email)
);
```

```sql
create table touches (
  id uuid primary key default gen_random_uuid(),
  workspace_id uuid not null references workspaces(id) on delete cascade,
  issue_id uuid not null references issues(id) on delete cascade,
  prospect_id uuid not null references prospects(id) on delete cascade,
  account_id uuid not null references sending_accounts(id),
  template_version_id uuid references template_versions(id),
  step_index int not null default 0,
  status touch_status not null default 'draft',
  subject text not null,
  body_html text,
  body_text text,
  scheduled_at timestamptz,
  sent_at timestamptz,
  provider_message_id text,
  rfc_message_id text,
  thread_key text,
  reply_to text,
  idempotency_key text unique,
  created_at timestamptz not null default now()
);

create table events (
  id uuid primary key default gen_random_uuid(),
  workspace_id uuid not null references workspaces(id) on delete cascade,
  issue_id uuid references issues(id) on delete set null,
  prospect_id uuid references prospects(id) on delete set null,
  touch_id uuid references touches(id) on delete set null,
  type event_type not null,
  provider text,
  provider_event_id text,
  payload jsonb not null default '{}',
  occurred_at timestamptz not null default now(),
  created_at timestamptz not null default now()
);
create unique index events_provider_event_uniq
  on events (provider, provider_event_id)
  where provider_event_id is not null;

create table suppressions (
  id uuid primary key default gen_random_uuid(),
  workspace_id uuid not null references workspaces(id) on delete cascade,
  email citext not null,
  reason text not null,
  source_event_id uuid references events(id),
  created_at timestamptz not null default now(),
  unique (workspace_id, email)
);
```

Indexes: workspace_members(user_id), all workspace_id FKs, touches(provider_message_id), prospects(email), issues(workspace_id, status).

### 3.3 RLS outline

Follow Supabase guidance: enable RLS, revoke default grants from anon/authenticated, grant back only needed ops, policy per operation, wrap auth.uid() in (select auth.uid()), test with `supabase test db`.
Cite: https://supabase.com/docs/guides/database/postgres/row-level-security

Helper (security definer, private schema — avoid recursive policies):

```sql
create schema if not exists private;

create or replace function private.is_workspace_member(ws uuid)
returns boolean
language sql
stable
security definer
set search_path = ''
as $$
  select exists (
    select 1 from public.workspace_members m
    where m.workspace_id = ws
      and m.user_id = (select auth.uid())
  );
$$;

revoke all on function private.is_workspace_member(uuid) from public;
grant execute on function private.is_workspace_member(uuid) to authenticated;
```

Pattern for every tenant table (issues, prospects, touches, events, templates, sending_accounts, suppressions):

```sql
alter table public.issues enable row level security;
revoke all on table public.issues from anon, authenticated;
grant select, insert, update, delete on table public.issues to authenticated;

create policy "members select issues"
  on public.issues for select to authenticated
  using ( private.is_workspace_member(workspace_id) );

create policy "members insert issues"
  on public.issues for insert to authenticated
  with check ( private.is_workspace_member(workspace_id) );

create policy "members update issues"
  on public.issues for update to authenticated
  using ( private.is_workspace_member(workspace_id) )
  with check ( private.is_workspace_member(workspace_id) );

create policy "members delete issues"
  on public.issues for delete to authenticated
  using ( private.is_workspace_member(workspace_id) );
```

Tighten later: owner/admin-only delete; vault/encrypt sending_accounts.config secrets (readable only via privileged send path).

Webhook / job writes: Route Handlers and Inngest use privileged admin client to insert events / update touches / write suppressions (bypasses RLS). Still scope updates by workspace_id resolved from the touch/account — never trust webhook body alone for tenancy.

anon: no grants on product tables.

---
## 4. Resend

Sources:
- Send: https://resend.com/docs/api-reference/emails/send-email
- Webhooks: https://resend.com/docs/dashboard/webhooks/introduction
- Verify: https://resend.com/docs/dashboard/webhooks/verify-webhooks-requests
- Event types: https://resend.com/docs/webhooks/event-types
- email.received: https://resend.com/docs/webhooks/emails/received
- Custom receiving domains: https://resend.com/docs/dashboard/receiving/custom-domains
- Add domain: https://resend.com/docs/add-a-domain
- Manage domains / SPF-DKIM: https://resend.com/docs/dashboard/domains/manage-domains
- Reply threading: https://resend.com/docs/dashboard/receiving/reply-to-emails
- Quotas: https://resend.com/docs/knowledge-base/account-quotas-and-limits
- Rate limit: https://resend.com/docs/api-reference/rate-limit
(Also summarized in architecture-prototype.md)

### 4.1 Send API (MVP)

```ts
import { Resend } from 'resend'
const resend = new Resend(process.env.RESEND_API_KEY)
const { data, error } = await resend.emails.send({
  from: 'Acme <outreach@send.yourdomain.com>',
  to: [prospect.email],
  subject,
  html,
  text,
  reply_to: [`replies+${touchId}@inbound.yourdomain.com`],
  tags: [
    { name: 'touch_id', value: touchId.replace(/-/g, '') },
  ],
})
// persist data.id -> touches.provider_message_id
```

Documented fields include from, to (max 50 addresses), subject, html/text/react, reply_to, headers, scheduled_at, tags, attachments (max 40MB per email after Base64), idempotency key (unique per request, 24h expiry, max 256 chars).

**Cited limits (re-verify on dashboard before shipping):**

| Control | Documented value | Source |
|---|---|---|
| Default API rate | 10 req/s per team (keys share pool) | Rate limit docs |
| Batch | Up to 100 emails per batch call | Rate limit / API docs |
| Free transactional | 100/day, 3,000/month (sent+received count) | Account quotas |
| Bounce / spam gates | Keep bounce < 4%, spam < 0.08% or sending may pause | Account quotas |
| Data retention | 30 days — store webhook payloads yourself | Account quotas |

If a limit is unclear at implement time, mark uncertain and check live docs — do not invent numbers.


### 4.2 Webhooks to subscribe (MVP)

Endpoint: POST /api/webhooks/resend

| Event | Product action |
|---|---|
| email.sent | Confirm accepted; timeline |
| email.delivered | Prospect/touch -> delivered; start reminder clock |
| email.delivery_delayed | Soft signal |
| email.bounced | Suppression + pause prospect/sequence |
| email.complained | Suppression + pause; Inbox cue |
| email.opened / email.clicked | Weak/strong engagement (tracking must be on) |
| email.failed / email.suppressed | Mark failed; hygiene |
| email.received | Inbound reply -> Inbox + Issue -> Replied |

Full list: https://resend.com/docs/webhooks/event-types

Semantics: email.sent != delivered; opens/clicks need tracking enabled.

Delivery: at-least-once; dedupe with svix-id; order not guaranteed — sort by created_at.
Retries if non-200: 5s -> 5m -> 30m -> 2h -> 5h -> 10h.
Cite: https://resend.com/docs/dashboard/webhooks/introduction

Verify signature — raw body required (https://resend.com/docs/dashboard/webhooks/verify-webhooks-requests):

```ts
export async function POST(req: NextRequest) {
  const payload = await req.text()
  try {
    const event = resend.webhooks.verify({
      payload,
      headers: {
        id: req.headers.get('svix-id')!,
        timestamp: req.headers.get('svix-timestamp')!,
        signature: req.headers.get('svix-signature')!,
      },
      webhookSecret: process.env.RESEND_WEBHOOK_SECRET!,
    })
    // enqueue Inngest event or process idempotently
    return NextResponse.json({ ok: true })
  } catch {
    return new NextResponse('Invalid webhook', { status: 400 })
  }
}
```

Env: RESEND_API_KEY, RESEND_WEBHOOK_SECRET.

### 4.3 Reply-To inbound pattern

Problem: Replies go to the human From/Reply-To mailbox, not Resend, unless you capture them (architecture-prototype.md).

MVP pattern:

1. Verify sending subdomain e.g. send.yourdomain.com.
2. Enable receiving on a separate subdomain e.g. inbound.yourdomain.com with Resend MX (priority as shown in dashboard, commonly 10). Cite: https://resend.com/docs/dashboard/receiving/custom-domains
3. Do not put Resend MX on the root if Google Workspace already owns root MX. Cite: https://resend.com/docs/knowledge-base/how-do-i-avoid-conflicting-with-my-mx-records
4. On send, set reply_to to replies+{touchId}@inbound.yourdomain.com (or signed token).
5. On email.received, webhook is metadata only — fetch body via resend.emails.receiving.get(email_id). Cite: https://resend.com/docs/dashboard/receiving/get-email-content
6. Match plus-address / In-Reply-To to touches.rfc_message_id; create events type received; move Issue toward replied; surface in Inbox.
7. Outbound follow-ups: set In-Reply-To + References. Cite: https://resend.com/docs/dashboard/receiving/reply-to-emails

Managed *.resend.app inbound works for prototypes with zero DNS.

### 4.4 Domain DNS

Sending (https://resend.com/docs/add-a-domain, https://resend.com/docs/dashboard/domains/manage-domains):

1. Add domain/subdomain in Resend (prefer subdomain for reputation isolation).
2. Copy exact records from Records tab — historically SPF TXT + MX on send, DKIM TXT (resend._domainkey); domains created after August 2026 may show CNAME instead of TXT/MX — always paste what the dashboard shows.
3. Disable CDN proxy (e.g. Cloudflare orange cloud) on verification CNAMEs.
4. Wait for verify (often <15m, up to 72h).
5. Optional: DMARC on parent domain; tracking subdomain only if you enable open/click (default off for cold 1:1 per product IA).

Receiving: extra MX on inbound host; enable Receiving toggle; verify MX only if sending already verified.

App: store Resend domain id / from-address on sending_accounts; API key in server env (Phase 0: single team key RESEND_API_KEY is fine).

---
## 5. Vercel: project, env, cron / Inngest, webhooks

### 5.1 Project setup

1. Create Vercel project from apps/web (Root Directory = apps/web if monorepo).
2. Framework preset: Next.js.
3. Link Supabase + Resend + Inngest env vars (Production / Preview / Development).
4. Custom domain -> app.yourdomain.com; add that origin to Google + Supabase redirect allowlists.
5. Preview deployments: either add wildcard redirect URLs in Supabase or disable OAuth on previews.

### 5.2 Env matrix (Vercel)

| Name | Prod | Preview | Notes |
|---|---|---|---|
| NEXT_PUBLIC_SUPABASE_URL | yes | yes | |
| NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY / ANON_KEY | yes | yes | Public |
| SUPABASE_SERVICE_ROLE_KEY | yes | yes | Secret — server only |
| NEXT_PUBLIC_SITE_URL | prod URL | preview URL or omit | |
| RESEND_API_KEY | yes | yes | Secret |
| RESEND_WEBHOOK_SECRET | yes | optional separate | Secret |
| CRON_SECRET | yes | optional | Vercel Cron bearer |
| INNGEST_EVENT_KEY | yes | yes | Often auto via Inngest Vercel integration |
| INNGEST_SIGNING_KEY | yes | yes | Same |

Never expose privileged DB key or Resend/Inngest secrets with NEXT_PUBLIC_.

### 5.3 Jobs: prefer Inngest; thin Vercel Cron optional

**Inngest (recommended primary)** — https://www.inngest.com/docs/deploy/vercel , https://www.inngest.com/docs/guides/scheduled-functions

```ts
// app/api/inngest/route.ts
import { serve } from 'inngest/next'
import { inngest } from '@/lib/inngest/client'
import { reminderTick, processResendWebhook, sendApprovedBatch } from '@/lib/inngest/functions'

export const maxDuration = 300 // raise as plan allows

export const { GET, POST, PUT } = serve({
  client: inngest,
  functions: [reminderTick, processResendWebhook, sendApprovedBatch],
})
```

Use Inngest for: reminder/sequence stepper, send throttle batches, webhook fan-in processing, later Gmail users.watch renewal.

Cron example (timezone-aware):

```ts
inngest.createFunction(
  { id: 'reminder-tick', triggers: [{ cron: 'TZ=Asia/Seoul */15 * * * *' }] },
  async ({ step }) => { /* query due touches; enqueue sends */ }
)
```

**Vercel Cron** — https://vercel.com/docs/cron-jobs/quickstart , https://vercel.com/docs/project-configuration/vercel-json

```json
{
  "$schema": "https://openapi.vercel.sh/vercel.json",
  "crons": [
    { "path": "/api/cron/heartbeat", "schedule": "0 5 * * *" }
  ]
}
```

Secure with Authorization: Bearer ${CRON_SECRET}. Crons run on production only; schedules are UTC in Vercel expressions.

**Plan limits:** third-party summaries claim Hobby 2 crons (daily), Pro 40 — treat exact numbers as **uncertain** until confirmed in Vercel Cron docs/pricing for your account. Prefer Inngest schedules for frequent reminder ticks.

**Trigger.dev** alternative if jobs routinely exceed serverless duration. MVP: Inngest is enough.

### 5.4 Webhook routes

| Route | Auth | Role |
|---|---|---|
| POST /api/webhooks/resend | Svix signature | Ingest provider events |
| POST /api/inngest | Inngest signing | Job runner |
| GET /api/cron/heartbeat | CRON_SECRET | Optional health / enqueue |
| Phase 1: Gmail Pub/Sub push | OIDC / token verify | Reply sync |

Keep handlers fast: verify -> idempotent write or inngest.send -> 200.

---
## 6. Gmail OAuth Phase 1 (later notes)

Do not block Phase 0 on this. Detail in architecture-prototype.md section 1; summary:

### 6.1 Separate from product Google login

| OAuth | Client | Scopes | Purpose |
|---|---|---|---|
| Supabase Auth Google | Web client in Supabase | openid email profile | App login |
| Gmail sending account | Separate Google Cloud OAuth client | Gmail API scopes below | Send / read as user |

### 6.2 Scopes

Cite: https://developers.google.com/workspace/gmail/api/auth/scopes

| Scope | Sensitivity | When |
|---|---|---|
| gmail.send | Sensitive | Phase 1a — send via mailbox |
| gmail.readonly or gmail.modify | Restricted | Phase 1b — reply detection via API |
| mail.google.com/ | Restricted | Avoid |

### 6.3 Verification reality

Restricted scopes that store/transmit user mail on your servers -> Google OAuth verification + often CASA assessment (https://support.google.com/cloud/answer/13465431). Budget weeks, not a weekend.

MVP strategy: Resend outbound + Reply-To inbound first; add gmail.send when verification allows; add read scopes only when Pub/Sub reply sync ships.

### 6.4 Reply sync sketch

Pub/Sub users.watch + history.list; renew watch <=7 days (daily job); store threadId + RFC Message-ID.
Cite: https://developers.google.com/workspace/gmail/api/guides/push

### 6.5 Limits (cited earlier — re-check before enforcing caps)

Workspace mailbox ~2,000/day; Gmail API 500 recipients/message; API quota units per https://developers.google.com/workspace/gmail/api/reference/quota .
Personal Gmail ~500/day (https://support.google.com/mail/answer/22839).
Product must enforce per-account caps below provider ceilings.

---
## 7. Phase 0 / 1 / 2 build checklist (solo founder + agent)

Estimates assume one founder + coding agent, Asia/Seoul workdays, existing research docs. Parallelize "founder: DNS/OAuth consoles" with "agent: code."

### Phase 0 — Resend MVP Issue Board (target ~10-12 founder-days)

| Day | Founder | Agent |
|---|---|---|
| D1 | Supabase project, Vercel project, Google OAuth client for login, domain DNS start | Scaffold apps/web, @supabase/ssr clients, middleware, /auth/callback, login page |
| D2 | Confirm Google login works on localhost + prod URL allowlists | Migrations: workspaces, members, RLS helper + tests skeleton |
| D3 | Resend account, add send. + inbound. DNS | Schema: issues, prospects, templates, versions, touches, events, suppressions + RLS |
| D4 | Resend domain verified; webhook endpoint registered (tunnel for local) | Accounts UI + store Resend sending_account; Board list/kanban read path |
| D5 | — | Issue detail + prospect CSV/add; template editor v0 |
| D6 | — | Compose/preview; Approve -> create touches; Resend send adapter + idempotency |
| D7 | Watch first real delivers in Resend dashboard | Webhook verify + event writer; bounce/complaint -> suppressions |
| D8 | Inbound MX verified | email.received -> fetch body -> Inbox + status replied |
| D9 | Inngest Vercel integration | Reminder cron function; Ready->Sending throttle; daily_cap guard |
| D10 | Soft launch to self | Polish Board columns, Pause/Close, suppression block on Approve; basic stats |
| D11-12 buffer | DMARC, privacy copy, rate-limit tuning | Bugfix, RLS pgTAP, empty states |

**Phase 0 exit criteria:** Google login; create Issue; send via Resend; see delivered/bounce in timeline; capture reply via Reply-To; reminder fires; no privileged DB key in client bundle.

### Phase 1 — Gmail + hardening (~8-14 days + verification wait)

| Slice | Estimate | Notes |
|---|---|---|
| Gmail OAuth connect UI (gmail.send) | 2-3d | Separate client; encrypt refresh tokens |
| Send adapter parity | 1-2d | Same touches model |
| Google verification package | external calendar | Can overlap coding; blocked for production restricted scopes |
| Pub/Sub reply sync + watch renew | 3-4d | Needs readonly/modify + verification |
| Account health / caps UI | 1-2d | |
| Open confidence scoring (optional) | 1-2d | Treat opens as weak |

### Phase 2 — Memory, MCP, multiplayer (~8-12 days)

| Slice | Estimate |
|---|---|
| Memory search (person/company -> touches) | 2-3d |
| Winners snippets (reply-rate threshold) | 1-2d |
| MCP Streamable HTTP /api/mcp + tools from IA | 3-4d |
| Workspace invites / roles polish | 1-2d |
| Optional Supabase Edge Function experiments | 1d |

**MCP later:** tools list_issues, get_issue, draft_touch, schedule_send (HITL), get_memory, get_stats, list_templates — draft+propose default (issue-board-ia.md). Authz = same RLS membership (service calls with user JWT, not privileged admin key).

---
## 8. Security checklist

### 8.1 Never expose service_role / privileged DB key

- Only in server: Route Handlers, carefully reviewed Server Actions, Inngest, migrations tooling.
- lib/supabase/admin.ts must not be imported from Client Components.
- Prefer user-scoped server client + RLS for UI reads/writes; admin client for webhooks/jobs only.
- Rotate if leaked; treat like a root password.
  Cite: https://supabase.com/docs/guides/database/postgres/row-level-security (bypass notes)

### 8.2 Webhook signature verify

- Resend: Svix headers + raw body (https://resend.com/docs/dashboard/webhooks/verify-webhooks-requests).
- Reject on failure with 400; do not process.
- Idempotency on svix-id / provider_event_id.
- Optional IP allowlist from Resend docs (IPv4/IPv6 list in webhooks intro) — defense in depth, not a substitute for signatures.

### 8.3 Rate limits and abuse

| Layer | Action |
|---|---|
| Resend API | Respect 10 rps/team; queue sends in Inngest with concurrency keys per account |
| App send Approve | Per-workspace and per-account hourly/daily caps in DB |
| Auth | Supabase Auth platform limits + login CSRF/PKCE |
| Public routes | Webhook and cron routes authenticated; no open send API |
| Vercel Cron | CRON_SECRET bearer check |
| Inngest | Signing key; do not expose event key to browser |
| RLS | Enabled on all tenant tables; anon revoked |
| Secrets | Resend keys, Google client secrets, Gmail refresh tokens encrypted at rest |
| Open redirects | Sanitize next param on auth callback (relative paths only) |
| Tracking | Default off for cold; disclose if enabled |

### 8.4 Optional Edge Functions

Use for Deno webhook receivers or co-located secrets if desired (https://supabase.com/docs/guides/functions). For this MVP, Next.js on Vercel is enough; adding Edge Functions early splits observability. If used: store secrets in Supabase project secrets; keep handlers idempotent; avoid long jobs (use Inngest).

---

## Quick reference — official docs (2026)

| Topic | URL |
|---|---|
| Supabase SSR Next.js | https://supabase.com/docs/guides/auth/server-side/nextjs |
| Supabase Google Auth | https://supabase.com/docs/guides/auth/social-login/auth-google |
| Supabase RLS | https://supabase.com/docs/guides/database/postgres/row-level-security |
| Supabase Edge Functions | https://supabase.com/docs/guides/functions |
| Resend Send API | https://resend.com/docs/api-reference/emails/send-email |
| Resend Webhooks | https://resend.com/docs/dashboard/webhooks/introduction |
| Resend Verify | https://resend.com/docs/dashboard/webhooks/verify-webhooks-requests |
| Resend Event types | https://resend.com/docs/webhooks/event-types |
| Resend Receiving / custom domains | https://resend.com/docs/dashboard/receiving/custom-domains |
| Resend Add domain | https://resend.com/docs/add-a-domain |
| Vercel Cron quickstart | https://vercel.com/docs/cron-jobs/quickstart |
| Inngest on Vercel | https://www.inngest.com/docs/deploy/vercel |
| Gmail scopes | https://developers.google.com/workspace/gmail/api/auth/scopes |

---

## Explicit uncertainties

- Exact Vercel Hobby/Pro cron job counts and minimum intervals — confirm on current Vercel pricing/docs for the linked account before designing around Vercel Cron frequency.
- Supabase dashboard key naming (anon vs publishable vs secret) — follow the project Connect dialog at create time (https://supabase.com/docs/guides/api/api-keys).
- Resend DNS shape (TXT/MX vs CNAME) depends on domain creation date (post-August 2026 CNAME path) — always copy dashboard Records.
- Google Workspace API overage billing details through 2026 — see Google tools-safety notices; do not hard-code unverified paid overage numbers.
- @supabase/supabase-js cookie timing around exchangeCodeForSession — retest after upgrades (https://github.com/supabase/supabase-js/issues/2037).

---

*End of blueprint. Implement Phase 0 against this file + issue-board-ia.md; keep provider adapters swappable for Phase 1 Gmail.*
