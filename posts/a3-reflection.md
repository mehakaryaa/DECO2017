---
title: "Working vs. Working Well — A Final Evaluation of Critique Canvas"
post_number: Blog 7
date: 2026-06-01
author: Mehak Arya
summary: A final evaluation of Critique Canvas — what the data actually means, what four users taught me, and an honest reckoning with every requirement I wrote in planning.
tags:
  - reflection
  - evaluation
  - accessibility
  - performance
  - lessons-learned
---

There is a saying in software development that is so obvious it gets ignored: *shipping is not the same as done.*

This final post evaluates what was built — through Lighthouse audits across every page on desktop and mobile, a WCAG AA accessibility review, manual keyboard testing, contrast checking, zoom reflow testing, and observing four people use the prototype for the first time without any guidance. The biggest surprise was not discovering bugs. It was discovering how many assumptions felt correct during development and only became questionable once someone else used the application.

Critique Canvas — the **flagship feature** of the platform, a spatially-anchored critique system for artwork — began as the sole focus of the prototype. By the end of development, it had evolved into a three-feature platform alongside Collab Roulette and Story Chain. What changed most throughout development was not the idea itself, but my understanding of how quickly *"working"* and *"working well"* diverge once real users, real keyboards, and real mobile networks are involved.

![Project timeline showing five stages from planning through building, testing, discovering issues and planning improvements](assets/project-timeline.png)
*The full project journey — from five planned requirements to a tested, evaluated three-feature platform.*

---

## 1. Performance and Technical Behaviour

Performance was evaluated using **Chrome Lighthouse in Navigation mode** across all primary pages on both desktop and simulated mobile (Moto G Power, Slow 4G throttling). The slow 4G simulation was deliberate — the application serves user-uploaded artwork images, and image-heavy pages under constrained connections reveal how the system actually performs for real users, not just those with fast Wi-Fi.

| Page | Desktop | Mobile | Mobile LCP |
|---|---|---|---|
| Homepage | 100 | 97 | 2.0s |
| Upload Artwork | 100 | 98 | 2.0s |
| Collab Roulette | 100 | 98 | 2.0s |
| My Artworks | 100 | 95 | 2.4s |
| Story Chain | 98 | 81 | 4.8s |
| Gallery | 89 | 73 | 12.2s |
| Story Chain Archive | 77 | 72 | 26.4s |

![Performance bar chart comparing desktop and mobile scores across all pages](assets/performance-chart.png)
*Desktop vs mobile performance — pages without user-uploaded images score 95-100. Image-heavy pages score significantly lower on mobile.*

The application was designed **desktop-first** — the annotation canvas and two-column layout with sidebar prioritise larger screens — with responsive behaviour added to support mobile and tablet use. This context explains why desktop scores are consistently higher than mobile across image-heavy pages.

> Lighthouse scores tell part of the story. Watching someone wait for a page to load tells the rest.

Most pages score between 95-100 on both desktop and mobile. **Total Blocking Time was 0ms across every page** — no JavaScript blocks the main thread anywhere in the application. In practice, users can interact with annotation features even while images below the fold are still loading. That is a direct consequence of the stack decisions made in planning: server-rendered HTML, HTMX for targeted partial updates, no heavy client-side framework. The technical choices produced a measurably more interactive experience.

Beyond load times, the system was tested against a **range of inputs and error conditions**:

- Uploading an unsupported file type is rejected before writing to disk
- The Quotable API fetch is wrapped in a try/catch — if the API fails, the Story Chain still completes with a hardcoded fallback theme
- The 2-hour claim timer runs server-side, automatically deleting expired claims on each page load
- The sequential panel lock is enforced server-side — a crafted POST request to claim an out-of-order slot is rejected, not silently accepted
- Collab Roulette chat is disabled and messages are **permanently deleted from the database** after 24 hours

These behaviours were verified by attempting edge cases directly. The system behaved predictably in all cases.

> As a developer, it is tempting to celebrate that nothing breaks. Users, however, rarely distinguish between a bug and a 12-second wait. Both feel like friction.

The gallery scored 73 on mobile with a **Largest Contentful Paint of 12.2 seconds**. The Story Chain Archive scored 72 on mobile with an LCP of 26.4 seconds and a total network payload of 4,861 KiB.

![Lighthouse gallery mobile metrics and insights showing LCP 12.2s and image delivery warning](assets/lighthouse-gallery-detail.png)
*Lighthouse flags "Improve image delivery — estimated savings of 1,585 KiB." The root cause is clear: no image compression on upload.*

Importantly, this issue did not fully surface during local development — it only became apparent once the application was tested across multiple devices on a simulated slow connection. Both pages work correctly. But **correctness and performance are separate properties**, and the system failed on the second despite passing the first.

---

## 2. User Experience and Accessibility

### Evaluation methods

Seven evaluation methods were combined to produce a complete picture of how the application performs for real users.

![Evaluation methodology diagram showing three categories: performance testing, accessibility testing, and user testing](assets/testing-methods-diagram.png)
*A multi-method approach — automated tools, WCAG checks, and direct user observation. Each method surfaces different issues.*

Each method surfaced different issues: Lighthouse identified performance bottlenecks and structural accessibility gaps, manual keyboard testing revealed interaction issues Lighthouse could not detect, WCAG checks covered contrast ratios, zoom reflow at 200% (WCAG 1.4.10), focus visibility (2.4.7), keyboard operability (2.1.1), and responsive behaviour at 375px — exposing the orange button contrast failure and confirming reflow compliance, and user testing surfaced discoverability friction in the annotation interaction. No single method would have caught all of these — which is why the multi-method approach was necessary.

### User testing

Four friends tested the prototype in two ways: two structured tasks followed by a free exploration of the full site. While four participants is a small sample, the goal was not statistical validity but identifying major usability issues and points of confusion in first-time use. Four participants were sufficient to identify recurring interaction patterns, although a larger sample would likely reveal less common usability issues. All four participants successfully completed both Task 1 and Task 2 without assistance or intervention.

**Task 1 — Leave a critique on an artwork.** This tested the core annotation interaction — the most novel and potentially confusing interaction in the application.

**Task 2 — Upload your own artwork.** This tested the upload flow, form validation, and what happens when required fields are left empty.

After both tasks, testers explored the full site freely — trying Collab Roulette, setting interests, matching with users, chatting, submitting work, and attempting Story Chain panels.

![UX evidence to insight map showing user observations, patterns identified and design responses](assets/ux-evidence-map.png)
*Four users, three different paths to the same outcome. The variance reveals more than the result alone.*

Navigation from the homepage was immediate across all four testers. All moved to the gallery first — consistent with the visual hierarchy, which introduces the three features and surfaces the gallery as the primary entry point. The category filter pills, the medium tags, the artwork cards — none required explanation.

The **annotation interaction** required slightly more discovery. One tester went directly to the canvas, guided by the crosshair cursor on hover. One read the helper prompt first. Two instinctively reached for the comment sidebar on the right — the familiar pattern from most platforms — then noticed the hint above the canvas and redirected immediately. All four completed the annotation without guidance.

> Designing an interaction is one thing. Watching someone discover it for the first time is something entirely different.

The interaction is **not immediately obvious — but it is quickly learned.** That is an important distinction. An interaction that stays confusing is a design failure. One that takes thirty seconds to learn and then feels natural is a considered trade-off — particularly when the pattern is genuinely novel and cannot be replaced with something familiar without losing the core spatial quality that makes it work.

![Artwork detail page showing multiple spatially-anchored critique pins on an artwork with feedback in sidebar](assets/annotation-pins.png)
*Critique Canvas in use — pins anchored at exact percentage coordinates, feedback visible in the sidebar. The flagship interaction working as designed.*

User testing also revealed minor **friction points** that the interface handles correctly:

- One tester attempted to upload an artwork without entering a title — form validation caught it with a clear error
- Another tried to critique an artwork marked *closed for critique* — the closed state was clearly communicated and no annotation form was available
- A third tried to submit a Story Chain panel without a caption — the required field prevented submission with a visible prompt

In each case, the system caught the error and communicated it clearly.

One outcome I did not anticipate was the reaction to the in-browser drawing canvas. Unlike the annotation interaction, which required brief discovery, all four testers engaged with the drawing canvas immediately and intuitively — none required guidance. This suggests that familiar tool metaphors (brush, eraser, colour picker) significantly reduce cognitive load even when the feature itself is technically complex. The interaction pattern was novel in context but not in concept. That alignment between technical ambition and genuine user delight is what makes a feature feel like it belongs.

![Story Chain panel grid showing 7 of 8 panels submitted with draw or upload options visible](assets/story-chain-grid.png)
*Story Chain — sequential panels, 2-hour claim window, and an in-browser drawing canvas. Panel 8 locked until the previous panel is submitted.*

The Collab Roulette flow — spin, match, chat, submit — is coherent when tested across two sessions.
![Collab Roulette showing spin wheel and interest category preferences](assets/collab-roulette.png)
*Collab Roulette — interest-based smart matching, 24-hour creative challenge window, and responsible data handling through automatic chat deletion on expiry.*

### Responsible design

The 24-hour chat window closing automatically and deleting messages from the database is invisible to users when it works correctly. A cookie consent banner informs users on first login that only essential session cookies are used — no tracking, no advertising. That invisibility and transparency is exactly what responsible design should feel like.

### Accessibility

Accessibility was evaluated using a **multi-method approach**: Lighthouse automated auditing, manual keyboard navigation, WCAG contrast checking via WebAIM, zoom reflow testing, and responsive testing at 375px and 768px viewport widths.

| Page | Accessibility Score |
|---|---|
| Login | 96/100 |
| Homepage | 96/100 |
| Gallery | 96/100 |
| Story Chain | 96/100 |
| Collab Roulette | 95/100 |
| Story Chain Archive | 100/100 |

![Lighthouse accessibility audit showing 96 out of 100 on the gallery page with 21 passed checks](assets/lighthouse-accessibility.png)
*96/100 on the gallery page — the most complex page in the application. 21 checks passing including skip links, ARIA attributes, landmark regions, and touch targets.*

**Contrast** was checked using the WebAIM contrast checker. The primary orange button (#ff6b35 on white text) returned a ratio of **2.83:1** — below the WCAG AA requirement of 4.5:1 for normal text.

![WebAIM contrast checker showing 2.83:1 ratio and WCAG AA Fail for white text on orange](assets/wcag-contrast.png)
*White text on orange (#ff6b35) fails WCAG AA at 2.83:1. A deliberate brand decision — documented as an intentional trade-off, not an oversight.*

This was a deliberate brand decision. Orange is the single accent colour across the entire design system, and reducing its saturation would undermine visual coherence throughout. However, this highlights a real tension between branding and accessibility standards. In future iterations, a slightly darker variant of the same brand colour would likely preserve visual identity while achieving WCAG AA compliance. All other text elements passed contrast requirements.

At **375px mobile width**, the layout reflows correctly — horizontal sections stack vertically, content remains readable, meeting *WCAG 1.4.10 (Reflow)* and *WCAG 1.4.4 (Resize Text)*.

**Focus indicators** — orange rings on all interactive elements — are visible throughout and were verified through tab navigation across all primary flows.

![Orange focus ring visible on upload form input field when navigated via keyboard](assets/focus-ring.png)
*Visible focus indicators on all interactive elements — orange rings meet WCAG 2.4.7 (Focus Visible).*

Manual keyboard testing found one gap Lighthouse did not surface: the login page's custom user selector required `tabindex`, `role="radio"`, `aria-checked`, and `onkeydown` handlers to be keyboard-operable. Lighthouse validated structural accessibility but missed the interaction gap.

> **Automated tools test structure. Manual testing tests behaviour. Both are necessary. Neither alone is sufficient.**

Every form input is explicitly labelled. Every annotation has an `aria-label`. A skip link is present on every page. An `aria-live` region announces the cookie consent banner to screen readers. These were built in from the start — not added as compliance patches at the end.

---

## 3. Critical Reflection and Improvement Planning

Two failures in the build reveal something worth examining carefully — not because they broke the application, but because of what each exposes about *cause*.

### Failure 1 - Mobile responsiveness

A CSS file inherited from the Bla+Bla base repository — `bla-bla-anim.css` — contained a body block setting `min-width: 1024px` and `min-height: 768px`. These declarations **silently overrode every mobile breakpoint** in the custom stylesheet. The responsive layouts were written correctly. The inherited file prevented them from firing on smaller screens.

> *The fix itself took minutes. Finding it took weeks.*

That ratio became a recurring theme: implementation was often easier than diagnosis. **The real issue was not CSS — it was process.** Mobile testing was treated as a final check, not a continuous practice. If responsive behaviour had been verified at each feature milestone, this would have been caught in the first week of layout work.

### Failure 2 - Image performance

The gallery's 12.2-second mobile LCP and the archive's 26.4-second mobile LCP both trace to the same root decision: **no image processing on upload**. The fix is technically clear and scoped:

- **Sharp library** on the upload route — resize to max 1200px, compress before storage. Single addition to `src/fileStore.ts`, nothing else changes. Reduces file sizes 60-80%.
- **`loading="lazy"`** on archive panel images — prevents 4,861 KiB loading on page entry
- **`font-display: swap`** on Google Fonts import — recovers 750-1,800ms across every page

*Three fixes. Independent. Prioritised by root cause and impact.*

In hindsight, image compression should have been treated as a **functional requirement** rather than an optimisation task — it directly affects whether the application is usable on mobile, which is a core user need, not an enhancement.

> The lesson was not that optimisation matters. The lesson was that **performance decisions are made long before performance problems become visible** — and the gap between those two moments is only closed by testing.

### What succeeded - and why

The **annotation coordinate system** held across all tested screen sizes. The **HTMX partial update** preserved spatial context on submission. The **sequential panel lock** worked correctly under direct edge case testing. The **Quotable API fallback** worked when tested with a simulated failure.

These decisions succeeded because they were grounded in *specific, testable requirements* from the planning phase. Good architectural decisions survive contact with real users. These did.

The Story Chain model — one active chain, eight sequential contributors — worked as designed but exposes a **structural bottleneck** that only becomes visible at scale. The cause is architectural: one chain means eight contributors must coordinate sequentially before anyone else can participate. Parallel chains with different themes and pre-drawn first panels would address that structural cause directly — not by adding a feature, but by changing the architecture so the bottleneck no longer exists.

---

## 4. Retrospective Assessment of Functional Requirements

Blog 3 defined five core requirements: image upload and storage, click-to-annotate interaction, annotation storage and retrieval, pin rendering at correct positions, and HTMX inline updates without page reload. **All five shipped as specified and held under testing.**

In retrospect, the original requirements were **realistic and appropriately prioritised**. The core annotation workflow was implemented before any secondary feature was considered, which prevented scope expansion from compromising the primary user need. The sequencing decision — build the core first, validate it, then expand — was correct.

![Requirements assessment diagram showing three columns: successfully realised, misjudged, and incomplete requirements](assets/requirements-assessment.png)
*A retrospective view of every requirement — what held up, what was wrong, and what was missing entirely.*

What the requirements underestimated was **scope**. Blog 6 described Collab Roulette and Story Chain as features under consideration. Both shipped as complete features. In hindsight, the project was less constrained by the technology stack than by initial assumptions about what could realistically be built. But expanding scope without explicit out-of-scope statements made performance trade-offs invisible until they became problems.

> *Out-of-scope statements are not admissions of failure. They are honest predictions about where the first version ends — and they make trade-offs visible before they become problems.*

The one requirement most misjudged was **keyboard accessibility on the login page**. Blog 6 listed it as a future concern. It should have been a core requirement from Blog 3. The annotation canvas being pointer-dependent is an unavoidable design constraint — it was designed around from the beginning, with an accessible annotation list view as a parallel interface. The login page being keyboard-inaccessible was not a constraint. It was an oversight. One was deliberate. One was not.

The most honest reassessment is this: **requirements tested against specific user scenarios held up. Requirements assumed without evidence needed fixing.** This suggests the original requirements were specific enough to guide implementation but not comprehensive enough to anticipate the performance and accessibility concerns that emerged later.

### Evaluation limitations

The evaluation has limitations worth acknowledging. User testing involved four participants from a similar demographic, and all testing occurred within a controlled, familiar environment. Different audiences, assistive technology users, or longer-term use may reveal issues not identified during this evaluation. The findings should therefore be interpreted as evidence of major usability patterns rather than comprehensive validation. A follow-up evaluation with a broader participant group and screen reader testing would strengthen the accessibility claims in particular.

---

## Lessons Learned

The most valuable lesson from this project was that implementation is rarely the hardest part of development. Most technical problems were solved relatively quickly once identified. The greater challenge was recognising *which problems were worth solving — and when*.

Early planning focused on features and functionality. Through testing and evaluation, I learned that seemingly small decisions — image compression, keyboard navigation, onboarding cues, mobile responsiveness — often have a **larger impact on user experience than major architectural choices**. Users notice friction before they notice architecture.

Several significant issues only became visible after testing across real devices and with real users. This reinforced that **evaluation is not a phase at the end of development — it is a practice woven through every iteration**. The responsive bug, the login keyboard gap, the image performance problem — all were discoverable earlier. They were discovered late because evaluation was treated as a checklist, not a habit.

Looking further ahead, Critique Canvas has genuine potential beyond this prototype. The core spatial annotation model could extend to design critiques, UI reviews, or educational feedback on student work — any context where precise, location-based feedback matters more than general comments. Collab Roulette could evolve into a structured community challenge system with public submission galleries and community voting. Story Chain could support multiple simultaneous narratives with different themes, removing the sequential bottleneck that currently limits participation. These are not just feature additions — they are architectural evolutions that the current prototype is deliberately built to support. The foundation is stable. The direction is clear.

> That gap between plan and outcome is not a failure. It is evidence that requirements, prototypes, and assumptions only become meaningful once they encounter real users. More than any individual feature, that was the most valuable lesson from building Critique Canvas.