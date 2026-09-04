# Cold Outreach Issue Board — UI/UX Build Plan (shadcn/ui + ko/en i18n)

**Date:** 2026-09-04 (Asia/Seoul)  
**Depends on:** [`issue-board-ia.md`](./issue-board-ia.md)  
**Stack assumption:** Next.js App Router + Tailwind + shadcn/ui (Radix) + next-intl  
**Audience:** indie founders / small agencies · Linear-like density

---

## 0. Design north star

Mirror the IA metaphor: **Issue = purpose**, Board = status flow, HITL Approve on Ready→Sending. UI should answer in <10s: *이 목적(Issue)으로, 어느 계정에서, 누구에게, 뭐 보냈고, 다음 리마인드, 회신은?*

Visual language: **Linear density** — tight vertical rhythm, muted surfaces, high-contrast primary actions, status as color-coded badges (not loud kanban pastels). Dark mode first-class (founders live in dark IDE UIs).

---

## 1. shadcn init conventions

### 1.1 Recommended init

Scaffold with create-next-app (TypeScript, Tailwind, App Router, src dir), then run shadcn init (latest CLI).

```text
pnpm create next-app@latest cold-outreach --src-dir --typescript --tailwind --eslint --app
cd cold-outreach
pnpm dlx shadcn@latest init
```

Prompts / components.json targets:

| Setting | Value | Why |
|---------|-------|-----|
| style | new-york | Official default; default deprecated. Tighter controls suit Linear density. |
| Primitives | Radix (CLI classic / radix path) | Mature a11y for Dialog, Sheet, Dropdown, Tabs, Checkbox. |
| rsc | true | App Router Server Components |
| tsx | true | — |
| tailwind.cssVariables | true | Semantic tokens; theme without rewriting component classes |
| tailwind.baseColor | neutral or zinc | Cool, product-y |
| iconLibrary | lucide | Matches shadcn docs |


Example components.json shape (align with generated output). Set style to new-york, rsc true, tsx true, cssVariables true, baseColor neutral, iconLibrary lucide, and standard aliases (@/components, @/lib/utils, @/components/ui, @/lib, @/hooks). Schema URL: https://ui.shadcn.com/schema.json

```json
{
  "$schema": "https://ui.shadcn.com/schema.json",
  "style": "new-york",
  "rsc": true,
  "tsx": true,
  "tailwind": {
    "config": "",
    "css": "src/app/globals.css",
    "baseColor": "neutral",
    "cssVariables": true
  },
  "iconLibrary": "lucide",
  "aliases": {
    "components": "@/components",
    "utils": "@/lib/utils",
    "ui": "@/components/ui",
    "lib": "@/lib",
    "hooks": "@/hooks"
  }
}
```

**Citations**

- Install (Next.js): https://ui.shadcn.com/docs/installation/next
- components.json / style / cssVariables: https://ui.shadcn.com/docs/components-json
- Theming tokens: https://ui.shadcn.com/docs/theming

### 1.2 Theming for Linear-like density

shadcn recommends CSS variables under :root / .dark with semantic pairs (primary / primary-foreground, plus sidebar-* tokens). See https://ui.shadcn.com/docs/theming.

**Density overrides** (project conventions — not shadcn defaults):

| Token / rule | Linear-like target |
|--------------|-------------------|
| --radius | 0.375rem–0.5rem (flatter than default 0.625rem) |
| Page padding | px-4 py-3 shell; content gap-3 not gap-6 |
| Card padding | Prefer p-3 / p-4; avoid spacious marketing cards on Board |
| Table row height | h-9–h-10; compact text-sm |
| Button | Default size sm in toolbars; default size only for Approve / Connect |
| Font | Inter or Geist; UI text-sm body, text-xs meta |
| Borders | Prefer border-border hairlines over shadows; shadow only on floating Sheet/Popover |
| Status colors | Map Issue statuses to Badge variants + custom CSS vars: --status-draft, --status-ready, --status-sending, --status-waiting, --status-replied, --status-snoozed, --status-closed |

Add custom tokens in globals.css (pattern from theming docs):

```css
:root {
  --radius: 0.5rem;
  --status-draft: oklch(0.7 0.02 260);
  --status-ready: oklch(0.75 0.12 85);
  --status-sending: oklch(0.65 0.15 250);
  --status-waiting: oklch(0.7 0.1 200);
  --status-replied: oklch(0.65 0.17 145);
  --status-snoozed: oklch(0.6 0.05 300);
  --status-closed: oklch(0.55 0.02 260);
}
@theme inline {
  --color-status-draft: var(--status-draft);
  /* repeat for each status */
}
```


Dark mode: next-themes + .dark class (shadcn dark mode docs). Mount theme toggle in Sidebar footer / Settings.

### 1.3 App shell layout

SidebarProvider wraps AppSidebar (Board, Inbox, Templates, Memory, Accounts, Settings) and SidebarInset (TopBar with breadcrumbs, Command trigger, locale switch, user; then page content). Mount Sonner Toaster at root.

Use official Sidebar composition: https://ui.shadcn.com/docs/components/base/sidebar

---


## 2. Component map per screen

Install baseline once (see section 3). Map = primary shadcn primitives + custom compositions.

### 2.1 Board (home / kanban + list)

| UI need | shadcn / custom |
|---------|-----------------|
| Columns by status | **Custom Kanban** (BoardColumn, IssueCard) — no first-party Kanban; use dnd-kit + Card |
| Issue card | **Card** + **Badge** (status, reply rate) + **Avatar** (owner) + account **Badge** chips |
| Filters | **DropdownMenu** / **Popover** + **Checkbox** / **Select**; filter bar = custom |
| Quick create | **Dialog** or **Sheet** (mobile Sheet) + **Form** (react-hook-form + zod) |
| List view toggle | **Tabs** (Board | List) |
| List rows | **Table** |
| Empty / loading | **Skeleton**; empty Card with CTA |
| Drag feedback | Custom; toast via **Sonner** on status change |
| Command jump | **Command** (CommandDialog) — Go to issue, Create issue |

Statuses (IA): Draft · Ready · Sending · Waiting · Replied · Snoozed · Closed.

### 2.2 Issue detail (hub)

| Region | Components |
|--------|------------|
| Header | Custom sticky header: title **Input** (inline), status **Select** / **Badge**, purpose one-liner |
| Top actions | **Button** group: Approve send · Pause · Close · Preview → **AlertDialog** confirm for Approve/Close |
| Left Prospects | **Table** + row **Checkbox**; **Sheet** for prospect drawer |
| Center Timeline | Custom vertical timeline; event chips = **Badge**; load more = **Button** |
| Right Sequence / accounts | **Card** + **Switch** (reminders) + **Select** (send-from) |
| Secondary nav | **Tabs**: Timeline · Prospects · Sequence · Preview · Stats |
| Overflow | **DropdownMenu**; agent draft cue = muted **Alert** |

### 2.3 Compose / Preview

| Need | Components |
|------|------------|
| Layout | Split panes (custom CSS grid); desktop chrome vs mobile frame |
| Subject / body | **Form** + **Input** + **Textarea** (or TipTap later); variables chips = **Badge** |
| From account | **Select** (Gmail vs Resend accounts) |
| Prospect picker | **Combobox** pattern (Popover + Command) |
| Actions | Send now / Schedule / Save draft — **Button** + Schedule **Popover** + **Calendar** |
| Preview | Sandboxed iframe (see section 5); device toggle = **Tabs** or **ToggleGroup** |
| Unsaved leave | **AlertDialog** |

### 2.4 Memory

| Need | Components |
|------|------------|
| Search | **Command** inline or **Input** + debounced results |
| Result list | **ScrollArea** + list rows |
| Person detail | **Card** sections: past Issues, touches (provider **Badge**), Winners snippets, suppression |
| Winners badge | **Badge** only if n ≥ threshold (IA rule) |
| Suppression | **Switch** + destructive **AlertDialog**; blocks Approve send |
| Empty | Illustrated empty state Card |

### 2.5 Inbox

| Need | Components |
|------|------------|
| Filters | **Tabs** or **ToggleGroup**: All · Replies · Bounces · Complaints |
| Thread list | Custom list or **Table**; unread = font-medium |
| Thread pane | **Card** / scroll; link to Issue via Link |
| Actions | **Button**: Mark replied · Pause sequence · Open issue |
| Bounce/complaint cue | **Alert** (destructive/warning) |

### 2.6 Accounts

| Need | Components |
|------|------------|
| Connected list | **Card** grid or **Table** |
| Connect Gmail / Resend | **Button** → OAuth redirect / **Dialog** for API key + domain |
| Caps / health | **Progress** (daily cap), health **Badge**, checklist |
| SPF/DKIM/DMARC | **Card** checklist; copy DNS = **Button** + Sonner |
| Tracking subdomain | **Switch** (Resend; default off for cold 1:1) |

### 2.7 Templates

| Need | Components |
|------|------------|
| Library | **Card** grid; performance **Badge** (n≥threshold) |
| Editor | Form + version **Tabs** or **Select** |
| Preview | Same iframe pattern as Compose |
| Version history | **Sheet** or side **ScrollArea** list |
| Create / duplicate | **Dialog** + **Form** |

### 2.8 Cross-cutting chrome

| Concern | Components |
|---------|------------|
| Nav | **Sidebar** (+ SidebarMenuBadge for Inbox unread) |
| Global search / actions | **Command** |
| Toasts | **Sonner** — https://ui.shadcn.com/docs/components/radix/sonner |
| Settings | Forms: workspace, compliance, MCP keys — **Form** + **Input** + **Separator** |
| Destructive confirms | **AlertDialog** |
| Mobile nav | Sidebar sheet behavior (built into Sidebar) |

### 2.9 Component → screen matrix (quick ref)

| Component | Board | Issue | Compose | Memory | Inbox | Accounts | Templates |
|-----------|:-----:|:-----:|:-------:|:------:|:-----:|:--------:|:---------:|
| Sidebar | yes | yes | yes | yes | yes | yes | yes |
| Card | yes | yes | | yes | yes | yes | yes |
| Badge | yes | yes | yes | yes | yes | yes | yes |
| Table | yes | yes | | | sec | sec | |
| Tabs | yes | yes | yes | | yes | | yes |
| Sheet | sec | yes | | | | sec | yes |
| Dialog / AlertDialog | yes | yes | yes | yes | | yes | yes |
| Form | yes | sec | yes | | | yes | yes |
| Command | yes | yes | yes | yes | yes | | |
| Select / Combobox | yes | yes | yes | | | yes | yes |
| Sonner | yes | yes | yes | yes | yes | yes | yes |
| Custom Kanban | yes | | | | | | |
| iframe preview | | sec | yes | | | | yes |

yes = primary · sec = secondary

---


## 3. Fast prototype workflow

### 3.1 One-shot shadcn add list (day 0)

Add in one CLI pass:

```text
pnpm dlx shadcn@latest add sidebar button badge card table tabs sheet dialog alert-dialog dropdown-menu popover select checkbox switch input textarea label separator scroll-area skeleton avatar progress calendar command sonner form toggle-group tooltip
```


Optional soon after: accordion, alert, breadcrumb, collapsible, hover-card, pagination, radio-group, slider.

Kanban: do not wait for a registry Kanban — scaffold components/board/{board,column,issue-card}.tsx with @dnd-kit/core + @dnd-kit/sortable.

### 3.2 Figma-free wire → component path

1. ASCII / Excalidraw (optional) — 10 min per screen from IA.
2. Route stubs under app/[locale]/(app)/… with hardcoded mock data.
3. Compose shadcn only — no custom CSS until density tokens hurt.
4. Promote repeats — e.g. StatusBadge, AccountChip, ProspectRow, EmailPreviewFrame.
5. Wire real data behind the same props (server fetch → client islands for DnD / editors).

Rule: one screen = one page file + colocated _components/; shared primitives stay in components/ui (CLI) and components/ (product).

### 3.3 Design tokens checklist

| Layer | What to lock week 1 |
|-------|---------------------|
| Color | Semantic shadcn tokens + status + sidebar |
| Radius | Single --radius |
| Type | text-xs meta · text-sm UI · text-base titles max |
| Space | 4px grid: gap-1…gap-4 in app chrome |
| Motion | Prefer none on Board DnD ghost; 150ms fades on Sheet/Dialog only |
| Elevation | Border-first; shadow-sm on popovers |

Visual preset playground: https://ui.shadcn.com/create (shadcn/create) — optional for brand preset export.

---

## 4. i18n: next-intl (pick) · ko / en

### 4.1 Decision: next-intl (not next-i18next)

| Criterion | next-intl | next-i18next / i18next |
|-----------|-----------|-------------------------|
| App Router + RSC | First-class docs and plugin | App Router support improved (v16+), but heavier ecosystem |
| Bundle / DX for Next-only | Small, ICU MessageFormat | Larger plugin surface |
| Locale routing | defineRouting + [locale] | Possible; more DIY historically |
| When to choose other | — | Existing i18next backends, Locize plugins, multi-framework |

**Pick next-intl** unless the team already depends on i18next backends/TMS plugins.

Docs: https://next-intl.dev/docs/getting-started/app-router · routing: https://next-intl.dev/docs/routing/setup · comparison: https://www.locize.com/blog/next-intl-vs-next-i18next

### 4.2 Locale routing /ko · /en

In src/i18n/routing.ts:

```ts
import { defineRouting } from 'next-intl/routing';

export const routing = defineRouting({
  locales: ['ko', 'en'],
  defaultLocale: 'ko', // or 'en' if global-first
  localePrefix: 'always' // /ko/board, /en/board
});
```


App tree:

```
src/app/[locale]/layout.tsx
src/app/[locale]/(app)/board/page.tsx
src/app/[locale]/(app)/inbox/page.tsx
…
```

Middleware / proxy (Next.js 16+ docs call this proxy.ts; older = middleware.ts): createMiddleware(routing) with matcher excluding api, trpc, _next, _vercel, and dotted static files.

Navigation wrappers via createNavigation(routing) → use product Link / useRouter so locale sticks: https://next-intl.dev/docs/routing/setup

i18n/request.ts: load messages; on Next 16.3+ prefer next/root-params for locale (see same routing setup doc). generateStaticParams for ko/en.

Set html lang={locale} in [locale]/layout.tsx.

### 4.3 Message file structure

```
messages/
  en.json
  ko.json
```

Namespace by screen / domain (stable keys, English identifiers). Suggested top-level namespaces: Nav, Board, IssueStatus, Issue, Compose, Inbox, Memory, Accounts, Templates, Common, Errors.

Example IssueStatus keys: draft, ready, sending, waiting, replied, snoozed, closed.

```json
{
  "Nav": {
    "board": "Board",
    "inbox": "Inbox",
    "templates": "Templates",
    "memory": "Memory",
    "accounts": "Accounts",
    "settings": "Settings"
  },
  "IssueStatus": {
    "draft": "Draft",
    "ready": "Ready",
    "sending": "Sending",
    "waiting": "Waiting",
    "replied": "Replied",
    "snoozed": "Snoozed",
    "closed": "Closed"
  },
  "Issue": {
    "approveSend": "Approve send",
    "pause": "Pause",
    "close": "Close",
    "preview": "Preview"
  },
  "Common": {
    "save": "Save",
    "cancel": "Cancel",
    "loading": "Loading…"
  }
}
```


Korean file mirrors keys; use ICU for plurals via next-intl.

**Do not translate:** user-generated Issue titles, email bodies, prospect names, template content (those stay as authored; UI chrome around them is translated).

### 4.4 Day-1 vs later translation scope

| Day-1 (must) | Later |
|--------------|--------|
| Nav labels, Board columns/statuses, primary CTAs (Approve, Pause, Connect) | Marketing / onboarding tour |
| Inbox filters and triage actions | Advanced Settings copy, MCP docs in-app |
| Compose actions + validation errors | Chart tooltips, long empty-state essays |
| Accounts connect + health labels | Email HTML template gallery blurbs |
| Common / Errors / AlertDialog strings | Changelog, billing |
| lang + locale switcher | Localized pathnames (keep /board, do not translate path segments) |
| Date/number formatting via next-intl (useFormatter) | Relative time edge cases polish |

Ship both ko.json and en.json from day 1 with parity on Day-1 keys (even if EN is source and KO is draft). Avoid hard-coded Korean or English in components.

Locale switcher: small Select in TopBar / Sidebar footer calling useRouter + pathname replace.

---


## 5. Accessibility and email preview (iframe sandbox)

### 5.1 a11y baseline (Radix + product rules)

- Prefer shadcn/Radix primitives for focus traps (Dialog, Sheet, AlertDialog, Dropdown).
- Icon-only buttons: always aria-label (i18n key).
- Board DnD: provide keyboard alternative (Move to status DropdownMenu on card) — drag-only is not enough.
- Status color: never color-only; pair with text label on Badge.
- Hit targets: toolbar icons ≥ 32px; dense tables OK if row actions are keyboard reachable.
- Toasts: Sonner for non-blocking feedback; use AlertDialog for irreversible Approve/Close.
- Skip link to #main inside SidebarInset.
- Respect prefers-reduced-motion for Sheet/Dialog animations.
- Forms: Label + error text linked with aria-describedby; zod messages via i18n.

### 5.2 Email preview iframe pattern

Goals: isolate untrusted HTML (templates, inbound snippets), avoid CSS bleed, block script execution.

Pattern sketch for EmailPreviewFrame:

```tsx
<iframe
  title={t("previewFrame")}
  sandbox="allow-same-origin"
  referrerPolicy="no-referrer"
  srcDoc={safeHtmlDocument}
  className="h-full w-full rounded-md border bg-white"
/>
```

Notes: no allow-scripts; prefer srcDoc or blob URL from sanitized HTML.

Practices:

| Practice | Detail |
|----------|--------|
| Sandbox | Default deny scripts/forms/top-nav. Prefer srcDoc or blob: |
| Sanitize | Server-side sanitize (e.g. DOMPurify isomorphic) before preview; strip script, event handlers, remote trackers if showing third-party HTML |
| Document chrome | Wrap body in minimal HTML + CSP meta default-src none; img-src data: https:; style-src unsafe-inline as defense in depth |
| Desktop / mobile | Outer device frame is React; inner iframe content width 600px vs 375px |
| Dark app / light email | Force iframe canvas light (bg-white) — email clients are light-first |
| Accessibility | title on iframe; visible Open plain-text toggle for screen-reader-friendly alternative |
| Compose sync | Debounced serialize editor → sanitized HTML → iframe; do not live-bind unsanitized user HTML |

Inbound Inbox HTML: treat as hostile — same sandbox + sanitize; never dangerouslySetInnerHTML in parent document.

---

## 6. One-week UI sprint (day-by-day)

Assumes Next.js app exists or is created day 1 morning; mock data OK; no real Gmail/Resend required for UI.

### Day 1 — Foundation · i18n · shell
- shadcn init (new-york, cssVariables, neutral)
- Bulk shadcn add (section 3.1)
- Density tokens + dark mode
- next-intl: routing ko/en, messages/*.json Day-1 keys, [locale] layout
- Sidebar + TopBar + Sonner + Command stub
- **Done when:** /ko and /en show shell with nav labels translated

### Day 2 — Board
- Custom Kanban columns from Issue statuses
- IssueCard (title, prospects count, reply rate, next reminder, account chips)
- Filters stub + Quick create Dialog
- Tabs: Board | List (Table)
- **Done when:** mock issues drag (or menu-move) across columns; create opens Dialog

### Day 3 — Issue detail
- Header + action Buttons + AlertDialog for Approve
- Tabs: Timeline · Prospects · Sequence · Preview · Stats
- Prospects Table; Timeline custom list; Sequence Card + Switches
- **Done when:** founder can walk one Issue hub end-to-end on mocks

### Day 4 — Compose / Preview + Templates
- Split Compose layout; Form fields; account Select
- EmailPreviewFrame iframe sandbox + desktop/mobile Tabs
- Templates grid Cards + editor stub + version Sheet
- **Done when:** preview updates from mock template HTML safely

### Day 5 — Inbox + Memory
- Inbox Tabs + thread list + Mark replied / Pause actions
- Memory search + person detail + Winners Badge + suppression Switch
- Wire Command palette to jump Issues / Memory
- **Done when:** triage + “what did we send this person?” flows clickable

### Day 6 — Accounts + Settings polish
- Accounts cards: connect CTAs, cap Progress, DNS checklist, health Badges
- Settings form stubs (workspace, compliance, MCP key field masked)
- Empty/skeleton/error states; Sonner on all mutations
- a11y pass: focus order, dialog labels, DnD keyboard menu
- **Done when:** all IA primary nav screens exist at prototype fidelity

### Day 7 — i18n parity · density QA · handoff
- Fill remaining Day-1 strings in ko.json / en.json; fix hard-coded UI copy
- Visual QA vs Linear density checklist (radius, row height, badge contrast)
- Record Loom or annotated screenshots per screen
- Backlog: real data hooks, TipTap, charts, onboarding
- **Done when:** bilingual UI demoable; this plan’s checklist checked

---

## 7. Definition of done (UI prototype)

- [ ] All IA primary nav screens rendered with shadcn + mock data
- [ ] Approve send is an explicit AlertDialog (HITL)
- [ ] Email preview uses sandboxed iframe (no parent XSS)
- [ ] /ko/* and /en/* parity for Day-1 message keys
- [ ] Keyboard path for status changes on Board
- [ ] Sidebar + Command + Sonner in app chrome

---

## 8. Citations (docs fetched / searched)

| Topic | URL |
|-------|-----|
| shadcn Next.js install | https://ui.shadcn.com/docs/installation/next |
| shadcn components.json | https://ui.shadcn.com/docs/components-json |
| shadcn theming / CSS variables | https://ui.shadcn.com/docs/theming |
| shadcn Sidebar | https://ui.shadcn.com/docs/components/base/sidebar |
| shadcn Sonner | https://ui.shadcn.com/docs/components/radix/sonner |
| shadcn Command | https://ui.shadcn.com/docs/components/aria/command |
| next-intl App Router | https://next-intl.dev/docs/getting-started/app-router |
| next-intl locale routing | https://next-intl.dev/docs/routing/setup |
| next-intl routing config | https://next-intl.dev/docs/routing/configuration |
| next-intl vs next-i18next | https://www.locize.com/blog/next-intl-vs-next-i18next |
| Product IA | ./issue-board-ia.md |

---

*End of plan — ready to execute Day 1 of the sprint.*
