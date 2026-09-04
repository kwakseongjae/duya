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

SEE_FULL_FILE_ON_DISK