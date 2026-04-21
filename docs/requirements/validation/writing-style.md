# Requirements Writing Style

## Purpose

This guide defines the writing-style expectations for requirements documents in this repo.

Use it when:

- authoring new requirements
- editing existing requirements
- evaluating editorial findings during document self-consistency review

This guide is the authoritative source of truth for writing-style review. Other validation guides may restate selected rules for authoring convenience, but should not redefine them.

## Goal

A requirements document should read as one coherent spec written by one careful author. A builder or reviewer should not have to guess whether two names refer to the same thing, whether a sentence describes an observable requirement or an implementation suggestion, or whether a schema field is directly captured vs synthesized.

This guide is not a mandate to report every possible prose improvement. Its purpose is to help reviewers identify writing-style issues that materially improve comprehension, reduce future drift, or clarify implementation or scoring meaning.

## Writing-Style Rules

### 1. Use one canonical name per concept

If a concept has a primary name, use that name consistently across headings, prose, tables, prompts, and appendices.

Avoid avoidable drift such as:

- `Guided Conversation Component` vs `guided conversation module`
- multiple names for the same deferred future screen
- `BP Onboarding Component` vs `BP Onboarding flow` when the actor is meant to be the same thing

If two terms must coexist, define their relationship once explicitly.

### 2. Make actor ownership explicit

Normative sentences should make it clear who performs the action:

- user
- UI / component
- flow / orchestration layer
- LLM
- server
- background process

Do not let the acting subject drift casually between related terms unless the distinction is intentional and defined.

### 3. Use consistent formal labels for structured requirement patterns

When the document defines a formal pattern, use that pattern consistently everywhere it appears.

Examples:

- `Intentional Changes from Prototype` entry labels such as `**Net-new concept:**`
- similarly structured bullets across feature sections and appendices

If one section uses a formal labeled structure, do not let parallel sections fall back to informal prose unless that difference is intentional.

### 4. Prefer observable requirements over implementation mechanics

Normative text should describe the required outcome or contract unless a specific mechanism is intentionally normative.

Prefer:

- "The component is responsible for ensuring the LLM receives the completion-token convention."

Over:

- "The component must append a system instruction block to the prompt before sending it."

If a mechanism is only one workable implementation, move it to a non-normative implementation note.

### 5. Avoid vague triggers in normative text

Normative instructions should not rely on undefined phrases such as:

- `if needed`
- `when appropriate`
- `briefly`
- `near the end`

Unless the condition is made concrete nearby.

If a decision point matters to implementation, define the trigger explicitly.

### 6. Keep schema naming, topic wording, and field shape aligned

When a topic, field name, and field type all refer to the same concept, they should line up cleanly.

Watch for:

- plural field names with singular free-form types
- topic wording that implies a dedicated field but no field exists
- topic labels that drift from schema labels in ways that obscure the mapping

If a shape is intentional but non-obvious, say so in the Notes column or nearby prose.

### 7. Mark synthesized fields as synthesized

If some schema fields are derived from the overall conversation rather than directly elicited, say so explicitly.

Do not make a reader infer which fields are:

- directly captured
- aggregated
- synthesized during extraction

If needed, mark them in the Notes column or add one short explanatory note above or below the schema.

### 8. Define important reference terms once

If the document relies on a core reference term such as:

- `prototype`
- `current profile`
- `active profile`
- a named screen or flow

Define it once in a clear location so later sections can rely on that definition.

### 9. Keep parallel sections stylistically parallel

When multiple sections serve the same function, they should use similar structure and phrasing.

Examples:

- feature-level `Intentional Changes from Prototype` sections
- repeated validation instructions
- repeated route or lifecycle descriptions

This reduces accidental drift and makes reviews easier.

## Severity Guidance

Most writing-style issues are `Editorial`, not `Blocking`.

Report writing-style findings only when they are high-signal and actionable. Do not report:

- isolated wording preferences
- one-off phrasing alternatives that do not change meaning
- minor prose polish that a careful reader would not trip over
- synonymous naming variations that do not create likely confusion

In a passing review, writing-style findings should be selective rather than exhaustive. If the document is already usable as a build spec, prefer reporting only the most valuable editorial issues instead of cataloging every stylistic imperfection.

Escalate a writing-style issue only if it materially causes:

- wrong implementation behavior
- wrong schema interpretation
- wrong routing interpretation
- wrong scoring interpretation
- genuine reviewer disagreement about meaning

Otherwise, keep it editorial.
