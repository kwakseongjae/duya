# Cold Outreach Research Session — 2026-09-04 evening (KST)

## In flight
- Parallel deep dives → `01-market-viral-conventions.md`, `02-stack-blueprint.md`, `03-ui-shadcn-i18n.md`
- Routine `cold-outreach-research-waves` every 15m until ~22:45 KST

## Locked product bets (from prior IA)
- Primary object: **Issue** (purpose cluster), not Campaign blast
- Surfaces: Board · Inbox · Templates · Memory · Accounts · Settings
- Stack target: **Next.js App Router + Vercel + Supabase + Resend + Google OAuth + shadcn + next-intl (ko/en)**
- Coding agents: prefer Grok Build when usable; else local Cursor CLI (not cloud)

## Early stack notes (parent pre-read)
### Auth
- MVP: **Supabase Auth + Google OAuth** with `@supabase/ssr` (browser/server/middleware clients)
- Flow: `signInWithOAuth` → `/auth/callback` → `exchangeCodeForSession` (PKCE)
- Allowlist: localhost + prod + Vercel preview patterns; Google Console must list Supabase callback `https://<project>.supabase.co/auth/v1/callback`
- Prefer Supabase over Better Auth for MVP because Postgres RLS + Auth share one identity (`auth.uid()`)

### i18n
- Prefer **next-intl** with `[locale]` segment (`ko`, `en`)
- Prefer `localePrefix: 'as-needed'` or `always` for SEO; day-1 messages for nav, Issue statuses, auth, empty states
- Next 16.2+: `next/root-params` for locale in `i18n/request.ts` when available; else `setRequestLocale` + `generateStaticParams`

### UI
- shadcn (Radix) + Linear-like density; Sidebar, Table, Sheet, Dialog, Badge, Command, Sonner
- Email preview: sandboxed iframe desktop + mobile frames

### Viral angles (hypothesis — validate in wave docs)
1. “Cold email as Issues” demo GIF (board → approve → reply)
2. MCP: `get_memory(person)` live in Cursor
3. Gmail + Resend one ledger screenshot
4. Open-source Issue board template / AGENTS.md starter
5. Before/after reply rate with transparent n (not open-rate vanity)

## Existing artifacts
- `issue-board-ia.md` + wireframes
- `market-competitors.md`, `architecture-prototype.md`
