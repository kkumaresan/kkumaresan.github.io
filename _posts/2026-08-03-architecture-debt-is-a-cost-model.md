---
title: "Architecture Debt Is a Cost Model, Not a Backlog"
description: Most organisations track technical debt as a list of deferred tasks. Treating it as a compounding cost model changes which decisions get funded.
date: 2026-08-03
tags: [architecture, finops]
---

> This is a scaffold post created during the v2 build — replace or delete it.

Most organisations track architecture debt as a backlog: a list of deferred
refactors, each with a rough estimate, each perpetually outranked by feature
work. The framing guarantees the outcome. A backlog item competes for capacity.
A cost model competes for budget — and budget conversations happen at a level
where architecture decisions actually get made.

## What the backlog framing hides

A deferred refactor has a carrying cost that compounds across three dimensions
at once:

- **Infrastructure** — over-provisioned services that were sized for an
  architecture that no longer exists.
- **Delivery** — every feature routed through the compromised boundary pays a
  coordination tax.
- **Risk** — the compliance and resilience surface widens quietly.

None of these appear on a ticket. All of them appear on a P&L.

## Reframing the conversation

<!--more-->

When I work with leadership teams on modernisation, the first artefact is
rarely a target architecture. It is a model that attaches a monthly number to
the current one. Once the carrying cost is visible, sequencing stops being an
engineering argument and becomes an economic one.

That shift is what makes modernisation fundable — and what keeps it funded
after the first quarter of work stops producing visible features.
