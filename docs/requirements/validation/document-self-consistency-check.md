# Document Self-Consistency Review

## Purpose

This process audits a requirements document for internal consistency. No source code is needed or relevant. A self-consistency review surfaces problems that would cause two careful readers to understand the document differently, or that would cause a builder to make wrong assumptions based on contradictions or ambiguities in the text.

## Validation Exit Criteria

This review is a **convergence gate**, not an open-ended search for every possible improvement. Its purpose is to determine whether a requirements document is internally coherent enough to use as a build specification for the document's current scope.

The review passes when there are **no blocking consistency findings** in the document's normative sections.

A feature section is not implementation-ready if a builder still has to invent product rules for current-scope input validity, error handling, or completion routing.

A finding is **blocking** if it would cause a careful reader, builder, or scorer to reasonably misunderstand the intended requirement. Blocking findings include:

- contradictions between normative sections
- broken or stale internal references
- mismatches between stated counts and actual lists
- mismatches between required topics, schemas, and lifecycle rules
- undefined or contradictory states in currently in-scope flows
- incomplete direct-input contracts that would let two careful builders implement materially different acceptance, rejection, or success behavior
- net-new feature sections whose core current-scope decisions are still guessable
- scoring-rule contradictions that make fidelity review ambiguous
- normative text that still depends on behavior explicitly deferred elsewhere

A finding is **not blocking** if it is clearly non-normative and does not conflict with the normative document. Non-blocking items include:

- editorial improvements that do not change meaning
- examples or implementation notes that are clearly labeled non-normative and do not contradict the spec
- future behavior explicitly deferred to a clearly non-normative location such as a todo appendix

A passing document is not required to be free of all editorial issues.

## Review Scope

For pass/fail purposes, prioritize the document's normative sections:

- feature requirements
- data model definitions
- lifecycle and persistence rules
- completion and extraction requirements
- validation and scoring rules
- Intentional Changes appendices where they affect scoring or interpretation

Treat clearly labeled non-normative content as non-blocking unless it contradicts a normative section.

Apply `docs/validation/writing-style.md` when evaluating writing-style and editorial consistency. Do not duplicate writing-style rules elsewhere in this guide.

## Finding Classification

Classify each finding as one of the following:

- **Blocking** — must be resolved before the document should be used as a build specification
- **Deferred** — intentionally out of scope for this version and clearly moved to a non-normative location
- **Editorial** — clarity improvement only; does not block use of the document

Only **Blocking** findings count against the summary verdict.

## Review Workflow

Run this review in batches, not as a continuous micro-loop:

1. Collect blocking findings from the current document snapshot.
2. Resolve them holistically, including all dependent references affected by each change.
3. Re-run the review against the updated snapshot.
4. Stop when no blocking findings remain.

If the same class of inconsistency keeps recurring, do not continue patching it locally. Refactor the document so that one section is authoritative and the others refer to it.

After no blocking findings remain, do not continue searching indefinitely for additional editorial findings. Report only high-signal editorial issues that materially improve comprehension, reduce likely future drift, or clarify implementation or scoring meaning.

## Review Checks

1. **Writing-style consistency** — Apply `docs/validation/writing-style.md` for terminology, actor clarity, normative wording, schema-expression clarity, and other editorial consistency checks.
2. **Cross-section contradictions** — Do any two sections make conflicting claims? This includes configuration tables vs. behavior descriptions, feature specs vs. the data model, and individual feature sections vs. the overall Intentional Changes appendix.
3. **Feature name accuracy** — Do feature names in the Features table exactly match the section headings for those features?
4. **Internal reference accuracy** — Do any references within the document point to something that does not exist or has been renamed?
5. **Intentional Changes correctness and completeness** — Do all documented departures appear as valid entries in either a relevant feature's "Intentional Changes from Prototype" section or the Overall Intentional Changes appendix?
6. **Scope boundary clarity** — Are there places where the document is ambiguous about what is in scope vs. out of scope?
7. **Validation fitness** — Can this document be scored cleanly using the validation and requirements fidelity process it defines?
8. **Example status and quality** — Are sample prompts, example schemas, and illustrative text clearly presented as examples rather than binding requirements when that is the intent?
9. **Enumerated set integrity** — Do stated counts and lists actually line up?
10. **Source precedence and consistency** — If a table, prose paragraph, and appendix all define the same thing, does the document make clear which is authoritative if they diverge?
11. **Direct vs. derived field clarity** — Does the document make clear which extraction fields are directly elicited versus synthesized from the conversation, and do the topic list and system prompt guidance provide sufficient coverage for both?
12. **State transition consistency** — Do lifecycle flags and statuses have valid entry and exit conditions with no conflicting transitions?
13. **Override and precedence rules** — If a global appendix rule and a feature-level rule both apply, does the document make clear whether the local section may narrow, override, or only supplement the global rule?
14. **Input contract completeness** — For features with direct user input, could two careful builders implement materially different acceptance, rejection, normalization, error, or success behavior because the document does not specify the input contract concretely enough?
15. **Net-new feature sufficiency** — For net-new features with no prototype equivalent, are the current-scope product rules concrete enough to implement without inventing missing behavior?

## Output Format

Structure the report using these sections in this order:

- `## Blocking Findings`
- `## Deferred Findings`
- `## Editorial Findings`

Always include all three sections. If a section has no items, write `None identified in this pass.` under that heading.

In a passing review, prefer zero to three editorial findings. Include more only when multiple editorial issues combine into a meaningful comprehension problem or the user explicitly asks for a deeper editorial pass.

For each finding:

- Present each finding as its own short section or numbered item with the metadata on separate lines. Do not collapse the full finding into one dense paragraph.
- The finding's name or heading must include a model abbreviation in parentheses identifying the model that produced it: `(C)` for Claude Code or Anthropic models, `(O)` for Codex or OpenAI models. For example: `Finding 3 (C) — Terminology drift between Features table and feature sections`.
- Write for a product manager or spec author who is trying to fix the document quickly.
- Use this line-by-line structure:
  - **Severity** — `Blocking`, `Deferred`, or `Editorial`
  - **Category** — which of the review checks above applies
  - **Location** — section name and approximate context
  - **Issue** — the inconsistency or ambiguity, stated plainly and specifically
  - **Remediation** — one or two sentences on how to fix it
- Keep the tone direct and readable rather than clinical.
- Separate the specific issue from the specific remediation onto different lines.
- Do not include isolated wording preferences, low-signal polish items, or editorial observations that are not worth the reader's time.

When updating an existing report, append new findings rather than replacing the file. Do not remove or rewrite findings authored by a different model unless the user explicitly asks for that.

End with a **summary verdict**: either `Document is internally consistent for its current normative scope.` or `N blocking findings require resolution before this document should be used as a build specification.`

## Reusable Prompt

Use the following prompt when asking an LLM to run a document self-consistency review. Pass all file references as parameters. Attach `[DOCUMENT_PATH]` only — no source code.

Before running the review, derive `[REPORT_PATH]` from `[DOCUMENT_PATH]`:

- place the report in the same directory as `[DOCUMENT_PATH]`
- keep the target filename stem
- append `-dc-report` before the extension

Example:

- `[DOCUMENT_PATH]` = `path/to/file.md`
- `[REPORT_PATH]` = `path/to/file-dc-report.md`

---

Review `[DOCUMENT_PATH]` for internal self-consistency. Do not look at any source code. Your job is to audit the document itself.

Apply the review rules defined in `[SELF_CONSISTENCY_GUIDE_PATH]`.
Apply the writing-style rules defined in `[WRITING_STYLE_GUIDE_PATH]` for editorial and wording-related findings.

Store the review result in `[REPORT_PATH]`.

Check for:
1. Writing-style consistency using `[WRITING_STYLE_GUIDE_PATH]`
2. Contradictions between any two sections
3. Feature names in the Features table that do not exactly match their section headings
4. Internal references that point to something nonexistent or renamed
5. Departures from prototype behavior acknowledged in the spec text but not formally listed in an "Intentional Changes from Prototype" section, and any listed Intentional Changes entries that are too broad, vague, or otherwise invalid under the scoring rules
6. Scope boundary ambiguity
7. Whether the document can be scored cleanly using its own validation and requirements fidelity process
8. Whether sample prompts, example schemas, and illustrative text are clearly examples when intended as examples
9. Whether counts and enumerated sets actually line up
10. Whether tables, prose, and appendices that define the same thing are both internally consistent and clear about which source is authoritative
11. Whether the document makes clear which extraction fields are directly elicited versus synthesized from the conversation, and whether the topic list and system prompt guidance provide sufficient coverage for both
12. Whether lifecycle flags and statuses have valid entry and exit conditions with no conflicting transitions
13. Whether the document makes clear how global rules and feature-level rules interact when both apply
14. For features with direct user input, whether the document specifies the input contract concretely enough that two careful builders would not invent different acceptance, rejection, normalization, error, or success behavior
15. For net-new features, whether core current-scope product rules are concrete enough that the section is implementation-ready rather than merely internally coherent

Present the report in three sections: `Blocking Findings`, `Deferred Findings`, and `Editorial Findings`. Include all three sections even if some are empty. For each finding, present the severity, category, location, issue, and remediation in a readable line-by-line format. Keep the issue and remediation on separate lines. Each finding's name must include a model abbreviation in parentheses identifying the producing model — `(C)` for Claude Code or Anthropic models, `(O)` for Codex or OpenAI models.

If the document has no blocking findings, do not keep searching indefinitely for additional editorial items. Report only the highest-signal editorial issues, and prefer zero to three editorial findings unless the user explicitly asks for a deeper editorial pass.

If the report file already exists, append your new findings to it. Do not remove or rewrite findings authored by a different model unless the user explicitly instructs you to.

End with a summary verdict: either `Document is internally consistent for its current normative scope.` or `N blocking findings require resolution before this document should be used as a build specification.`

---
