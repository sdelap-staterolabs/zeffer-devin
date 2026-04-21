# Requirements Fidelity Review

## Purpose

This process assesses whether a feature implementation matches its requirements. The goal is a scored, behavioral evaluation — not a code review. The question is whether the user experience and data behavior match the intent of the spec, not whether the code is structured the way the spec describes.

This review is strict by default. A gap is a gap unless it is explicitly and narrowly covered by an "Intentional Changes from Prototype" entry. High scores must be earned by the implementation, not granted by documentation.

This review measures **observable behavioral and data outcomes** — whether data ends up in the right place, in the right shape, at the right point in the user flow, and whether the right user-facing states and transitions occur. The test is: could a user or QA tester observe this difference by using the product? Internal fragilities that require reading the code to identify — framework timing, async state sequencing, potential race conditions that don't produce observable differences in practice — are out of scope.

When a feature requirement describes a data outcome, completeness rule, or fidelity constraint, score the implementation against that outcome rather than against any particular internal extraction or transformation mechanism unless the requirement explicitly marks that mechanism as normative. For example, if a spec says saved profile data must reflect the full completed conversation, the score should evaluate whether the resulting saved data matches that requirement, not whether the implementation used one exact extraction call pattern internally.

When a requirements document explicitly marks some extraction fields as derived or synthesized from the overall conversation, score the implementation on whether the conversation gathers enough source material for those fields and whether the final saved outputs accurately reflect that material. Do not penalize an implementation solely because it does not ask the derived field as a direct question.

**The prototype is read-only.** It exists as a reference baseline and must never be modified to close a gap. When a gap is found, there are exactly two valid responses:

1. **Update the requirements** — if the prototype behavior is correct and the spec is wrong or incomplete, fix the spec.
2. **Add an Intentional Change entry** — if the requirements intentionally depart from the prototype and that departure is not yet listed, add a specific, narrow entry to the feature's "Intentional Changes from Prototype" section. Use the Overall Intentional Changes appendix only for truly document-wide decisions that apply across multiple features; it supplements feature-level entries and does not replace them for feature scoring.

Remediations Needed lists must only contain items of these two types. Any remediation that implies modifying the prototype source code is invalid.

## Intentional Changes from Prototype

Each feature spec may include an "Intentional Changes from Prototype" section. This section lists specific, deliberate decisions where the requirements intentionally depart from the prototype. Entries fall into three categories:

For feature scoring, the feature-level section is primary. The Overall Intentional Changes appendix may supplement it when a document-wide entry clearly and literally applies, but the overall appendix does not replace feature-level entries.

- **Rename** — an identifier, token, or label was deliberately renamed. Must name the old value and new value explicitly.
- **Net-new concept** — something exists in requirements that has no prototype equivalent because it is entirely additive. The entire concept is new; there is nothing in the prototype to compare against.
- **Architectural upgrade** — a specific mechanism was intentionally replaced with a better one. Must name both the old and new mechanism explicitly.

**What cannot be listed:**
- Vague or categorical entries ("session management is different")
- Bugs or incomplete implementations in the prototype
- Behavioral gaps that result from the prototype being unfinished rather than from a conscious decision
- Broad entries that shield entire subsystems from scrutiny

## How Intentional Changes Affect Scoring

Intentional Changes entries provide **narrow, literal exemptions** — not broad leniency. Apply them as follows:

- A **Rename** entry exempts only the specific identifier named. All behavior associated with that identifier must still match the spec.
- A **Net-new concept** entry exempts only the scoring of behavior that depends on that concept having a prototype equivalent. If the spec defines behavior for a net-new concept, that behavior is not scored against the prototype — it is the target, not a comparison.
- An **Architectural upgrade** entry exempts only the routing mechanism itself. All logic and behavior built on top of that mechanism must still match the spec and is fully subject to scoring.

**The exemption is literal.** If a gap is adjacent to, broader than, or only loosely related to a listed entry, it is not covered. When in doubt, do not grant the exemption — flag it as a missing entry instead.

**Documenting a gap does not excuse it.** A behavioral gap that is not covered by a valid Intentional Changes entry must still reduce the score, even if it is mentioned elsewhere in the document. The section is not a place to acknowledge known problems — it is a place to record approved decisions.

## Feature-by-Feature Scoring

This review is run per feature, in the order they appear in the Features table. Each feature falls into one of three categories:

- **Skipped** — Feature is listed in the Features table but has no spec yet. Record as "Skipped — no spec." Not included in the document composite.
- **New** — Feature has a spec but no prototype equivalent (covered by a net-new Intentional Changes entry for the entire feature). Score is 100/100 by definition.
- **Existing** — Feature has both a spec and a prototype. Scored per subsection, then rolled up to a feature composite.

### Subsection Scoring (Existing features only)

Each specced feature is broken into its natural subsections based on what the spec describes. Each subsection is scored 0–100 independently, then averaged into a feature composite. Use the canonical subsection names from `feature-template.md` when they apply. When the requirements document uses those canonical subsection names, use those exact subsection names in the score report. Typical subsections:

- **Behavioral Requirements** — Does the AI-driven behavior match the spec's persona, topic scope, turn rules, and completion condition?
- **Session Lifecycle** — Does routing, resume, and state initialization behave as specified?
- **Mid-Flow Persistence** — Is data saved at the right times and in the right shape?
- **Completion Handling** — Does post-completion extraction, storage, and navigation match the spec?
- **Layout** — Does the interface reflect the intended chrome, states, and transitions?

For each subsection:
1. List behavioral and UX gaps found in the prototype vs the spec.
2. Cross-reference each gap first against the feature's Intentional Changes from Prototype section, then against the Overall Intentional Changes appendix only if a document-wide entry clearly and literally applies. Apply exemptions literally and narrowly. The overall appendix supplements feature-level entries and does not replace them for feature scoring. If a gap seems like it should be listed but isn't, flag it as a missing entry — do not grant an implicit exemption.
3. Score 0–100 based on the remaining unexempted gaps.
4. If the score is less than 100, add a **Remediations Needed:** list. The prototype is read-only — remediations must be one of: (a) a specific update to the requirements spec, including new behavioral requirements or implementation concepts that production code should follow, or (b) a new Intentional Change entry to add. Never suggest modifying the prototype source code. Clarifying questions and product decisions that must be made before the spec can be updated are also valid remediation items.

## Overall Document Score

After all features are scored, compute a weighted composite. Weight each feature by its complexity multiplier from the Features table. Skipped features are excluded. New features count as 100.

## Scoring Prompt

Use the following prompt when asking an LLM to run a requirements fidelity review for an existing feature. Pass all file references as parameters. Substitute `[Feature Name]`, `[list subsections]`, `[REQUIREMENTS_DOCUMENT_PATH]`, and `[SCORING_GUIDE_PATH]` for the feature being evaluated, and attach the relevant source files along with `[REQUIREMENTS_DOCUMENT_PATH]`.

If the review needs to write a new score report or note, derive that output path relative to `[REQUIREMENTS_DOCUMENT_PATH]` rather than hard-coding a repo-specific location.

---

Review the requirements for **[Feature Name]** in `[REQUIREMENTS_DOCUMENT_PATH]` against the provided source code.

Apply the scoring rules defined in `[SCORING_GUIDE_PATH]`.

Score each of the following subsections from 0 to 100:

[list subsections]

For each subsection:
1. List all behavioral and data outcome gaps between the prototype and the spec. The test for every finding is: could a user or QA tester observe this difference by using the product? Flag gaps where data ends up in the wrong place, saved at the wrong time in a way that affects observable outcomes, has the wrong shape, or where user-facing states and transitions differ from the spec. Do not flag internal fragilities that require reading the code to identify — framework timing, async state sequencing, or potential race conditions that don't produce observable differences in practice. Do not flag structural, architectural, or organizational differences.
2. Cross-reference each gap first against the feature's "Intentional Changes from Prototype" section, then against the Overall Intentional Changes appendix only if a document-wide entry clearly and literally applies. Apply exemptions literally and narrowly — only the exact thing described is exempted. The overall appendix supplements feature-level entries and does not replace them for feature scoring. If a gap is adjacent to or broader than a listed entry, it is not covered. If a gap seems like it should be listed but isn't, flag it as a missing Intentional Changes entry rather than granting an implicit exemption.
3. State the score based on remaining unexempted gaps.
4. If the score is less than 100, add **Remediations Needed:** listing what must change to close to 100. The prototype is read-only — every remediation must be one of: (a) a specific update to the requirements spec, including new behavioral requirements or implementation concepts that production code should follow, or (b) a new Intentional Change entry to add. Clarifying questions and product decisions needed before the spec can be updated are also valid. Never suggest modifying the prototype source code.

Then provide a **feature composite score** (simple average of subsection scores).

If the feature composite is less than 100, add a **Feature Remediations Summary** consolidating the highest-priority items across all subsections.

---

## Scoring Guide

```
| Score     | Interpretation                                                               |
|-----------|------------------------------------------------------------------------------|
| 90–100    | Fully captures the spec's intent. Minor cosmetic gaps only.                  |
| 75–89     | Mostly correct. One or two behavioral gaps, none blocking.                   |
| 60–74     | Partial. Core behavior is present but meaningful gaps exist.                 |
| 40–59     | Significant gaps. Key behaviors are missing or wrong.                        |
| Below 40  | Major divergence. The feature does not meet the spec's intent.               |
```
