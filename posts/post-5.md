---
title: Building It - Stack Decisions, Design Choices, and What Changed
post_number: Blog 5
date: 2026-05-10
author: Mehak Arya
summary: Moving from plan to prototype — why the stack choices were intentional, what the build revealed, and two design decisions that emerged only during implementation.
tags:
  - technical-decisions
  - implementation
  - design-decisions
  - stack
---

**Plans survive until you start building.** Blogs 1 through 4 mapped what Critique Canvas needed to do. This post is about what happened when we built it - technical decisions that
connected the stack to requirements, and two design decisions only visible once the prototype was running.

---

## Why this stack — argued, not assumed

The brief required MojoJS, HTMX, and SQLite. Each serves a
specific requirement — not just a course constraint.

**HTMX** exists for one precise reason: the annotation
interaction cannot survive a full page reload. When a reviewer
submits a comment, reloading resets their spatial position —
they lose track of where they were looking. HTMX partial
updates append the new pin without touching the rest of the
page. This is a functional requirement disguised as a
technical choice.

**SQLite** fits because annotations have a natural relational
structure. One artwork has many annotations. One user leaves
many across many artworks. A relational model handles this
with a foreign key — a document store like MongoDB would
require joining data manually on every request, a problem
SQLite solves natively.

**MojoJS** pairs with HTMX because it renders server-side
HTML partials natively. When HTMX fires after submission,
MojoJS returns a rendered HTML fragment — just the new pin.
React or Vue would have added client-side complexity without
solving any requirement these two tools do not already handle.

![Stack architecture diagram](/DECO2017/assets/post4-architecture.png)
*Figure 1: One user click travels through four layers and
returns as an HTML partial — canvas state preserved, no
full page reload.*

---

## What the build revealed — coordinate rendering

Blog 4 flagged whether percentage-based coordinates would
hold across screen sizes. Testing confirmed mostly yes — with
one edge case. When the browser resizes while an annotation
is visible, pins shift before snapping back. The percentage
calculation fires on initial render, not on resize.

Fix: a resize event listener recalculating pin positions when
the viewport changes. One function, no schema changes. The decision from Blog 3 was correct - it just needed one more case handled.

> A good architectural decision does not mean zero edge cases.
It means edge cases are **fixable without restructuring
everything.**

---

## Two design decisions from the build

**Decision 1: Progressive disclosure for secondary features**

As the homepage took shape, a question emerged: where do
secondary features live? The community could benefit from
features beyond critique — a collaboration board, a challenge
feed, or a resources section. But placing these in the top
navigation means every user sees them on every visit,
competing with the primary reason they came.

The decision was progressive disclosure — a dedicated community
page housing secondary features, accessed by choice. The top
navigation stays focused: Gallery, Upload, My Artworks.
The core interaction — annotate, critique, improve — stays
front and centre.

**Decision 2: The paintbrush cursor**

The login page felt functional but generic. A paintbrush
cursor effect was added: as the cursor moves, it leaves a
faint paint trail. This was not in any requirements document
— it emerged from using the prototype and noticing it felt
static. Minimal implementation time, immediate community
identity. Some decisions cannot be planned.

This reflects a principle the brief implies but doesn't state:
the community's identity should be legible before any
interaction begins. A paintbrush cursor communicates "this
is a creative space" in the same moment the user arrives.

![Login page with paintbrush cursor effect](/DECO2017/assets/post4-login.png)
*Figure 2: Login page — paintbrush cursor effect. Community
identity communicated before a single click.*

![Upload form showing file constraints](/DECO2017/assets/post4-upload.png)
*Figure 3: Upload form — JPEG or PNG, max 10MB. File size
constraint visible in the UI, connected to the persistence
requirement.*

![Annotation interaction mid-flow](/DECO2017/assets/post4-annotation.png)
*Figure 4: Annotation mid-flow — comment input open, category
selector visible. HTMX fires on submit, canvas state
preserved.*

---

## Reflection

The stack decisions connect to specific behaviours, not
convenience. The build confirmed most assumptions, revealed
one coordinate edge case, and surfaced two decisions that
could not have been made on paper. The prototype is stable.

> Building reveals what planning cannot. The goal now is
making sure what exists works **correctly for every user**,
not just the ones who use it the way we expected.