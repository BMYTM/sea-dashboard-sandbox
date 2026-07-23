# SEA Dashboard — Activity Sandbox

A **sandbox copy** of the HONO SEA weekly pipeline review dashboard, extended with
activity-mapped prep tracking. It exists to trial a new operating model before any
of it touches production.

> **This copy is deliberately disconnected from live team data.**
> Both Power Automate flow URLs are blank, so it can neither read from nor write to
> the production SharePoint list. It persists to the browser only, under its own
> `localStorage` namespace (`sea-sandbox-prep-v1`). Nothing done here can affect
> the live dashboard.

---

## Why this exists

The production dashboard tracks deals well but starts at **Qualification**. Everything
that happens *before the first meeting* — deciding the account is worth pursuing,
identifying who signs, forming a point of view — was invisible. And stage checklist
items were plain tick-boxes: no owner, no due date, no evidence. So "who is doing what
before Thursday's demo" had nowhere to live.

This build adds that layer.

---

## What's new

### 1. Pre-engagement stage
A new **Prospect / Pre-Engagement** stage sits before Qualification, with six required
activities that must happen before we ever meet the client:

- Account scored against ICP (fit confirmed)
- Trigger / compelling event researched
- Org structure mapped — economic buyer named
- Value hypothesis (POV) drafted
- Entry path agreed (referral / partner / exec intro / outbound)
- First-meeting objective and desired next step written

The pre-engagement block stays visible on a deal until it is finished, whatever stage
the deal has reached — so gaps don't get skipped by advancing past them.

### 2. Activities carry owner, due date and evidence
Every checklist item is now `{ done, owner, due, note }` instead of a bare boolean.
Overdue open items flag red automatically.

Old boolean data upgrades transparently on load, so this is safe to merge.

### 3. Meeting prep packs
Five prep packs — **First Meeting, Demo, Proposal, Review, Onsite** — auto-selected from
the meeting mode and overridable per meeting. Each meeting shows a live **readiness %**
before it runs, visible both in the meeting modal and on the parent deal.

### 4. Real stage gating
The original `advanceStage()` carried the comment *"only if checklist complete"* but never
enforced it — only the button was disabled, so any programmatic path advanced freely.
It is now a hard check with an explanatory prompt.

### 5. "Prep & Activities" tab
One screen showing every open activity across all deals and meetings, overdue first, with:

- Overdue count
- Total open activities
- Unassigned count
- Meetings not yet prep-ready

### 6. Extended HONO score
Adds pre-engagement completeness (up to 8 pts) and next-meeting readiness (up to 7 pts);
deducts 4 points per overdue activity.

---

## Running it

Open `index.html` in a browser. No build step, no server, no dependencies to install —
it is a single self-contained file. Charts, Excel export and PPT export load from CDN.

Via GitHub Pages: **Settings → Pages → Source: `main` / root**.

---

## Taking it live later

The sandbox is local-only by design. To connect it to real data **after** the model is agreed:

1. Create a **separate** SharePoint list — do not point at the production one.
2. Create a **new** pair of Power Automate flows (read + write) against that list.
3. Paste those two URLs into the `CONFIG` block at the top of the `<script>` section.

**Do not paste the production flow URLs into this file.**

### A note on secrets

Power Automate flow URLs contain a `sig=` token that grants full read/write access to
whatever they point at. Anything committed to a public repository is public permanently,
including in git history. Keep these URLs out of public repos — or accept that the data
behind them is effectively public.

---

## Merging back

The changes are additive and isolated to identifiable blocks:

| Area | Change |
|---|---|
| `CONFIG` | sandboxed — **do not merge**, production keeps its own |
| `LS_KEY` | sandboxed — **do not merge** |
| `STAGES` / `STAGE_MAP` / `FUNNEL` / `STAGE_ORDER` | added `prospect` |
| Activity Engine block | new — insert wholesale |
| `renderActivityBlock(d)` | replaces the inline checklist IIFE in `renderDealRow` |
| `calcHonoScore` | extended scoring |
| Tab nav / `renderTabNav` / `render` | added `prep` view |
| `#mtgPrepHost` + `_injectMtgPrep` | meeting modal prep block |
| Stylesheet | appended `.act-*` and `.prep-*` rules |

Everything else is untouched from the original.
