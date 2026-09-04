# GPM Route Builder — Spec Plan v1
**Status:** READY FOR CLI HANDOFF  
**Date:** 2026-06-24  
**Owner:** Matthieu Fournier — Green Property Management  
**Stack:** Google Apps Script Web App + Google Sheets + Google Route Optimization API

---

## Single Responsibility

A mobile-accessible GAS Web App that lets GPM maintenance techs self-select their daily work orders, pin urgent jobs, and receive an optimized route — with full manager visibility into scheduled vs. unscheduled work, workload rebalancing controls, and a tech-facing help flag for skill gaps or Jason escalations.

---

## Architecture

```
AppFolio (source of truth)
    ↓ [manual export or Sheets sync]
Google Sheets (backend / log)
    ↓
GAS Web App (tech-facing UI)
    ↓ [tech selects, pins, confirms, flags for help]
Google Route Optimization API (solver)
    ↓
Ordered Route Output → displayed to tech + logged to Sheets
    ↓
Manager Dashboard (Sheets tab)
    ↓ [manager reassigns WOs between techs]
Rebalanced WO → moves to new tech's queue
```

---

## Data Inputs

| Field | Source | Notes |
|---|---|---|
| Work Order ID | AppFolio → Sheets | Key identifier |
| Property Address | AppFolio → Sheets | Used for geo-routing |
| Work Type | AppFolio → Sheets | Plumbing, HVAC, etc. |
| Assigned Tech | AppFolio → Sheets | Filter by logged-in tech |
| Scheduled (Y/N) | AppFolio → Sheets | Pre-existing appointment |
| Priority Flag | AppFolio → Sheets | Urgent / Normal |
| Estimated Duration | Manual or AppFolio | Needed for route solver |
| Tech Start Location | GPS or manual entry | Route origin point |

**Known Gap:** AppFolio's API is limited. Data pull likely requires a manual Sheets export step or CSV import pipeline until a direct sync is built. Flag this as Day 1 constraint.

---

## Core Features — V1 Scope

### 1. Tech Login / Identity
- Tech opens URL → selects their name from dropdown (no auth for MVP)
- Loads only their assigned work orders for today

### 2. Work Order Selection
- List view: all assigned WOs for the day
- Checkboxes: tech selects what they're taking on
- Unscheduled WOs clearly marked — tech decides to include or defer

### 3. Pin Urgent
- Any WO flagged urgent can be pinned to top of route
- Pinned = fixed position #1, #2, etc. — optimizer routes around them

### 4. Route Generation
- On "Build My Route" button press:
  - Pinned jobs → locked sequence at top
  - Remaining selected jobs → sent to Google Route Optimization API
  - Returns ordered stop list with estimated drive time between stops

### 5. Route Output Display
- Ordered list: Stop #, Address, WO Type, Estimated Duration
- Simple map link per stop (Google Maps deep link)
- "Confirm Route" button → logs final route to Sheets

### 6. Manager Rebalance View
- Dedicated Sheets tab: all techs' daily load side-by-side
- Columns per tech: Stop Count, Estimated Total Hours, WO IDs
- Manager reassigns a WO by changing the Assigned Tech cell → WO disappears from Tech A's queue, appears in Tech B's on next app refresh
- Manager is the only one who can initiate reassignment — no tech-to-tech transfers in V1
- Help-flagged WOs surface prominently in this view

### 7. Tech Help Flag
- Tech taps "Flag for Help" on any WO in their queue
- Two reason options: "Don't know how to do this" / "Bringing to Jason's attention"
- Optional free-text note (max 140 chars)
- Flag writes to Sheets: Tech Name, WO ID, Reason, Note, Timestamp
- GAS email trigger fires to Jason automatically on submission

---

## Out of Scope — V1

- Real-time GPS tracking
- Push notifications / reminders
- Automated AppFolio sync (manual export only in V1)
- Time-window enforcement
- In-app WO status updates (tech marks complete)
- Tech-to-tech direct transfers (manager-only reassignment)

---

## V2 Backlog

1. Direct AppFolio → Sheets live sync
2. Manager dispatcher view (all techs on one screen)
3. Tech marks WO complete in-app → writes back to AppFolio
4. Time-window constraints (tenant availability windows)
5. RouteIQ commercialization layer (multi-tenant, per-seat billing)
6. Real auth (Google SSO via GAS)

---

## Constraints & Safety

| Risk | Mitigation |
|---|---|
| Tech skips app, calls in anyway | Measure adoption weekly — if <50% usage in 30 days, reframe or kill |
| AppFolio data is stale | Timestamp on every Sheets pull — warn tech if data >24hrs old |
| Route Optimization API quota | 100 requests/day free tier — sufficient for MVP |
| Wrong tech assigned to WO | WO assignment stays in AppFolio, app is read-only on assignments |
| Tech goes offline mid-day | Route output is static once confirmed — no live dependency |

---

## Acceptance Criteria

1. A tech can open the URL on a phone, select their jobs, pin urgents, and receive an ordered route in under 3 minutes.
2. Manager can see all confirmed routes for the day in Sheets without asking techs directly.
3. Scheduled vs. unscheduled work is visually distinct — tech cannot accidentally skip a scheduled WO.

---

## Files to Build

### GAS Project
| File | Purpose |
|---|---|
| `Code.gs` | Web app entry point, doGet(), route to views |
| `RouteBuilder.gs` | Core logic: load WOs, handle selection, call Route API |
| `SheetsConnector.gs` | Read WO data from Sheets, write route log |
| `RouteOptimizer.gs` | Google Route Optimization API call + response parser |
| `HelpFlag.gs` | Write flag to HelpFlags tab, trigger email to Jason |
| `Index.html` | Tech-facing UI (selection, pin, confirm, flag) |
| `Stylesheet.html` | Embedded CSS |

### Sheets Structure
| Tab | Purpose |
|---|---|
| `WorkOrders` | Source data (AppFolio export lands here) |
| `RouteLog` | Confirmed daily routes per tech |
| `TechList` | Tech names for dropdown |
| `Config` | API keys, depot address, default start time, Jason's email |
| `HelpFlags` | All flagged WOs: Tech, WO ID, Reason, Note, Timestamp, Resolved (Y/N) |
| `ManagerView` | Auto-generated daily load summary per tech for rebalancing |

---

## CLI Handoff Instructions

Paste the following into Claude CLI to begin build:

```
I have a completed spec for a GAS Web App called gpm-route-builder.

Stack: Google Apps Script + Google Sheets + Google Route Optimization API

Single responsibility: Let GPM maintenance techs self-select daily work orders, 
pin urgent jobs to top of route, receive an optimized stop sequence, and flag 
WOs for help — with manager visibility and workload rebalancing logged to Sheets.

Build the following files in order:
1. SheetsConnector.gs — read WorkOrders tab, write to RouteLog and ManagerView tabs
2. RouteOptimizer.gs — call Google Route Optimization API, return ordered stops
3. RouteBuilder.gs — orchestrate selection logic, pinning, and API call
4. HelpFlag.gs — write flag to HelpFlags tab, trigger email to Jason
5. Code.gs — doGet() entry point, serve Index.html
6. Index.html + Stylesheet.html — mobile-first UI

Constraints:
- No auth in V1 (name dropdown only)
- Pinned WOs are fixed at top — not re-ordered by optimizer
- AppFolio data arrives via manual Sheets export (no live API in V1)
- Route confirmed by tech → written to RouteLog tab with timestamp
- Help flag triggers email to Jason automatically via GAS MailApp
- Manager reassigns WOs by editing Assigned Tech in WorkOrders tab only
- Tech-to-tech transfers not permitted — manager-only reassignment

Start with SheetsConnector.gs. Ask me for the exact Sheets column headers 
before writing any read/write logic.
```

---

## Open Questions Before Build

1. What columns does your current AppFolio WO export produce? (CLI needs exact headers)
2. What is the depot / start address for each tech? (fixed shop address or tech's home?)
3. Do techs have company phones with data, or is this WiFi-only?
4. Is estimated job duration available in AppFolio, or does someone need to assign it manually?
