# GPM Tech Point System — Concept Note
**Status:** IDEA — not part of Route Builder spec  
**Date:** 2026-06-24  
**Link to:** gpm-route-builder (data source for WO completions and drive data)

---

## Concept

Weekly productivity scoring system for GPM maintenance technicians. Objective, data-driven, visible to management.

---

## Scoring Inputs (draft)

| Signal | Source | Notes |
|---|---|---|
| Work Orders Completed | RouteLog tab | Count per week |
| Hours Logged per WO | AppFolio or manual | Quality signal — too fast or too slow both flag |
| Driving Score | Route Builder (drive time vs. actual) | Efficiency vs. planned route |

---

## Open Questions Before Spec

1. Who sees scores — management only, or techs too?
2. What happens with the score? (Bonus, recognition, PIP trigger?)
3. How do you handle WO complexity variance? (A furnace replacement ≠ a light bulb swap)
4. Is driving score based on time deviation, mileage deviation, or both?
5. Weekly reset or rolling average?

---

## Risks to Flag Before Building

- Techs may rush WOs to inflate completion count if score is visible
- Hours logged accuracy depends on AppFolio discipline — currently unknown
- Driving score requires confirmed route vs. actual path comparison — needs GPS or mileage input
- Could create tension if scores feel punitive rather than developmental

---

## Dependencies

- Route Builder must be live and logging to Sheets first
- AppFolio WO hours must be reliably captured
- Jason buy-in required — he's the single point of contact for techs

---

## Status

Park until Route Builder V1 is stable. Revisit as a standalone spec.
