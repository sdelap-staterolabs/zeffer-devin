# UI Fidelity Review

## Purpose

This process reviews whether an implemented UI matches the intended visual and interaction design for a feature.

Its purpose is to evaluate **visible UX fidelity** — not hidden business logic, not code quality, and not requirements self-consistency. The question is whether the built screen looks and behaves, from the user's point of view, like the intended screen design.

Use this guide when:

- reviewing a built screen against approved screenshots
- reviewing a built screen against a feature section that includes explicit visual requirements
- checking whether an AI coding agent achieved sufficiently high UX fidelity during implementation

## Validation Exit Criteria

This review is a **convergence gate**, not an open-ended search for every cosmetic improvement.

A screen passes when there are **no blocking UI fidelity mismatches** in:

- page structure
- visible region order
- layout and hierarchy
- major visible states
- navigation affordances
- responsive behavior that materially changes usability

A passing screen is not required to be pixel-perfect.

A finding is **blocking** if it would cause a careful reviewer, designer, or implementer to conclude that the built UI materially differs from the intended experience.

Blocking findings include:

- missing or misordered major regions
- materially wrong layout structure
- incorrect or missing major navigation affordances
- missing required visible states
- responsive behavior that breaks the intended screen structure
- typography or spacing drift severe enough to flatten or distort the intended hierarchy

Non-blocking findings include:

- small spacing inconsistencies
- minor color or icon-size drift
- border-radius or shadow differences that do not materially change the intended hierarchy
- small typography differences that do not alter the visual structure

## Review Inputs

Use only the following inputs when running this review:

- the implemented UI
- the approved screenshots or design references for that feature
- the relevant feature section in the requirements document
- any explicit UI-specific notes in the requirements

Do not use the following as substitute sources of truth:

- assumptions from adjacent screens
- unstated product intent
- code structure as evidence of intended UX
- unrelated implementations elsewhere in the product unless the requirements explicitly say to follow them

## Source Precedence

When the review inputs differ, use the following precedence rules:

1. The requirements text is authoritative for:
   - hidden behavior
   - routing behavior
   - persistence behavior
   - actor ownership
   - any visible UX rule stated explicitly in the requirements
2. Approved screenshots are authoritative for:
   - visible composition
   - layout hierarchy
   - region order
   - component styling intent
   - typography hierarchy
   - spacing density
3. If multiple screenshots exist, the requirements should make clear whether they represent:
   - alternate states
   - desktop vs mobile
   - sequential scroll captures of one screen
   - exploratory references rather than final targets
4. If screenshots conflict and the requirements do not resolve the conflict, report that ambiguity as a review finding rather than guessing.
5. If the implementation matches the screenshot but violates an explicit UI requirement in the requirements text, that is still a finding.

## What to Check

1. **Page structure and region order** — Are the major visible regions present and arranged in the intended order?
2. **Layout and alignment** — Does the overall screen structure, column behavior, and alignment match the intended design?
3. **Typography hierarchy** — Do headline, section, body, label, and supporting text levels preserve the intended hierarchy?
4. **Spacing and density** — Does the implementation preserve the intended whitespace rhythm and screen density?
5. **Card, border, and container treatment** — Do cards, panels, dividers, borders, and background blocks match the intended visual grouping?
6. **Color and emphasis hierarchy** — Do colors, accents, and emphasis treatments preserve the intended visual priority?
7. **Icons and supporting visual cues** — Are icons, markers, badges, and small affordances present and used at the intended level of emphasis?
8. **Navigation affordances** — Are buttons, arrows, clickable cards, back paths, and entry points visually clear and placed as intended?
9. **State fidelity** — Do loading, empty, ready, success, error, disabled, and completion states match the intended visible behavior?
10. **Responsive behavior** — Does the UI preserve the intended hierarchy and usability on smaller screen widths?
11. **Scroll and sticky behavior** — If the screen design implies sticky headers, internal scroll regions, or document-style scrolling, is that behavior preserved?
12. **Screenshot interpretation accuracy** — Did the implementation correctly interpret the screenshots as visible UX references rather than inventing hidden behavior?

## State Review Rules

When a feature has multiple visible states, review them as separate fidelity targets rather than judging only the happy path.

At minimum, review any state that is relevant to the feature:

- loading
- empty
- ready
- success
- error
- disabled
- completion
- hover / focus / pressed, when clearly meaningful to the design

Do not assume that a ready-state match excuses missing or materially wrong supporting states.

If a screenshot set covers only one state, use the requirements text to determine which additional states are still required for the review.

## Responsive Review Rules

Responsive review is not optional when the screen is expected to work on both desktop and mobile.

Check at minimum:

- whether major regions collapse or stack in a usable way
- whether typography remains hierarchical rather than oversized or cramped
- whether cards or grids preserve readable density
- whether navigation affordances remain visible and usable
- whether sticky or scroll behavior creates overlap, clipping, or inaccessible content

A screen does not fail responsive review because it differs slightly from the desktop composition. It fails if the responsive behavior breaks the intended hierarchy, readability, or usability.

## Severity Guidance

Classify each finding as one of the following:

- **Blocking** — the implementation materially fails to match the intended visual experience
- **Notable** — the implementation is usable and directionally correct, but a meaningful fidelity gap remains
- **Polish** — small cosmetic drift only

Escalate to **Blocking** when the issue materially affects:

- visible structure
- hierarchy
- state comprehension
- navigation clarity
- responsive usability

Keep a finding as **Notable** when:

- the intended design is clearly recognizable
- the screen is usable
- but the visual treatment still differs in a way a careful reviewer would notice quickly

Keep a finding as **Polish** when:

- the intended design is already clearly achieved
- the issue is a small refinement rather than a structural mismatch

## Anti-Churn Rules

After no blocking findings remain, do not continue searching indefinitely for tiny cosmetic issues.

In a passing review:

- prefer zero to five findings total
- prefer zero to three `Notable` findings
- report `Polish` findings only when they are high-signal and easy to act on

Do not report:

- microscopic spacing differences that a typical user would not perceive
- color-value differences that do not change hierarchy or emphasis
- tiny border, radius, or shadow mismatches that do not affect grouping
- generic observations such as "could feel more polished"

The goal is to produce a useful implementation review, not an endless design-polish backlog.

## Output Format

Structure the report using these sections in this order:

- `## Blocking Findings`
- `## Notable Findings`
- `## Polish Findings`

Always include all three sections. If a section has no items, write `None identified in this pass.` under that heading.

For each finding:

- present each finding as its own short section or numbered item
- use this structure:
  - **Severity** — `Blocking`, `Notable`, or `Polish`
  - **Category** — which UI fidelity check above applies
  - **Location** — screen area and approximate context
  - **Expected** — what the screenshot and/or requirements indicate should be present
  - **Observed** — what the implementation currently does instead
  - **Remediation** — one or two sentences on how to close the gap

Keep findings concrete, visual, and easy for an implementer to act on.

## Reusable Prompt

Use the following prompt when asking an LLM to run a UI fidelity review. Pass all file references as parameters and attach the approved screenshots together with the implemented UI evidence.

If the review needs to write a report file, derive that output path relative to `[REQUIREMENTS_DOCUMENT_PATH]` rather than hard-coding a repo-specific location.

---

Review the implemented UI for **[Feature Name]** against the approved screenshots and the relevant feature section in `[REQUIREMENTS_DOCUMENT_PATH]`.

Apply the review rules defined in `docs/validation/ui-fidelity.md`.

Use the requirements text as authoritative for hidden behavior, routing behavior, persistence behavior, actor ownership, and any visible rule stated explicitly in the requirements.

Use the screenshots as authoritative for visible composition, hierarchy, styling intent, typography hierarchy, spacing density, and region order.

Do not infer hidden logic from the screenshots.
Do not use code structure as evidence of intended UX.

Check for:
1. page structure and region order
2. layout and alignment
3. typography hierarchy
4. spacing and density
5. card, border, and container treatment
6. color and emphasis hierarchy
7. icons and supporting visual cues
8. navigation affordances
9. state fidelity
10. responsive behavior
11. scroll and sticky behavior
12. screenshot interpretation accuracy

Present the report in three sections: `Blocking Findings`, `Notable Findings`, and `Polish Findings`. Include all three sections even if some are empty. For each finding, provide severity, category, location, expected, observed, and remediation.

If no blocking findings remain, do not keep searching indefinitely for tiny cosmetic issues. Prefer a short, high-signal report over an exhaustive polish list.

---
