# Introduction

Zeffer is an AI marketing platform built for small business owners who don't have a marketing team. It starts with a conversational onboarding interview — conducted by an AI acting as a Head of Marketing — that extracts the business's positioning, customers, voice, and goals in a single session. Everything that follows is generated from that foundation.

The platform covers the full marketing workflow in one place: brand identity (strategy, visual identity, brand assets), market intelligence (customer personas, competitive analysis), content production (social, email, campaigns, a 90-day content calendar), and performance tracking (campaign ROI, analytics, website optimization). A persistent AI Marketing Coach is available throughout, and a Daily Game Plan surfaces three specific tasks each day based on the business's active work.

## Data Model

The top-level object is an Organization — a company, agency, or individual operator. An organization may own zero or more Business Profiles over its lifetime. Each Business Profile represents a distinct brand or business unit with its own positioning, voice, customers, and marketing activity. This allows a single user or team to manage multiple brands (or client accounts) from one place without context bleeding between them.

The first-run onboarding UX requires creation of an initial Business Profile so the user becomes acclimated to the product, but that requirement is only a flow rule for first-run onboarding. It does not impose a permanent minimum profile count. An organization may later delete all Business Profiles and then create new ones.

For the current specced scope, first-run onboarding is the only defined Business Profile creation path, so an organization will normally have one profile immediately after onboarding completes. The future create/select Business Profile screen is a future feature. If an organization later has zero profiles, or if the product later needs profile-selection behavior beyond the single-profile case, routing hands off to that deferred future screen rather than re-entering first-run onboarding.

Each Business Profile is created through onboarding and progressively enriched by the platform's tools. Campaigns, products, uploaded brand assets, and performance results are child records linked to the profile — not the organization — so every brand stays self-contained.

---

# Features

The following features make up the full Zeffer product. Each entry notes its documentation status and a complexity multiplier relative to Business Profile Onboarding (1× = same complexity). Features marked **Specced** have full requirements in this document. Features marked **Skipped** are acknowledged but not yet specced — they will be added in future versions.

```
| Feature                  | Status   | Complexity | Notes                                       |
|--------------------------|----------|------------|---------------------------------------------|
| Accounts                 | Specced  | 0.5×       | Auth/registration flow                      |
| Organizations            | Specced  | 0.5×       | Single naming step with first-run flag      |
| Business Profile Onboarding | Specced  | 1× (base)  | LLM chat loop, persistence, extraction      |
| Dashboard                | Specced  | 1×         | Data aggregation, summary cards, game plan  |
| Brand Platform           | Specced  | 0.5×       | Generated brand strategy document and display |
| Visual Identity          | Skipped  | 1×         | AI-generated visual identity, regeneration  |
| Brand Kit                | Skipped  | 2×         | Multi-section brand kit assembly and export |
| Brand Assets             | Skipped  | 0.5×       | File upload and asset gallery               |
| Social Posts             | Skipped  | 0.5×       | AI post generation with platform selector   |
| Email Sequences          | Skipped  | 1×         | Multi-step email sequence generation        |
| Content Calendar         | Skipped  | 2×         | 90-day calendar grid, scheduling, posts     |
| Campaign Creator         | Skipped  | 0.5×       | Campaign brief form with AI output          |
| Saved Campaigns          | Skipped  | 0.5×       | List/detail view for saved campaigns        |
| Products                 | Skipped  | 1×         | Product catalog with AI descriptions        |
| My Company               | Skipped  | 0.5×       | Read/edit company profile fields            |
| My Market                | Skipped  | 0.5×       | Market intelligence display                 |
| My Customers             | Skipped  | 0.5×       | Customer persona display                    |
| Marketing Coach          | Skipped  | 2×         | Persistent AI chat with memory and context  |
| Analytics                | Skipped  | 1×         | Campaign ROI tracking, charts, performance  |
| Website Optimization     | Skipped  | 2×         | URL analysis, AI audit, recommendations     |
| Local Marketing          | Skipped  | 3×         | Multi-channel local strategy, maps, listings|
| Video Ad Creator         | Skipped  | 2×         | Script generation, storyboard, export       |
| My Tools                 | Specced  | 0.5×       | Tool directory and integrations list        |
```

---

# 1. Accounts

`Accounts` establishes the user's authenticated identity in Zeffer. In this version, account setup follows a standard username/password pattern rather than magic-link or social sign-in.

The account feature exists to let a user register, log in, log out, and re-enter the product as the same identity over time. It does not own company data, Business Profiles, or tool outputs. Those belong under the Organization root object after authentication succeeds.

## Scope In This Version

In this version, `Accounts` is responsible for:

- allowing a new user to create an account with a username and password
- allowing an existing user to log in with that username and password
- establishing an authenticated session for the user after successful login
- allowing the user to log out
- restoring the same authenticated identity on later visits until the session ends or expires

## Data Outcomes

`Accounts` creates and maintains the user's authentication identity.

In this version, the account record must store at minimum:

- `username`
- a password credential stored securely by the authentication system

`Accounts` does not create the Organization, Business Profile, or downstream marketing records. Those are separate data outcomes owned by adjacent features after authentication succeeds.

After successful account creation:

- the user must have a valid authenticated identity
- the system must be able to recognize that user on later logins with the same username/password pair

In this version, `username` is the canonical login identifier. Username matching is case-insensitive after normalization.

## Relationship To Adjacent Features

`Accounts` owns user identity and session establishment.

`Organizations` owns the top-level business object that the authenticated user works inside. Authentication success alone is not enough to enter the product's main workflow; the session must also resolve or create an Organization.

## Behavioral Requirements

In this version, account setup must support a standard credential flow:

- **Registration:** the user provides a username and password to create a new account
- **Login:** the user provides an existing username and password to enter the product
- **Logout:** the user can end the authenticated session

The system must treat one username/password combination as one account identity. Re-entering the same valid credentials on a later visit must restore that same identity rather than creating a duplicate account.

For registration in this version, the direct user inputs are:

- `username`
- `password`
- `confirm_password`

The registration input contract is:

- `username`
  - required
  - accepted format: 3-32 characters after normalization
  - allowed characters: letters, numbers, underscores, periods, and hyphens only
  - normalization: trim leading and trailing whitespace, then lowercase before uniqueness checks and storage
  - invalid if empty after trim, shorter than 3 characters, longer than 32 characters, contains disallowed characters, or is already in use
- `password`
  - required
  - accepted format: 8-72 characters
  - invalid if shorter than 8 characters or longer than 72 characters
- `confirm_password`
  - required during registration
  - invalid if it does not exactly match the submitted `password`

For login in this version, the direct user inputs are:

- `username`
- `password`

The login input contract is:

- `username`
  - required
  - normalization: trim leading and trailing whitespace, then lowercase before account lookup
  - invalid if empty after trim or if no account exists for the normalized username
- `password`
  - required
  - invalid if empty or if it does not match the stored credential for the resolved account

If registration fails because the requested username is unavailable or because the submitted credentials do not meet the current validation rules, the UI must remain on the account-setup surface and show a visible error message tied to the failed submission.

If login fails because the submitted credentials are invalid, the UI must remain on the login surface and show a visible error message tied to the failed submission.

If registration succeeds, create the account, establish the authenticated session immediately, and continue to the Organization resolution step.

If login succeeds, establish the authenticated session immediately and continue to the Organization resolution step.

If logout succeeds, end the authenticated session immediately and return the user to the login surface.

## Session Lifecycle

Before entering the authenticated product flow:

1. If the user does not have an authenticated session, route them to the account setup / login surface.
2. If the user chooses registration and completes username/password account creation successfully, establish an authenticated session for that new account.
3. If the user chooses login and submits valid existing credentials, establish an authenticated session for that account.
4. If the user already has a valid authenticated session, do not require them to re-enter credentials before resolving the next feature in the first-run flow.
5. After authentication succeeds, immediately resolve whether the account already has an Organization. If no Organization exists, route to `Organizations`. If an Organization exists, continue into the next lifecycle decision owned by `Organizations` and `Business Profile Onboarding`.

The account surface must not render authenticated product content before this Organization-resolution step completes.

## Completion Handling

Successful account registration or login completes only the identity step.

After successful completion:

1. The authenticated session becomes active for that account.
2. The system must resolve the account's Organization state before the user enters product content.
3. If the account has no Organization yet, route to the Organization naming step.
4. If the account already has an Organization, hand off to the Organization-owned lifecycle rules to determine whether onboarding or a post-onboarding destination should render.

If registration or login fails, remain on the current account surface, preserve the user's ability to correct the inputs, and allow immediate retry.

## Layout

In this version, the account surface must provide:

- a visible registration path for new users
- a visible login path for existing users
- input controls for `username` and `password`
- an input control for `confirm_password` on the registration path
- a primary submit action for the currently selected path
- a visible error state for invalid or rejected credentials

The exact visual styling is not defined in this document. The normative requirement is that registration and login are clearly distinguishable, credential entry is explicit, authentication errors are visible, and the user can retry without losing their place in the flow.

## Out of Scope / Deferred

The following are not defined by this `Accounts` section:

- social sign-in providers
- magic-link or passwordless authentication
- multi-factor authentication
- account linking across multiple auth methods
- password reset or account-recovery flows
- team membership, invitations, or role management

## Intentional Changes from Prototype

- **Net-new concept:** Account creation, username/password login, and authenticated user identity are additive. There is no registration flow, no login surface, and no user-account concept in the prototype. All requirements behavior that depends on authenticated user identity is new by design rather than a prototype gap.

---

# 2. Organizations

`Organizations` defines the top-level root object for the product. An authenticated user does not work directly at the account level; they work inside an Organization that owns Business Profiles and all downstream business data.

In this version, Organization setup is intentionally narrow. If the authenticated account does not yet have an Organization, the system must require creation of one before the user can enter Business Profile Onboarding, Dashboard, or any tool surface.

## Scope In This Version

In this version, `Organizations` is responsible for:

- defining the Organization as the top-level root object for business data
- requiring Organization creation immediately after login if none exists for the authenticated account
- collecting the Organization `name`
- persisting the Organization record before first-run onboarding begins
- storing the `initial_business_profile_created` lifecycle flag used by downstream first-run routing

## Data Outcomes

The Organization record is the top-level persistent business object for the authenticated account.

In this version, the Organization record must store at minimum:

- `name`
- `initial_business_profile_created`

`initial_business_profile_created` is `false` for a newly created Organization.

It is set to `true` only after the first Business Profile has been successfully finalized. Successful finalization means the Business Profile has been saved with extracted profile data, full conversation history, and `onboarding_complete: true`. It is not reset if that first Business Profile is later deleted.

The Organization record owns top-level business identity, but it does not store Business Profile fields, tool outputs, campaigns, products, or brand documents directly. Those remain child data linked beneath the Organization through Business Profiles.

## Relationship To Adjacent Features

`Accounts` owns user identity and authenticated session state.

`Organizations` owns the top-level business root object that the authenticated user enters next.

`Business Profile Onboarding` owns creation of the first Business Profile after the Organization exists. The Organization feature stops at successfully creating and naming the Organization, then handing off to onboarding or later Organization-driven routing.

## Behavioral Requirements

In this version, Organization setup must require exactly one direct user input:

- `name`

The `name` input contract in this version is:

- required
- accepted format: 1-80 visible characters after normalization
- normalization: trim leading and trailing whitespace and collapse repeated internal whitespace to single spaces before validation and storage
- invalid if empty after normalization or longer than 80 characters after normalization

The system must not let the user skip Organization creation when no Organization exists for the authenticated account.

The saved Organization name becomes the Organization's display name for downstream product context.

If the submitted Organization name is invalid, the Organization setup surface must remain visible and show a visible error message tied to the failed submission.

If the submitted Organization name is valid and persistence succeeds, create the Organization immediately and continue into first-run onboarding.

## Session Lifecycle

After account authentication succeeds:

1. Resolve whether the authenticated account already has an Organization record.
2. If no Organization exists, route the user to the Organization naming step immediately. Do not render Business Profile Onboarding, Dashboard, or downstream tools first.
3. If an Organization exists but does not yet have a valid saved `name`, keep the user on the Organization naming step until a valid name is saved.
4. If an Organization exists with a valid saved `name`, do not show the Organization naming step again during that session's first-run routing.
5. Once a valid Organization exists, use that Organization's `initial_business_profile_created` flag to determine whether to hand off into Business Profile Onboarding or the post-onboarding experience defined elsewhere in this document.

## Completion Handling

Successful Organization setup completes when the Organization record has been created and saved with a valid `name`.

After successful completion:

1. Persist the Organization record with the submitted `name`.
2. Set `initial_business_profile_created: false` for a newly created Organization.
3. Route the user into `Business Profile Onboarding` for first-run Business Profile creation.

If Organization creation fails, remain on the Organization naming step, present a visible error, and allow immediate retry with corrected or resubmitted input rather than advancing into onboarding.

## Layout

In this version, the Organization setup surface must provide:

- a visible explanation that the user is naming their Organization
- a single visible input for the Organization `name`
- a primary continue / create action
- a visible error state if save fails or the submitted name is invalid

The exact visual styling is not defined in this document. The normative requirement is that the page clearly communicates that Organization creation is required before entering the product, that naming is the only required Organization input in this version, and that invalid or failed submissions can be corrected and retried in place.

## Out of Scope / Deferred

The following are not defined by this `Organizations` section:

- multiple Organizations per authenticated account
- Organization switching
- Organization-member invitations, teams, or permissions
- Organization editing beyond the initial naming step
- Organization-level settings beyond `name` and `initial_business_profile_created`
- billing, plans, or account-to-organization administration flows

## Intentional Changes from Prototype

- **Net-new concept:** The Organization root object, its `name`, and its `initial_business_profile_created` flag are additive. The prototype has no Organization entity, no Organization naming step, and no Organization-level first-run routing state. All requirements behavior that depends on the Organization object is new by design rather than a prototype gap.

---

# 3. Business Profile Onboarding

The initial user experience flows from Organization naming into a broader first-run onboarding journey. In this version, the only specced onboarding phase in that journey is Business Profile Onboarding.

The Organization record's saved name and `initial_business_profile_created` flag determine whether first-run onboarding should run. If the Organization has a saved name and `initial_business_profile_created` is `false`, the application routes the user into the first Business Profile onboarding flow — either resuming a prior partial session or starting fresh. If the Organization has a saved name and `initial_business_profile_created` is `true`, first-run onboarding is skipped on all future logins regardless of device or browser state.

The Business Profile is created through a conversational chat UI — not a form. The conversation is designed to feel like a briefing with a senior marketing advisor, not a data entry exercise.

Reference designs are provided in `./design` — see the `onboarding*.png` files for the intended visual layout and flow.

## Scope In This Version

Business Profile Onboarding is the current first-run onboarding phase. Its purpose in this version is to:

- create the initial Business Profile
- collect the minimum business, customer, brand, and marketing inputs needed to generate that profile
- finalize the profile so the rest of the product can use it as the starting business context

## Relationship To Broader Onboarding

Future onboarding revisions may add later onboarding phases that enrich the Business Profile after this initial creation step. Those later phases may cover additional setup such as deeper branding inputs, brand assets, or other profile-enrichment work.

Those future phases are not part of the current Business Profile Onboarding requirements unless they are explicitly specced in a later revision of this document.

## Topics Covered

The conversation must cover all items listed below, in natural conversational order:

1. Business basics: name, type, industry, location, years in business, number of employees, revenue range
2. Products and services offered
3. Ideal customer: who they are, where they are, what pain they have
4. Competitive differentiator and known competitors, if any
5. Brand voice and values
6. Current marketing channels that have already worked
7. Marketing goals and biggest challenge
8. Revenue target for the next 12 months
9. Monthly marketing budget

---

## Guided Conversation Component

### Purpose

A reusable, domain-agnostic chat UI module that drives a goal-oriented conversation with an LLM to completion. It is intended for use across multiple product workflows and must not hard-code business-profile-specific behavior.

This component is designed for reuse — Business Profile Onboarding is its first consumer, but other guided conversations in the application should be able to use the same component with a different system prompt.

### Behavioral Requirements

The component must support the following inputs:

- **Conversation definition input**
  - A system prompt containing the conversation's persona, outcome, topic scope, turn rules, quick reply usage, tone/style, and opening behavior
- **Runtime / integration inputs**
  - A completion callback that receives the full conversation history when conversation completion is detected
  - An optional per-turn callback that receives the full conversation history after each assistant turn
  - Optional initial conversation history for resume behavior
  - Optional input lock state such as `disabled` or `isReadOnly` so the host flow can prevent further user input during save, completion, or navigation states
- **UI configuration inputs**
  - Configurable placeholder text for the input field

When prior conversation history is provided, the component must restore that conversation without generating an opening AI turn. When no prior conversation history is provided, the component must generate the opening AI turn based on the configured system prompt.

The component must remain domain-agnostic. Domain-specific conversation behavior belongs in the configured system prompt rather than in hard-coded component logic. Runtime, persistence, and UI wiring remain host responsibilities.

### System Prompt Contract

The system prompt is the component's only domain-specific conversation-definition input. It must define:

- **Persona** — who the AI is in this conversation
- **Outcome** — what the conversation is trying to produce
- **Topic scope** — what subject areas to cover
- **Turn rules** — e.g., one question at a time, probe vague answers, acknowledge before advancing
- **Quick reply usage** — when to offer options and in what format
- **Tone / style** — how the AI should sound
- **Opening behavior** — how the conversation should begin on a fresh start

The module relies on the configured system prompt to define these behaviors rather than enforcing domain-specific rules itself.

The component must append a standardized system instruction block to the provided system prompt before sending it to the LLM. For now, that appended block must define the shared completion convention:

`When you determine that the conversation has reached its intended outcome, end your final assistant message by emitting the exact token CONVERSATION_COMPLETE.`

Host-provided system prompts must not redefine or override this completion token convention.

### Quick Reply Format

The LLM appends suggested options at the end of a message using:

```
[OPTIONS: option1 | option2 | option3]
```

The component must strip the tag before display and render the options as tappable chips.
Selecting a quick reply must behave exactly like submitting typed user input.

### Display Rules

- Strip the completion token from message content before rendering.
- Filter out messages that are empty after stripping.
- Hide quick replies and the chat input after completion is detected.
- Show a spinner or completion indicator while the completion state is being displayed.

### Implementation Notes (Non-Normative)

The following notes are advisory implementation guidance only. They are not normative requirements and must not be scored in a requirements fidelity review.

One workable integration interface for this module is:

```
| Prop              | Type     | Description                                                                        |
|-------------------|----------|------------------------------------------------------------------------------------|
| systemPrompt      | string   | LLM instructions: persona, goal, topics, turn rules, reply format, and opening.   |
| onComplete        | function | Callback invoked with messages[] when `CONVERSATION_COMPLETE` is detected.         |
| onTurnComplete    | function | Optional. Invoked after each AI turn with the current messages[]. Use to persist   |
|                   |          | conversation progress so sessions can resume after interruption.                   |
| initialMessages   | array    | Controls start behavior. If provided with content, pre-populates the conversation  |
|                   |          | and makes no AI call on mount (resume mode). If empty or absent, the component     |
|                   |          | fires an opening AI turn on mount (fresh-start mode). The system prompt is         |
|                   |          | expected to include an opening instruction for the LLM to follow.                  |
| disabled/readOnly | boolean  | Optional. Locks user input while the host flow is saving, completing, or routing.  |
| placeholder       | string   | Input field placeholder text.                                                      |
```

Because `systemPrompt` is free-form text, a workable implementation should validate it at runtime before first use. At minimum, validate that it includes clear instructions for persona, outcome, topic scope, turn rules, quick reply usage, tone/style, and opening behavior. The code should also include comments or prompt-template guidance so human authors and AI code generators produce prompts that follow a consistent structure.

One workable internal state shape is:

```
| Variable         | Description                                                                    |
|------------------|--------------------------------------------------------------------------------|
| messages[]       | Full conversation history. Each entry: { role: 'user'|'assistant', content }  |
| isTyping         | True while awaiting LLM response                                               |
| quickReplies[]   | Parsed options from the last AI response, cleared on next user turn            |
| isComplete       | True once `CONVERSATION_COMPLETE` is detected; hides input UI                  |
```

One workable turn-loop implementation is:

`sendAIMessage(currentMessages)`

1. Set `isTyping = true`, clear `quickReplies`.
2. Build a transcript from `currentMessages`, formatted as labeled turns.
3. Call the LLM with: `systemPrompt` + appended component completion instruction + formatted transcript.
4. Parse response for `[OPTIONS: a | b | c]` tag — extract and split on `|`, strip tag from displayed content.
5. Append `{ role: 'assistant', content }` to `messages`.
6. Set `isTyping = false`. If options are present, set `quickReplies`.
7. If content includes `CONVERSATION_COMPLETE`: set `isComplete = true`, call `onComplete(updatedMessages)`.

`handleUserMessage(text)`

1. Append `{ role: 'user', content: text }` to `messages[]`.
2. Clear `quickReplies`.
3. Call `sendAIMessage(updatedMessages)`.

All rendering concerns may be delegated to sub-components such as `ChatMessage`, `TypingIndicator`, `QuickReplies`, and `ChatInput`.

### Example System Prompt (Non-Normative)

The following illustrates the pattern with a completely different domain — a mock behavioral interview coach. The persona, topic scope, turn rules, and completion payload all differ, but the component wiring is identical.

```
You are an expert career coach conducting a mock behavioral interview to help a
candidate prepare for a software engineering role.

Your goal is to assess the candidate's ability to articulate their experience
clearly using the STAR method (Situation, Task, Action, Result), then give them
actionable feedback on each answer and an overall coaching summary.

AREAS TO COVER:
1. A time they handled a technically complex problem under pressure
2. A time they disagreed with a teammate or manager and how they resolved it
3. A time they led or influenced a project without formal authority
4. A time they failed or made a mistake and what they learned
5. Their proudest technical achievement and why it mattered to the business

CRITICAL RULES:
* Ask ONE question at a time. Never stack questions.
* After the candidate answers, give brief, specific feedback on what landed well
  and what was missing. Then move to the next question.
* If an answer is vague or skips a STAR component, probe once before moving on.
* Keep your tone encouraging but honest.
* After all five areas are covered, give a 3–4 sentence overall coaching summary.

QUICK REPLY FORMAT: When a quick reply would help orient the candidate, append:
[OPTIONS: option1 | option2 | option3]

Start by introducing yourself warmly and asking which role or company they are
interviewing for.
```

---

## BP Onboarding Component

### Purpose

The BP Onboarding Component implements the business profile onboarding flow using the reusable Guided Conversation Component. It is responsible for onboarding-specific behavior: session lifecycle, mid-conversation persistence, profile extraction, and onboarding chrome.

In this subsection, the **BP Onboarding flow** means the onboarding orchestration implemented by the BP Onboarding Component. Use **BP Onboarding Component** when referring to the UI/component layer itself, and **BP Onboarding flow** when referring to the overall runtime behavior for this onboarding phase.

### Behavioral Requirements

#### System Prompt

Pass the following behavioral configuration to the Guided Conversation Component:

- **Persona:** Warm, sharp, experienced marketing advisor — Zeffer's AI marketing strategist. Sound like a brilliant Head of Marketing, not a form.
- **Outcome:** Guide the owner through a conversation that produces enough business intelligence to generate a complete brand and marketing profile.
- **Topic scope:** Cover all items listed in the Topics Covered section above, in conversational order.
- **Profile coverage:** Gather enough information to populate both the directly captured profile fields and the fields that are later synthesized from the full conversation during extraction.
- **Competitor handling:** Ask about competitors or alternatives if the owner knows them, but do not force an answer. If none are provided, `competitors` may remain `null` in extraction.
- **Turn rules:**
  - One question per message — never compound questions
  - If an answer is thin or vague, probe with one follow-up before advancing
  - After a substantial answer, briefly acknowledge it before asking the next question
  - Target 15–20 exchanges total
- **Quick replies:** Offer 2–3 options when they reduce friction. Use the `[OPTIONS: …]` format.
- **Completion criteria:** After all areas are covered, tell the owner you have everything you need and are building their profile. The Guided Conversation Component will append the standardized `CONVERSATION_COMPLETE` instruction automatically.
- **Opening:** Introduce yourself warmly as their new AI marketing department, then ask for their business name.

#### Session Lifecycle

Session routing is **server-driven** — the source of truth is the Organization record, not client-side storage. Client-side storage may be used to track the *active* profile after onboarding completes, but it must never be used to decide whether onboarding itself should run.

Before rendering the onboarding conversation, resolve the session state by querying the server:

1. Check whether an Organization record exists and whether it has a saved name.
2. If no Organization exists, or if the Organization exists but does not have a saved name, redirect to the Organization naming step immediately. Do not render onboarding.
3. If the Organization has a saved name and `initial_business_profile_created` is `true`, do not route the user back into first-run onboarding.
4. If `initial_business_profile_created` is `true` and the Organization has zero Business Profiles, route the user to the future create/select Business Profile screen. That screen is intentionally deferred and out of scope for this document, but it is the authoritative destination for the zero-profile post-onboarding state.
5. If `initial_business_profile_created` is `true` and the Organization has exactly one Business Profile, treat that profile as the current profile. If `zeffer_active_profile_id` is absent or does not match that profile's id, set it to that profile's id, then route the user to the Dashboard for that profile.
6. If `initial_business_profile_created` is `true` and the Organization has more than one Business Profile, route the user to the future create/select Business Profile screen until broader profile-management behavior is specced.
7. If the Organization has a saved name and `initial_business_profile_created` is `false`, look up the most recently created incomplete Business Profile for this org (where `onboarding_complete` is `false`).
8. If one exists and it has saved conversation messages, restore that conversation without making an opening AI call.
9. If one exists but has no saved messages, start fresh with an AI-initiated opening message.
10. If no incomplete Business Profile exists, create a new Business Profile record marked incomplete, then start fresh with an AI-initiated opening message.

On completion, set `initial_business_profile_created: true` on the Organization record only as part of the same successful completion path that finalizes the Business Profile.

Retain the profile identifier for use in persistence and completion handling. For this version, `zeffer_active_profile_id` is only a convenience for returning to the current profile in the single-profile case. It is not authoritative for onboarding gating, and it must not be treated as the sole determinant of post-onboarding routing when the server can see zero, one, or many Business Profiles.

The forced creation of an initial Business Profile is a first-run onboarding rule only. After first-run onboarding has been completed, an Organization may later have zero Business Profiles if they are all deleted. That zero-profile state does not re-enable first-run onboarding.

Long-term product note: the future create/select Business Profile screen will handle organizations that have already completed first-run onboarding and later have zero Business Profiles, along with broader current-profile selection behavior such as selecting among multiple profiles. That future screen is the intended destination for those states, but its UI and behavior are out of scope for this document.

#### Mid-Conversation Persistence

Guided conversations must be saved as they progress so that a session can be resumed after any interruption. The BP Onboarding flow must persist the current conversation state to the Business Profile record after each AI turn — including the completion turn, before profile extraction begins. This allows the user to close or refresh at any point and return to exactly where they left off.

#### Completion Handling

Each Business Profile stores an `onboarding_complete` boolean.

- New Business Profiles are created with `onboarding_complete: false`.
- `onboarding_complete` remains `false` while onboarding is in progress.
- It is set to `true` only after the extracted profile data and full conversation history have been successfully saved to the Business Profile record.
- It is not reset under the current spec.

When `CONVERSATION_COMPLETE` is detected, the BP Onboarding flow must derive a structured profile object from the completed conversation. The extraction input must reflect the full completed conversation, not only the final turn or a partial transcript. The normative requirement is the resulting data outcome: the saved Business Profile must accurately reflect the completed conversation in the specified schema. The exact extraction mechanism is an implementation detail unless a future revision of this document states otherwise. Use `null` for any field not mentioned in the conversation. In this schema, `current_channels` means the channels the owner says have already worked for the business, and `competitors` may be `null` if the owner skips that topic or does not identify any. Fields marked as derived are synthesized from the overall conversation during extraction rather than elicited as direct questions. Fields to extract:

```
| Field                   | Type           | Notes                               |
|-------------------------|----------------|-------------------------------------|
| business_name           | string | null  | —                                   |
| business_type           | string | null  | —                                   |
| industry                | string | null  | —                                   |
| years_in_business       | string | null  | —                                   |
| location                | string | null  | —                                   |
| num_employees           | string | null  | —                                   |
| revenue_range           | string | null  | —                                   |
| business_description    | string | null  | 2-3 sentences; derived              |
| products_services       | string | null  | —                                   |
| ideal_customer          | string | null  | Detailed description                |
| customer_location       | string | null  | Where the ideal customer is located |
| customer_pain_points    | string | null  | —                                   |
| differentiator          | string | null  | —                                   |
| competitors             | string[] | null | List competitors if known; `null` if none are named or the owner skips it |
| brand_voice             | string | null  | —                                   |
| brand_values            | string | null  | —                                   |
| marketing_goals         | string | null  | —                                   |
| current_channels        | string | null  | Channels the owner says have already worked |
| monthly_budget          | string | null  | —                                   |
| biggest_challenge       | string | null  | —                                   |
| target_revenue_goal     | string | null  | —                                   |
| brand_summary           | string | null  | 2-3 sentence brand narrative; derived |
| positioning_statement   | string | null  | One sentence; derived               |
| marketing_priorities    | string[] | null | 3-5 actionable items; derived      |
| recommended_channels    | string[] | null | 3-5 best channels for this business; derived |
```

After extraction:

1. The BP Onboarding flow saves the extracted profile data and full conversation history to the Business Profile record, and sets `onboarding_complete: true` on that Business Profile.
2. If and only if the Business Profile finalization succeeds, the BP Onboarding flow sets `initial_business_profile_created: true` on the Organization record.
3. For this version, the profile that just completed onboarding is the current profile. The BP Onboarding flow marks this profile as the active profile in client-side storage (key: `zeffer_active_profile_id`).
4. After a brief delay (approximately 2.5 seconds, to allow the completion UI to display), the BP Onboarding flow navigates to the Dashboard for that profile.

The profile that just completed onboarding becomes the current profile for Dashboard purposes. The Dashboard defined in Section 4 is the destination both immediately after onboarding completion and on later first-run-complete visits when a current profile exists.

#### Layout

The BP Onboarding Component wraps the Guided Conversation Component with onboarding-specific chrome:

- **Header:** Zeffer logo + label reading "Your Marketing Intake Session" with an active/online status indicator
- **Progress banner:** Full-width strip reading "🎯 Answer honestly — Zeffer does the marketing thinking for you"
- **Completion state:** While the profile is being generated after the conversation completes, display a "Building your brand profile…" loading indicator until navigation occurs.
- **Input area:** The conversation input must be hidden once onboarding is complete.

### Implementation Notes (Non-Normative)

The following notes are advisory implementation guidance only. They are not normative requirements and must not be scored in a requirements fidelity review.

Current planned implementation uses a page-level BP Onboarding component that composes the Guided Conversation Component.

One workable integration approach is to:

- Pass onboarding behavior into the Guided Conversation Component via `systemPrompt`, `initialMessages`, `onComplete`, and `onTurnComplete`
- Use `initialMessages` to distinguish resume behavior from fresh-start behavior
- Persist the full conversation after each assistant turn through `onTurnComplete`
- Trigger extraction from `onComplete` when `CONVERSATION_COMPLETE` is detected
- Build an extraction transcript by labeling assistant turns as `Zeffer:` and user turns as `Owner:`
- Drive the completion loading state from the Guided Conversation Component's completion state once typing has stopped

## Intentional Changes from Prototype
- **Net-new concept:** The Organization entity and its `initial_business_profile_created` flag do not exist in the prototype. All requirements behavior that depends on the Organization entity (session lifecycle step 1, post-completion flag update) has no prototype equivalent by design — this is additive, not a gap in the prototype.
- **Architectural upgrade:** Session routing is specified as server-driven via the Organization record's saved name and `initial_business_profile_created` flag. The prototype uses `localStorage` (`zeffer_active_profile_id`) for this routing decision. The shift to server-authoritative state is an intentional production requirement. Only the routing mechanism itself is covered — all other session lifecycle steps (resume, create, start fresh) must still match the spec and are subject to scoring.
- **Architectural upgrade:** The extraction schema in production must declare nullable types (`string | null`, `string[] | null`) so the LLM can return `null` for fields not mentioned in the conversation. The prototype's schema declares non-nullable `string` and `array` types, which coerces missing values to `""` instead. Only the schema nullability declaration is covered — the production extraction schema must still match the extraction table above.

---

# 4. Dashboard

The Dashboard is the post-onboarding home surface for the current completed Business Profile. It gives the owner a fast read on what Zeffer knows, what to do next, and where to go next inside the product without forcing them directly into a tool flow.

In this version, the Dashboard is a screenshot-backed summary-and-navigation rough-in rather than a full workspace for every downstream feature. It presents high-level status, today's suggested actions, and visible entry points into adjacent product areas, while leaving most downstream feature behavior to later sections. Reference designs are provided in `./design` — see the `dashboard*.png` files for intended visual hierarchy and layout.

## Scope In This Version

In this version, the Dashboard is responsible for:

- serving as the default post-onboarding home surface for the current completed Business Profile
- presenting a welcome summary for the current profile
- presenting a dismissible quick-start checklist
- presenting a momentum / wins summary when enough underlying activity exists
- presenting a daily game plan with exactly three suggested marketing actions for the current day
- presenting visible entry points to adjacent product areas, including `My Tools`
- allowing the user to switch the current Business Profile from the Dashboard chrome
- exposing a visible `Re-do intake` action that routes back into onboarding

In this version, the Dashboard is not responsible for becoming the source of truth for downstream feature completion. It may show shallow summaries or presentational status cues, but those remain derivative of other profile data or Dashboard-owned convenience state.

## Data Outcomes

The Dashboard does not create or finalize canonical Business Profile data of its own. It reads from the current completed Business Profile and from downstream feature outputs that already exist for that profile.

In this version, Dashboard-owned state is convenience state rather than canonical business state. Dashboard-owned convenience state may include:

- which completed Business Profile is currently active
- whether the quick-start checklist has been dismissed for the current profile
- the current day's game-plan task list for the current profile
- the current day's task-completion checkmarks for that game plan
- momentum indicators such as recent active-day streaks

This convenience state must not overwrite or redefine the canonical Business Profile, tool outputs, or downstream feature records. If Dashboard-owned convenience state is cleared, the underlying Business Profile and downstream feature data remain unchanged.

## Relationship To Adjacent Features

The Dashboard owns the Dashboard surface itself and the Dashboard-side presentation of navigation.

`My Tools` owns the tool-directory experience after the user enters `My Tools`. This Dashboard section defines only the Dashboard's visible entry point into `My Tools`, not the tool-hub behavior that follows.

`My Company`, `My Market`, `My Customers`, `Marketing Coach`, `Campaign Creator`, and other linked destinations own their own workflows and data contracts. The Dashboard may summarize or link to those areas, but it does not define their internal behavior.

For this version, the `My Company`, `My Market`, `My Customers`, and `My Tools` cards should be treated first as polished Dashboard entry points and summary stubs. Their presence here does not imply that the linked destination features are already fully defined elsewhere in the document.

## Behavioral Requirements

The Dashboard must behave as a profile-aware summary surface for the current completed Business Profile.

It must present:

- a welcome banner that greets the owner by the current profile's business name
- a short profile summary drawn from already available business context
- a quick-start checklist for the early product journey
- a momentum / wins summary when enough underlying activity exists to show one
- a daily game plan containing exactly three current-day marketing tasks
- a visible call to action for `Your AI Marketing Coach`
- visible hub cards for `My Company`, `My Market`, `My Customers`, and `My Tools`

The welcome banner must present:

- a small high-emphasis eyebrow label indicating that the AI marketing department is live
- a large welcome headline using the current business name
- one short supporting paragraph sourced from available profile summary data when present, with a product-default fallback when no profile summary exists

The quick-start checklist must:

- show the owner's current progress through the early setup path
- show four steps in this version
- treat the initial brand/profile setup step as already completed once the user is on the Dashboard after onboarding
- provide visible actions for incomplete steps that route into the linked adjacent area
- allow the owner to dismiss the checklist
- render each step as its own full-width row inside one parent card rather than as compact checkboxes

In this version, the visible checklist step labels are:

1. `Your brand profile is ready`
2. `Know your market`
3. `Define your customers`
4. `Create your first campaign`

The momentum / wins summary must appear only when there is meaningful activity to summarize. It may summarize, at minimum:

- active-day streaks
- tasks completed this week
- tools already built for the current profile

When shown, the wins summary should read as a compact horizontal momentum strip rather than as a full card-sized module.

The daily game plan must:

- produce exactly three actionable tasks for the current day
- tailor those tasks to the current Business Profile and any already-available marketing context
- allow each task to be marked complete or incomplete
- allow the owner to refresh the day's plan
- allow the owner to hand a task off to `Marketing Coach` via a visible per-task action
- render those tasks inside one parent card region with a titled header and three visible child task cards in the ready state

The Dashboard may allow the daily game plan region to collapse or expand, so long as the screenshot-backed ready-state composition remains the primary default presentation.

The `Your AI Marketing Coach` region is a promotional CTA block rather than a standard utility card. It must visually read as a full-width high-emphasis entry point distinct from the surrounding informational cards.

The hub-card area must provide visible entry points from the Dashboard into:

- `My Company`
- `My Market`
- `My Customers`
- `My Tools`

The `My Tools` card is the Dashboard-owned entry point into `My Tools`. In this version, the Dashboard defines only the existence, labeling, summary copy, and navigation affordance of that card.

The hub cards are not intended to imply full downstream feature completeness. In this version they serve as high-fidelity entry surfaces and summary stubs that create a stable Dashboard layout while those adjacent features continue to evolve.

## Session Lifecycle

The Dashboard requires a current completed Business Profile.

Before rendering Dashboard content:

1. Resolve the current completed Business Profile.
2. If the Dashboard is still resolving the current profile, render a loading state instead of partial Dashboard content.
3. If no completed Business Profile can be resolved, route the user back into onboarding instead of rendering the Dashboard.
4. If a previously selected active completed profile is available, use it as the current profile.
5. If no previously selected active completed profile is available but one or more completed profiles exist, select one completed profile as the current profile and retain it as the active profile for later Dashboard visits.

The Dashboard header must allow the owner to switch the current profile. When the current profile changes, the Dashboard must refresh its displayed summary content for that profile.

The Dashboard header must also expose a visible `Re-do intake` action. In this version, activating that action routes the user back into onboarding rather than running an inline Dashboard flow.

## Mid-Flow Persistence

In this version, the Dashboard may retain non-authoritative per-profile convenience state across visits.

That retained state must include, at minimum:

- the current active completed profile for Dashboard use
- whether the quick-start checklist is dismissed for a given profile
- the current day's cached game-plan tasks for a given profile
- the current day's completion checkmarks for those tasks

If the owner refreshes the daily game plan, the Dashboard replaces that profile's cached plan for the current day and resets that current-day task-completion state.

Dashboard-owned convenience state must remain separate from the canonical Business Profile record and from downstream feature outputs. Dashboard summaries such as tool counts or profile-based status chips are derived from already-existing underlying data rather than saved as a second source of truth.

## Layout

The Dashboard wraps its content in a persistent Dashboard chrome with the following visible structure:

- **Header:** Zeffer logo, `Re-do intake` action, and current-profile switcher
- **Welcome banner:** prominent brand-colored hero region with the current business name and summary copy
- **Quick-start checklist:** dismissible setup card positioned near the top of the Dashboard
- **Wins / momentum bar:** compact summary strip shown only when there is enough activity to display it
- **Today's Game Plan:** a three-card task region with refresh and per-task actions
- **Marketing Coach CTA:** large promotional card for entering `Marketing Coach`
- **Hub cards:** a grid of visible entry cards for `My Company`, `My Market`, `My Customers`, and `My Tools`

On larger screens, the Dashboard should read as one vertically scrolling page with stacked full-width regions and a two-column hub-card grid near the bottom. On smaller screens, these regions may stack vertically while preserving their visible order.

The Dashboard must define the following visible states:

- **Loading state:** centered loading treatment while the current completed profile is being resolved
- **Checklist dismissed state:** quick-start checklist hidden after dismissal for the current profile
- **Wins hidden state:** wins / momentum bar omitted when there is nothing meaningful to summarize yet
- **Game-plan loading state:** visible loading treatment while the current day's plan is being created
- **Game-plan ready state:** exactly three task cards visible

The ready-state Dashboard composition must preserve these visible layout characteristics:

- the hero banner spans the full content width and uses a saturated blue promotional treatment
- the quick-start checklist appears as a large white card directly under the hero
- the wins strip appears as a compact dark horizontal bar between the checklist and game plan
- the game plan appears as a larger white card with three lighter child cards arranged side by side on desktop
- the Marketing Coach CTA appears as a full-width green promotional banner
- the hub cards appear as four large white cards arranged in a two-by-two grid on desktop

### UI Fidelity Guidance

For this feature, the requirements text is authoritative for behavior, routing, persistence, and actor ownership. The approved `dashboard1.png` and `dashboard2.png` screenshots are authoritative for visible composition, hierarchy, styling intent, and region order. In this set, `dashboard1.png` and `dashboard2.png` are sequential desktop captures of the same ready-state screen rather than alternate designs.

- **Region order:** Header -> welcome banner -> quick-start checklist -> wins / momentum bar -> daily game plan -> Marketing Coach CTA -> hub-card grid
- **Layout structure:** Constrained single-column page shell with full-width stacked sections; hub cards render as a two-column grid on desktop and collapse vertically on smaller screens
- **Typography hierarchy:** Welcome headline and major section headings must be visually dominant; supporting copy must remain secondary and lower contrast
- **Card and container treatment:** The quick-start checklist, daily game plan, Marketing Coach CTA, and hub cards must render as visually distinct containers with rounded corners, light borders or filled backgrounds, and clear separation from the page background; the wins region is intentionally flatter and more strip-like than the other cards
- **Color and emphasis hierarchy:** The welcome banner and Marketing Coach CTA are high-emphasis promotional regions; informational cards and checklist rows should use lighter treatments with accent colors reserved for category cues and actions
- **Icons and visual cues:** The hero eyebrow, checklist step markers, hub-card icons, and Marketing Coach icon block should remain visible because they contribute materially to scanability and hierarchy
- **Navigation affordances:** `Re-do intake`, checklist actions, the Marketing Coach CTA, and hub cards must read as clearly interactive from their visual treatment alone
- **Ready-state density:** The Dashboard should feel intentionally full and useful rather than sparse; avoid collapsing the screenshot-backed modules into overly abstract placeholders
- **Responsive behavior:** Preserve the visible order of regions and collapse dense multi-column areas before shrinking text enough to flatten hierarchy
- **Fidelity priorities:** Match the screenshot-backed hierarchy, section order, and card compositions closely; exact text wrapping, micro-spacing, and icon sizing may vary slightly so long as the visible structure remains intact

## Out of Scope / Deferred

The following are not defined by this Dashboard section:

- the internal behavior of downstream tool destinations beyond the Dashboard-owned entry points into them
- the detailed behavior of `Brand Platform` or any other downstream tool once entered
- server-authoritative persistence for Dashboard convenience state
- cross-surface completion summaries that would require Dashboard to become the source of truth for downstream tool completion
- detailed requirements for `My Company`, `My Market`, `My Customers`, `Marketing Coach`, or `Campaign Creator` beyond the Dashboard's own visible links and summaries

## Intentional Changes from Prototype

None.

---

# 5. My Tools

`My Tools` is the tool-hub surface that users enter from `Dashboard`. It presents Zeffer's visible tool catalog in one place, organized by stage, so the owner can browse available tools and enter a specific tool from a consistent directory rather than from scattered Dashboard cards alone.

In this version, `My Tools` is a high-fidelity navigation-and-organization rough-in. It defines the tool-hub surface, the visible grouping of tools, and the entry points into downstream tools, while intentionally leaving most downstream tool behavior to later sections. Reference designs are provided in `./design` — see `mytools.png` for the intended visual hierarchy and stage grouping.

## Scope In This Version

In this version, `My Tools` is responsible for:

- serving as the tool-directory surface reached from `Dashboard`
- presenting a visible back-navigation path from `My Tools` to `Dashboard`
- presenting the tool catalog grouped into visible stages
- presenting visible tool cards with labels, summaries, and entry affordances
- routing the user from a tool card into the selected downstream tool page
- including `Brand Platform` as one of the visible tool destinations

In this version, `My Tools` is also responsible for preserving the screenshot-backed stage-and-card directory composition even when some downstream tools are still presentational or evolving.

## Data Outcomes

`My Tools` does not create or finalize canonical Business Profile data, tool output data, or downstream feature records of its own.

In this version, `My Tools` is primarily a read-and-navigate surface. It reads the current completed Business Profile context only to determine whether the tool hub may render for the current user session.

Any business data creation, AI generation, extraction, save behavior, or completion behavior after the user enters a tool belongs to the destination tool, not to `My Tools`.

## Relationship To Adjacent Features

`Dashboard` owns the Dashboard-side presentation and navigation into `My Tools`. `My Tools` owns the tool-hub experience after the user enters it.

`Brand Platform`, `Visual Identity`, `Brand Kit`, `Products & Services`, `Brand Assets`, `Campaign Builder`, `Email Sequences`, `Social Posts`, `Video Ad Creator`, `Content Calendar`, `Saved Campaigns`, `Performance Analytics`, `Website Optimization`, and `Local Marketing` each own their own behavior after entry. This section defines only how those tools are presented and entered from the hub.

In particular, `Brand Platform` is a destination listed inside `My Tools`, but the `My Tools` section does not define `Brand Platform` generation behavior, saved outputs, loading behavior beyond the hub-to-tool handoff, or post-completion behavior.

Where a visible tool-card label differs from the canonical feature name elsewhere in this document, this section is authoritative for the visible `My Tools` label and the linked destination is authoritative for the downstream feature behavior. In this version:

- `Products & Services` is the visible `My Tools` label for the `Products` destination
- `Campaign Builder` is the visible `My Tools` label for the `Campaign Creator` destination
- `Performance Analytics` is the visible `My Tools` label for the `Analytics` destination

## Behavioral Requirements

`My Tools` must behave as a stage-organized directory of Zeffer tools for the current completed Business Profile.

It must present:

- a visible page title for `My Tools`
- short summary copy explaining that the tools are organized by stage
- a visible back-navigation control to `Dashboard`
- a visible stage-based grouping of tool cards

In this version, the tool catalog must be organized into exactly three visible stages:

1. `Build Your Brand`
2. `Create Content`
3. `Analyze & Optimize`

Each stage must present:

- a visible stage number
- a visible stage label
- a thin divider or continuation line that visually extends the stage header across the row
- one or more tool cards belonging to that stage

Each tool card must present:

- the tool's visible name
- a short summary description
- a leading icon block with its own accent color treatment
- a visible entry affordance

In this version, the visible tool cards must include at minimum:

- `Build Your Brand`
  - `Brand Platform`
  - `Visual Identity`
  - `Brand Kit`
  - `Products & Services`
  - `Brand Assets`
- `Create Content`
  - `Campaign Builder`
  - `Email Sequences`
  - `Social Posts`
  - `Video Ad Creator`
  - `Content Calendar`
- `Analyze & Optimize`
  - `Saved Campaigns`
  - `Performance Analytics`
  - `Website Optimization`
  - `Local Marketing`

Selecting a tool card must navigate the user from `My Tools` into that tool's destination page.

The visible stage grouping is an organization aid for browsing and discovery in this version. It does not, by itself, define mandatory product prerequisites, completion gating, or locked/unlocked sequencing rules unless a later feature section explicitly does so.

The screenshot-backed ready state uses a dense card directory rather than oversized hero modules. `My Tools` should therefore preserve a compact, browseable tool-grid feel rather than promoting one tool so strongly that the stage organization becomes secondary.

## Session Lifecycle

`My Tools` requires a current completed Business Profile.

Before rendering the tool hub:

1. Resolve the current completed Business Profile for the session.
2. If `My Tools` is still resolving the current profile, render a loading state instead of partial tool-hub content.
3. If no completed Business Profile can be resolved, route the user back into onboarding instead of rendering `My Tools`.
4. If a current completed profile is available, render the tool hub for that profile.

The visible back-navigation control in `My Tools` must return the user to `Dashboard`.

## Layout

`My Tools` wraps its content in a persistent tool-hub chrome with the following visible structure:

- **Header:** back-navigation control to `Dashboard`, Zeffer logo, and a `My Tools` label
- **Page intro:** page title and short summary copy
- **Stage sections:** vertically stacked stage groups with numbered headers
- **Tool grid:** card grid of visible tool destinations within each stage

On larger screens, the tool hub should read as a vertically stacked directory with multi-column tool grids under each stage. On smaller screens, tool cards may stack into fewer columns, but the visible stage order and card readability must remain intact.

`My Tools` must define the following visible states:

- **Loading state:** centered loading treatment while the current completed profile is being resolved
- **Ready state:** all three stage groups and their visible tool cards are rendered

If a listed downstream feature is not yet fully specced elsewhere in this document, its tool card may still appear as a presentational entry point in `My Tools`. In that case, `My Tools` remains responsible only for the tool-hub presentation and navigation affordance, not for defining hidden downstream behavior.

### UI Fidelity Guidance

For this feature, the requirements text is authoritative for behavior, routing, persistence, and actor ownership. The approved `mytools.png` screenshot is authoritative for visible composition, hierarchy, styling intent, and region order for the desktop ready state.

- **Region order:** Header -> page intro -> stage 1 section -> stage 2 section -> stage 3 section
- **Layout structure:** Tool hub renders as vertically stacked stage sections, each with a numbered stage header and a multi-card grid beneath it
- **Typography hierarchy:** The page title must dominate the page intro; stage labels must read as strong subsection headings; tool names must remain more prominent than their descriptions
- **Card and container treatment:** Tool cards must read as lightweight navigational cards with rounded corners, light borders, white backgrounds, and a distinct icon block at the leading edge; the stage sections themselves should feel lighter than the cards they contain
- **Color and emphasis hierarchy:** Each stage and tool family may use accent color cues, but the page should preserve a calm directory feel rather than a promotional CTA-heavy feel
- **Grid density:** Desktop ready state should preserve the screenshot's compact multi-column browsing density, including partial final rows where a stage has fewer cards than the row capacity
- **Navigation affordances:** The back path to `Dashboard` and each tool card's entry affordance must be visually obvious without requiring hover to understand clickability
- **Visible-label fidelity:** The visible card labels and summaries should match the screenshot-backed tool directory closely even when the underlying destination feature uses a different canonical internal name elsewhere in the document
- **Responsive behavior:** Preserve stage order and card readability on smaller screens by reducing grid columns before compressing the cards too tightly
- **Fidelity priorities:** Match the stage grouping, tool-card density, and clean directory structure closely; small differences in icon choice or exact line wrapping are lower priority than preserving the browsing experience

## Out of Scope / Deferred

The following are not defined by this `My Tools` section:

- the internal behavior, data outcomes, or completion rules of `Brand Platform` after entry
- the internal behavior of any other downstream tool after entry
- tool locking, unlock criteria, mandatory sequencing, or completion badges across stages
- post-completion routing from downstream tools back into `My Tools`
- cross-tool summaries, status rollups, or save behavior beyond the hub's own visible organization

## Intentional Changes from Prototype

None.

---

# 6. Brand Platform

`Brand Platform` is a downstream tool entered from `My Tools`. It synthesizes the current Business Profile's onboarding intelligence into a structured brand strategy document, saves that document on the current Business Profile, and renders it as a read-only strategic reference for later use across the product.

In this version, `Brand Platform` is the first fully behavioral downstream tool in the product flow. It owns both the Brand Platform screen and the associated LLM generation behavior that creates the saved Brand Platform document.

Reference designs are provided in `./design` — see the `brandplatform*.png` files for the intended visual hierarchy, section ordering, and document presentation.

## Scope In This Version

In this version, `Brand Platform` is responsible for:

- serving as a destination entered from `My Tools`
- resolving the current completed Business Profile
- automatically generating a Brand Platform document when the current profile does not yet have one
- saving the generated Brand Platform document on the current Business Profile
- rendering the saved Brand Platform document as a structured, read-only artifact
- presenting loading and retry behavior around generation failures

No separate manual Brand Platform intake is required in this version. Business Profile Onboarding already captures enough source material for Brand Platform generation.

## Data Outcomes

`Brand Platform` does not collect new user-authored input through a separate form or chat flow in this version.

Its primary data outcome is a generated structured artifact saved on the current Business Profile as `brand_platform_doc`.

That generated artifact must:

- be derived from the current Business Profile's already-saved onboarding/profile data and onboarding conversation history
- remain associated with the current Business Profile rather than the Organization
- be reusable by later downstream features without requiring Brand Platform to be regenerated on every visit

Generating or regenerating the Brand Platform must not erase or replace the canonical onboarding/profile source fields on the Business Profile. Those source fields remain the upstream business context from which the Brand Platform document is synthesized.

## Relationship To Adjacent Features

`My Tools` owns the tool-hub experience and the entry point into `Brand Platform`.

Business Profile Onboarding owns the upstream business context that Brand Platform relies on. In this version, Brand Platform generation depends on the current Business Profile's saved onboarding/profile fields and the saved onboarding conversation transcript rather than on a separate Brand Platform intake flow.

`Visual Identity`, `Brand Kit`, and other downstream features may later consume the saved Brand Platform document, but those consumer behaviors are not defined by this section.

## Behavioral Requirements

`Brand Platform` must behave as a generated strategic document for the current completed Business Profile.

When the user enters `Brand Platform`:

- if the current completed Business Profile already has a saved usable Brand Platform document, the screen must render that saved document instead of automatically regenerating it
- if the current completed Business Profile does not yet have a saved Brand Platform document, the tool must automatically begin generation without requiring additional manual input

Brand Platform generation must use two classes of source input:

- **Structured profile context** — saved onboarding/profile fields already present on the current Business Profile
- **Full onboarding conversation transcript** — the saved onboarding conversation history for the current Business Profile, excluding completion-token artifacts

In this version, the structured profile context must include at minimum:

- `business_name`
- `business_description`
- `products_services`
- `ideal_customer`
- `differentiator`
- `brand_voice`
- `brand_values`
- `marketing_goals`
- `positioning_statement`
- `brand_summary`

The generation contract must produce a Brand Platform document that is specific to the current business rather than generic filler.

## Session Lifecycle

`Brand Platform` requires a current completed Business Profile.

Before rendering Brand Platform content:

1. Resolve the current completed Business Profile for the session.
2. If Brand Platform is still resolving the current profile, render a loading state instead of partial Brand Platform content.
3. If no completed Business Profile can be resolved, route the user back into onboarding instead of rendering Brand Platform.
4. If a saved usable `brand_platform_doc` already exists on the current Business Profile, render that saved document.
5. If no saved usable `brand_platform_doc` exists, automatically start Brand Platform generation for the current Business Profile.
6. If generation fails, remain on the Brand Platform screen and present a retry path rather than routing the user away.

## Completion Handling

Successful Brand Platform generation must save the generated document on the current Business Profile in `brand_platform_doc`, then render the saved document on the Brand Platform screen.

The Brand Platform document must support the following structured shape:

```
| Field                      | Type             | Notes                                                        |
|---------------------------|------------------|--------------------------------------------------------------|
| category_frame_headline   | string | null    | Short internal label; minimum validity field in current code |
| category_frame_description| string | null    | 2-3 sentence explanation of what the business truly is       |
| category_frame_not        | string[] | null  | 3-4 "Not X: reason" statements                               |
| brand_enemy               | string | null    | Status quo or category enemy                                 |
| brand_enemy_human_truth   | string | null    | Inner-voice customer truth                                   |
| core_insight              | string | null    | Headline insight                                             |
| core_tension              | string | null    | Push-pull tension statement                                  |
| positioning_statement     | string | null    | Full positioning statement                                   |
| brand_promise             | string | null    | Short brand promise; minimum validity field in current code  |
| rallying_cry              | string | null    | Internal rallying cry                                        |
| brand_values              | object[] | null  | 4 objects with `title` and `description`                     |
| brand_personality_traits  | object[] | null  | 5 objects with `trait` and `one_line`                        |
| voice_archetype_name      | string | null    | Voice archetype label                                        |
| voice_archetype_description | string | null  | 3-4 sentence voice-archetype explanation                     |
| voice_principles          | object[] | null  | 4 objects with `title`, `sounds_like`, `doesnt_sound_like`   |
| messaging_pillars         | object[] | null  | 3 objects with pillar, support, and hook fields              |
| audience_insight          | string | null    | 2-3 sentence audience summary                                |
| core_audience_personas    | object[] | null  | 2 persona objects with name/job/triggers/emotional/social needs |
```

In the current implementation, `category_frame_headline`, `positioning_statement`, and `brand_promise` are the minimum required fields for a valid generated document. The broader Brand Platform artifact should still populate the full structure above so the full screen can render as intended.

If generation fails, the screen must present an error state with a visible retry action. Retrying generation must target the same current Business Profile rather than treating the attempt as a new feature flow.

## Layout

The Brand Platform screen wraps its content in persistent Brand Platform chrome with the following visible structure:

- **Header:** visible back-navigation control, Zeffer logo, Brand Platform label, and current business name
- **Cover section:** product label, large business name, and introductory strategic summary copy
- **Section index:** visible six-part overview of the document sections
- **Document body:** a read-only strategic document presented in the following visible order:
  - `Category Frame`
  - `Core Audience`
  - `Core Insight + Tension`
  - `Brand Foundations`
  - `Brand Voice`
  - `Messaging Framework`
- **Footer:** Zeffer branding and generated-document attribution

The Brand Platform screen must define the following visible states:

- **Loading state:** centered loading treatment while the current completed profile is being resolved
- **Generating state:** full-page generation treatment while the Brand Platform document is being created
- **Error state:** visible error message with retry action if generation fails
- **Ready state:** the saved Brand Platform document rendered as the full read-only strategic artifact

### UI Fidelity Guidance

For this feature, the requirements text is authoritative for behavior, routing, persistence, actor ownership, and generation rules. The approved `brandplatform*.png` screenshots are authoritative for visible composition, hierarchy, styling intent, and region order. In this set, the screenshots represent sequential scroll captures of the same desktop ready-state document rather than alternate layouts.

- **Region order:** Header -> cover section -> section index -> Category Frame -> Core Audience -> Core Insight + Tension -> Brand Foundations -> Brand Voice -> Messaging Framework -> footer
- **Layout structure:** The ready state must read as a long-form document page with one primary content column and section-based vertical progression rather than a dashboard or card grid
- **Typography hierarchy:** The business name and primary section titles must be visually dominant; small uppercase labels should act as lightweight section markers rather than competing headings
- **Card and container treatment:** Within the document body, strategic content may use visually distinct blocks, quote panels, split layouts, and framed sections, but the screen should still read as one cohesive editorial document
- **Color and emphasis hierarchy:** Use strong emphasis selectively for key strategic callouts such as the category label, core tension block, or rallying-cry treatment; most of the document should preserve a restrained editorial visual system
- **State presentation:** The generating state should replace the document body with a focused full-page progress treatment; the ready state should not appear partially rendered
- **Scroll behavior:** The page should behave like a vertically scrolling document with the sticky top header remaining visually stable while the body scrolls beneath it
- **Responsive behavior:** Preserve the editorial reading hierarchy on smaller screens by stacking split layouts vertically and maintaining readable text measure rather than forcing dense side-by-side compositions
- **Fidelity priorities:** Match the document-like reading experience, section ordering, and visual hierarchy closely; small implementation differences in paragraph wrapping or decorative spacing are lower priority than preserving the strategic-document feel

## Out of Scope / Deferred

The following are not defined by this Brand Platform section:

- a separate manual Brand Platform intake flow
- editing the generated Brand Platform document in place
- downstream consumer behavior in `Visual Identity`, `Brand Kit`, or other tools that later use Brand Platform outputs
- richer post-completion routing rules beyond remaining on or returning to the Brand Platform screen after successful generation
- normative use of every available onboarding/profile field in the structured prompt context beyond the minimum current structured-input set listed above

## Intentional Changes from Prototype

None.

---

# Appendix: Validation

Two distinct review processes are defined here. Run them independently — they have different inputs, different outputs, and different purposes.

## Applying Validation Findings

Edits made in response to validation findings must be **holistic**. Do not fix a finding with an isolated sentence change if that change leaves duplicated references, schemas, counts, prompts, or scoring rules out of sync elsewhere in the document.

When resolving a finding from either the **Document Self-Consistency Review** or the **Requirements Fidelity Review**:

1. Identify the authoritative section for the behavior, entity, count, schema, or rule being changed.
2. Update all dependent references in the same edit set. This includes repeated counts, subsection summaries, extraction schemas, system-prompt summaries, appendices, Intentional Changes entries, and Todo notes when they depend on the changed rule.
3. Resolve the target issue without introducing a new inconsistency in adjacent sections.
4. Do not improve a fidelity score by silently weakening or deleting a product requirement unless that reduction in scope is an intentional product decision. If scope is intentionally reduced, make that reduction explicit and update all dependent sections so the lowered scope is internally consistent.
5. If a decision is being deferred rather than specified now, move it to a clearly non-normative location such as `Appendix: Todos`, and remove any current normative requirement that still depends on that deferred behavior.

Before considering the edit complete, perform a local consistency pass on the affected area:

- counts and enumerated sets still line up
- duplicated descriptions still match or explicitly defer to one source
- schemas and topic lists still describe the same scope
- references still point to sections or features that actually exist
- the edit does not create a new fidelity-scoring ambiguity
- the edit does not lower the intended product bar accidentally

Validation edits should reduce ambiguity, preserve the intended product standard, and leave the document more internally coherent than it was before the finding was addressed.

## Document Self-Consistency Review

Run the reusable review defined in `docs/validation/document-self-consistency-check.md` against the target requirements document.

For a target document at `path/to/file.md`:

- **Input document** — `path/to/file.md`
- **Review report path** — `path/to/file-dc-report.md`

Store review results in the same directory as the target document, using the target filename with `-dc-report.md` appended before the extension.

## Requirements Fidelity Review

Run the reusable review defined in `docs/validation/scoring-check.md` against the target requirements document and the relevant source files for the feature being scored.

For a target document at `path/to/file.md`:

- **Input document** — `path/to/file.md`
- **Scoring guide** — `docs/validation/scoring-check.md`

Use the target document's own feature sections and `Appendix: Overall Intentional Changes from Prototype` together with the reusable scoring guide.

# Appendix: Overall Intentional Changes from Prototype

The following intentional changes apply across the full requirements document and are not specific to any single feature. This appendix supplements feature-level "Intentional Changes from Prototype" sections; it does not replace them for feature scoring.

- **Net-new concepts:** Account and Organization entities do not exist anywhere in the prototype. All requirements behavior that depends on them is additive — there is nothing in the prototype to score against for these concepts.
- **Architectural upgrade:** The prototype uses `localStorage` as the primary mechanism for routing and session state decisions. The requirements specify server-authoritative state (database-backed flags and records) for all critical routing decisions. This upgrade applies wherever the requirements describe server-driven state. Only the mechanism is covered — the routing logic and behavior built on top of it must still match the spec and are subject to scoring.

---

# Appendix: Todos

- Revisit whether Brand Assets should become part of Business Profile Onboarding in a future revision of this requirements document.
