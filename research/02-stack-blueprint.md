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

