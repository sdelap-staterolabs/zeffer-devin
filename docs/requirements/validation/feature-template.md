# Feature Requirements Template

## Purpose

This guide prescribes how to author a feature requirements section so that it is structurally ready for writing-style review, self-consistency review, and downstream fidelity scoring.

A feature section is not implementation-ready if a builder still has to invent product rules for current-scope input validity, error handling, or completion routing.

Use this guide when:

- writing a new feature section from scratch
- restructuring an existing feature section to close blocking findings

The three review guides describe what a good feature section looks like *in retrospect*. This guide describes what to produce *up front* so the reviews converge quickly.

This is a clean-room authoring template. Requirements may be informed by existing code, prompts, screenshots, and prior documentation when reverse engineering current behavior. However, the final requirements must be authored as a clean behavioral specification: organized around intended user-visible behavior, data outcomes, lifecycle rules, and scope decisions, rather than around current implementation structure. If the requirements intentionally differ from the current implementation or prototype, document that in `Intentional Changes from Prototype`.

## How This File Fits With The Others

```
| File                                  | Role                                                     |
|---------------------------------------|----------------------------------------------------------|
| feature-template.md (this file)       | Authoring prescription — section structure and exemplars |
| writing-style.md                      | Prose and naming rules — authoritative for review        |
| document-self-consistency-check.md    | Coherence review — structural and cross-section checks   |
| scoring-check.md                      | Behavioral fidelity review against the prototype         |
```

This file may restate the most operationally important rules from the other guides for authoring convenience, but the other guides remain authoritative for review.

## Authoring Inputs

Before drafting a feature section, gather:

- product goal
- intended user flow
- intended saved data outcomes
- intended lifecycle / routing behavior
- intended completion behavior
- intended UI states
- scope boundaries for this version
- any approved intentional departures from prior systems or prototypes

These inputs may come from product planning, reverse engineering, or both. The requirement is that the final feature spec be anchored in behavioral decisions and observable outcomes rather than organized as a description of current implementation structure.

## Screenshots Guidance

Screenshots may be used to inform:

- visual layout
- labels and naming visible to users
- ordering of visible UI regions
- visible states such as loading, empty, success, or completion

Screenshots are not authoritative for:

- routing behavior
- persistence timing
- saved data shape
- background processing
- derived outputs
- system ownership or actor responsibilities
- hidden business logic

If a screenshot suggests behavior that is not explicitly stated in the requirements text, the text is authoritative unless the document explicitly says otherwise.

## Required Top-Level Section Structure

Every feature section must include, in this order:

1. **Feature heading** — `# N. Feature Name`. The heading must match the Features-table row exactly (writing-style rule 1, self-consistency check 3).
2. **Opening prose** — one or two short paragraphs describing what the feature is and what the user experience looks like at a glance. No implementation detail here.
3. **Scope In This Version** — a short bullet list that states what the feature does today, what it must produce today, and what future expansion is intentionally not part of this version.
4. **Data Outcomes** — a short section defining what gets saved, what does not get saved, which outputs are directly captured, which outputs are derived or synthesized, when they are saved, and what must be true after completion.
5. **Relationship To Adjacent Features** (only when relevant) — one or two sentences clarifying what this feature is not, and which adjacent feature owns the related behavior.
6. **Canonical Behavioral Subsections** — one subsection per dimension from the canonical list below. Include only the dimensions the feature actually has; do not invent empty ones.
7. **Out of Scope / Deferred** — a short section naming related future work, the role it will eventually play, and the fact that its detailed behavior is not defined here.
8. **Intentional Changes from Prototype** — a labeled bullet list using the three formal categories defined in `scoring-check.md`. Never plain prose.

Features that have no prototype equivalent (entirely net-new) still follow this structure, but the `Intentional Changes from Prototype` section contains a single `**Net-new concept:**` bullet covering the whole feature.

## Canonical Behavioral Subsections

`scoring-check.md` scores each feature by its natural subsections and rolls those scores into a feature composite. To make scoring unambiguous, use the canonical subsection names below when they apply. Name them exactly; do not paraphrase.

```
| Subsection Heading                 | Covers                                                                       |
|------------------------------------|------------------------------------------------------------------------------|
| Behavioral Requirements            | The feature's core behavior — inputs, actions, AI persona/prompt if any      |
| Session Lifecycle                  | Routing, resume, entry conditions, and state initialization                  |
| Mid-Flow Persistence               | What is saved during the feature's flow and when                             |
| Completion Handling                | What happens after the primary action — extraction, save, navigation         |
| Layout                             | Chrome, visual states, and UI transitions                                    |
```

Not every feature needs every subsection. A form-based generator like Campaign Creator may have:

- `Behavioral Requirements` — the brief form fields, the AI generation behavior, and the output shape
- `Completion Handling` — how the generated campaign is persisted and where the user lands afterward
- `Layout` — the brief form and the result view

A feature with no persisted state at all may skip both persistence-shaped subsections.

If a feature needs a behavioral dimension that is not in the table above, add a new subsection heading and describe what it covers at the top of that subsection. Do not hide novel dimensions inside a canonical subsection.

## Authoring Rules

### Scope In This Version is required

Every feature section must explicitly define what is in scope now:

- what this feature does today
- what it must produce today
- what future expansion is intentionally not part of this version

This prevents accidental implied scope.

### Out of Scope / Deferred is required

Every feature section must explicitly define:

- what related future work is expected
- what role that future work will eventually play
- that its detailed behavior is not defined here

This is especially important when screenshots or product discussions hint at broader future workflows.

### Data Outcomes is required

Every feature section must explicitly define:

- what gets saved
- what does not get saved
- which outputs are directly captured
- which outputs are derived or synthesized
- when they are saved
- what must be true after completion

This keeps a clean-room spec from drifting into vague UX prose.

### Current-Scope Decisions Must Be Concrete

For current-scope behavior, do not leave builders to infer product rules that the document could define directly.

For every user-entered field required in the current version, define:

- field name
- required vs optional status
- validation rules
- invalid conditions
- normalization rules, if any
- submit behavior
- visible error behavior

If those details are left unspecified, two careful builders can produce materially different implementations while both claiming to follow the document.

### Direct Input Contracts are required

For every direct user input in current scope, define:

- field name
- required or optional status
- accepted shape or format
- normalization rules
- validation rules
- rejection conditions
- visible error behavior
- what happens on successful submission

If the feature has user-entered data, the document should make acceptance, rejection, and success behavior concrete enough that a builder does not have to invent product rules.

### Actor Ownership is required

For each major action, specify who performs it:

- user
- UI/component
- orchestration flow
- LLM
- server
- background process

Do not leave important actions with ambiguous ownership.

### Observable Behavior Over Mechanism is required

Write what the system must do and what the user must observe.
Do not specify implementation structure unless the mechanism is intentionally normative.

This prevents the clean-room requirements from becoming implementation summaries.

### Minimum specificity is required for identity, form, and CRUD-like flows

For identity, form, setup, or CRUD-like flows, the spec must explicitly define:

- entry condition
- required fields
- validation and rejection rules
- success destination
- failure state
- retry behavior
- persistence timing

These flows are especially prone to hidden ambiguity because builders will otherwise fill in missing product decisions from habit.

### Direct vs. Derived Outputs is required

If a feature produces outputs that are synthesized or inferred:

- name them
- mark them as derived
- state what source material they come from
- do not imply they are directly elicited unless they actually are

### Canonical Naming is required

- each important concept gets one primary name
- if multiple terms must exist, define their relationship
- do not let screenshots, product language, and schema names drift independently

### Source of Truth for duplicated concepts is required

Whenever a concept appears in more than one place, identify one authoritative location.

Examples:

- topic list vs schema
- UI summary vs lifecycle rules
- table vs prose
- feature-level `Intentional Changes` vs overall appendix

### Dimension checklist

Before finalizing, confirm the feature has clear, observable, testable answers for each of these dimensions. Missing answers are the most common source of blocking findings.

- **Inputs** — what the user and the system must provide for this feature to act
- **Input contract** — for each direct user input, the accepted format, validation rules, rejection conditions, normalization rules, and success behavior
- **Actor ownership** — for each normative sentence, who performs the action (writing-style rule 2)
- **Persistence shape** — if anything is saved, what is saved, when, and in what schema
- **State transitions** — each lifecycle flag's entry and exit conditions (self-consistency check 12)
- **Completion outcome** — where the user lands after the primary action and what state is set on the server
- **Scope boundary** — which adjacent behavior is explicitly out of scope for this version
- **Non-normative content** — any example prompts, sample schemas, or illustrative text must be clearly marked `(Non-Normative)` (self-consistency check 8)

## Schema Row Exemplar

When a feature extracts, generates, or persists structured data, declare the schema as a fenced monospace table. Use `string | null`, `string[] | null`, or equivalent nullable types so the LLM or server can emit `null` for missing values. Mark synthesized fields explicitly (writing-style rule 7, self-consistency check 11).

```
| Field                   | Type             | Notes                                              |
|-------------------------|------------------|----------------------------------------------------|
| campaign_name           | string | null    | Directly captured from the brief form               |
| target_audience         | string | null    | Directly captured                                   |
| channels                | string[] | null  | Directly captured; multi-select                     |
| campaign_summary        | string | null    | 2-3 sentence summary; derived                       |
| suggested_tactics       | string[] | null  | 3-5 actionable tactics; derived                     |
```

Rules the exemplar enforces:

- plural field names use list-shaped types, never a single free-form `string` (writing-style rule 6)
- the `Notes` column either says `directly captured`, says `derived`, or gives a one-line shape hint
- empty `Notes` cells use `—`, never blank (self-consistency check 10)
- field names are `snake_case` and match any topic or prompt that references them exactly

If a plural concept genuinely stores as a single free-form string (for example, to preserve prototype shape), say so in the `Notes` column explicitly: `Stored as a single free-form string; non-list by design.`

## Intentional Changes From Prototype — Gold Exemplars

Every feature section ends with `## Intentional Changes from Prototype`. Use the three formal categories from `scoring-check.md` and nothing else. Each entry must name the specific mechanism or identifier it exempts and state what is *not* exempted.

The following are gold-standard entries. When authoring a new feature, write entries that match this shape.

### Rename (gold)

```
- **Rename:** The brief-form field previously labeled `audience` in the prototype is renamed
  `target_audience` in this document's schema. Only the field name is covered — the field's
  type, required/optional status, and downstream use in the generated campaign must still
  match the prototype behavior and are subject to scoring.
```

Why this is gold:

- names the old value (`audience`) and the new value (`target_audience`) explicitly
- narrows the exemption to the rename itself
- preserves scoring scope for everything else

### Net-new concept (gold)

```
- **Net-new concept:** Scheduled campaign launch is additive. The prototype generates
  campaigns for immediate use only; the requirements add a `launch_at` timestamp and a
  server-side scheduler that fires the launch at that time. All requirements behavior that
  depends on `launch_at` or the scheduler has no prototype equivalent by design — this is
  additive, not a gap in the prototype. All other campaign-generation behavior (brief form,
  generation pipeline, output shape) is not covered by this entry and must still match the
  prototype.
```

Why this is gold:

- names the new concept (scheduled launch, `launch_at`, scheduler) precisely
- explains why it has no prototype equivalent
- explicitly draws the exemption boundary so unrelated behavior is not shielded

### Architectural upgrade (gold)

```
- **Architectural upgrade:** Campaign draft persistence is specified as server-driven via
  the `campaigns` table's `draft_state` column. The prototype persists drafts in
  `localStorage` under `zeffer_campaign_draft`. The shift from `localStorage` to a
  server-backed `draft_state` column is an intentional production requirement. Only the
  draft-persistence mechanism itself is covered — the draft's shape, save cadence, resume
  behavior, and all other draft-related requirements must still match the spec and are
  subject to scoring.
```

Why this is gold:

- names the old mechanism (`localStorage` key `zeffer_campaign_draft`) and the new mechanism (`campaigns.draft_state` column)
- restricts the exemption to the mechanism swap
- keeps all behavior built on top of the mechanism in scope for scoring

### Anti-patterns

Do not write entries like:

- `- Persistence is different.` — categorical, no identifier, no exemption boundary
- `- We upgraded the form handling.` — no old/new mechanism named
- `- The prototype didn't finish this, so we redesigned it.` — documenting a prototype bug is not an Intentional Change

`scoring-check.md` will treat these as missing entries and penalize the score accordingly.

## Preflight Checklist

Before considering a new feature section complete, confirm:

- [ ] The section is written as a behavioral specification, not as a description of implementation structure, even when current code or prompts were used as discovery inputs
- [ ] Screenshots, if used, only inform visible UX and are not silently defining hidden behavior
- [ ] Feature heading matches the Features-table row exactly
- [ ] Opening prose describes observable behavior, not implementation
- [ ] Current scope is explicit
- [ ] Deferred scope is explicit
- [ ] Every major visible state is described
- [ ] Every direct user input has an explicit input contract: required/optional status, accepted shape, normalization rules, validation rules, rejection conditions, and visible error behavior
- [ ] Every saved or derived output is accounted for
- [ ] Direct vs derived outputs are explicitly marked
- [ ] Actor ownership is clear
- [ ] Duplicated concepts have an identified source of truth
- [ ] The section could be reviewed with self-consistency and scoring guides without guessing
- [ ] No builder would need to invent current-scope product rules for input validity, error handling, or completion routing
- [ ] Every canonical subsection the feature needs is present, with the exact heading name from the canonical table
- [ ] Each normative sentence has one clear actor (user, UI, component, flow, LLM, server, background process)
- [ ] Any schema is declared with nullable types; list-shaped fields use `T[] | null`; derived fields are marked `derived` in `Notes`
- [ ] Every lifecycle flag or status has explicit entry and exit conditions
- [ ] No vague triggers (`if needed`, `when appropriate`, `briefly`, `near the end`) in normative text
- [ ] Non-normative content is labeled `(Non-Normative)` where it appears
- [ ] `Intentional Changes from Prototype` uses one of the three formal labels for every entry
- [ ] Each Intentional Change entry names the specific identifier or mechanism and draws an explicit exemption boundary
- [ ] Any term the section relies on (`current profile`, `active profile`, a named screen) is defined once in the document

## Reusable Authoring Prompt

Use the following prompt when asking an LLM to author a new clean-room feature section from product intent and design references.

---

Write a clean-room requirements section for **[Feature Name]** to be appended to `[REQUIREMENTS_DOCUMENT_PATH]`.

Apply `docs/validation/feature-template.md` as the authoring prescription.
Apply `docs/validation/writing-style.md` as the prose and naming contract.
Cross-check the draft against `docs/validation/document-self-consistency-check.md` and `docs/validation/scoring-check.md` before returning it.

Inputs:

- Product goal and scope notes: `[PRODUCT_NOTES]`
- User flow notes: `[USER_FLOW_NOTES]`
- Data outcome notes: `[DATA_OUTCOME_NOTES]`
- Design references and screenshots: `[DESIGN_PATHS]`
- Approved Intentional Changes from Prototype: `[INTENTIONAL_CHANGE_NOTES]`

Produce the full feature section — heading, opening prose, Scope In This Version, Data Outcomes, any relationship paragraphs, every canonical behavioral subsection the feature requires, Out of Scope / Deferred, and a formal `Intentional Changes from Prototype` section. Use one canonical name per concept, write in observable behavioral terms, and mark all non-normative content as such. Treat screenshots as illustrative for visible UX only, not as authoritative for hidden behavior unless the notes say otherwise.

After drafting, run the preflight checklist at the end of `feature-template.md` against the draft and fix any item that is not satisfied before returning.

---
