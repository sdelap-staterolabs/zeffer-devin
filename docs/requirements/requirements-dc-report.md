# Document Self-Consistency Review

Target document: `docs/notes/requirements.md`
Guides applied: `docs/validation/document-self-consistency-check.md`, `docs/validation/writing-style.md`

## Blocking Findings

None identified in this pass.

## Deferred Findings

None identified in this pass.

## Editorial Findings

### Finding 1 (C) — Topic 3's "where they are" has no dedicated extraction field

- **Severity** — Editorial
- **Category** — Enumerated set integrity / Writing-style rule 6 (keep schema naming, topic wording, and field shape aligned)
- **Location** — Section 3 "Topics Covered" item 3 (line 127) vs. extraction schema (lines 371–372).
- **Issue** — Topic 3 directs the conversation to cover "who they are, where they are, what pain they have," but the schema has `ideal_customer` (who) and `customer_pain_points` (pain) with no location-shaped field for "where." Two builders can reasonably disagree about whether location should be folded into `ideal_customer` or dropped, and two runs of extraction can produce divergent `ideal_customer` shapes as a result.
- **Remediation** — Either add a `customer_location` (or similar) field to the schema, or amend Topic 3 and the `ideal_customer` Notes cell to state explicitly that customer location belongs inside `ideal_customer`.

### Finding 2 (C) — `competitors` schema shape does not match its name or Notes

- **Severity** — Editorial
- **Category** — Writing-style rule 6 (schema naming / field shape alignment)
- **Location** — Section 3 extraction schema, `competitors` row (line 374), compared with sibling list-shaped fields `marketing_priorities` and `recommended_channels` (lines 384–385).
- **Issue** — `competitors` is plural and its Notes read "Ask if known; `null` if none are named or the owner skips it" — phrasing that implies multiple values — but the declared type is `string | null` while the other list-shaped fields use `string[] | null`. A reader and an LLM emitting structured output cannot tell whether the singular string is intentional (one free-form mention) or a type mismatch, and the two interpretations produce incompatible runtime shapes.
- **Remediation** — Either change the type to `string[] | null` to match the other list-shaped fields, or add a one-line Notes clarification that competitors are intentionally stored as a single free-form string (for example, to preserve the prototype's shape).

### Finding 3 (C) — "If needed, refresh `zeffer_active_profile_id`" leaves the trigger undefined

- **Severity** — Editorial
- **Category** — Writing-style rule 5 (avoid vague triggers in normative text)
- **Location** — Section 3 Session Lifecycle step 5 (line 329).
- **Issue** — Step 5 reads: "If needed, refresh `zeffer_active_profile_id` to that profile's id, then route the user to the stand-in Business Profile Dashboard page for that profile." "If needed" is not defined anywhere, so two builders can reasonably disagree about when the refresh must happen (always, only when absent, only on mismatch) — and a scorer cannot judge a specific implementation against the spec.
- **Remediation** — State the exact condition, for example: "If `zeffer_active_profile_id` is absent or does not match this profile's id, set it to that profile's id."

## Summary Verdict

Document is internally consistent for its current normative scope.

---

# Document Self-Consistency Review

Target document: `docs/notes/requirements.md`
Guides applied: `docs/validation/document-self-consistency-check.md`, `docs/validation/writing-style.md`

## Blocking Findings

None identified in this pass.

## Deferred Findings

None identified in this pass.

## Editorial Findings

### Finding 1 (O) — "Guided Conversation Component" and "module" terminology still drift in the reusable component section

- **Severity** — Editorial
- **Category** — Writing-style consistency
- **Location** — Section 3, `Guided Conversation Component` subsection, especially the opening paragraph, `System Prompt Contract`, and `Implementation Notes`.
- **Issue** — The reusable chat primitive is now mostly named `Guided Conversation Component`, but the same subsection still calls it a `chat UI module`, says "The module relies on the configured system prompt...", and refers to "One workable integration interface for this module...". A careful reader can still infer the intent, but the section does not yet use one canonical term consistently.
- **Remediation** — Standardize this subsection on `Guided Conversation Component` / `component` terminology throughout. Remove the remaining `module` references unless the document explicitly wants `module` as a defined synonym.

### Finding 2 (O) — Sections 1 and 2 still use informal Intentional Changes entries while later sections use formal labeled entries

- **Severity** — Editorial
- **Category** — Writing-style consistency / Source precedence and consistency
- **Location** — Section 1 `Accounts` Intentional Changes, Section 2 `Organizations` Intentional Changes, compared with Section 3 and `Appendix: Overall Intentional Changes from Prototype`.
- **Issue** — Section 3 and the Overall appendix use the explicit labeled format `- **Net-new concept:** ...` and `- **Architectural upgrade:** ...`, but Sections 1 and 2 still describe their Intentional Changes in plain prose. The meaning is understandable, but parallel sections no longer follow one shared structural pattern.
- **Remediation** — Rewrite the Section 1 and Section 2 Intentional Changes bullets using the same labeled format already used in Section 3 and the Overall appendix.

## Summary Verdict

Document is internally consistent for its current normative scope.
