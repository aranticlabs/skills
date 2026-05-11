---
name: write-prd
description: Creates or updates Product Requirements Documents (PRDs) using the Arantic template system. The 7-document set is the baseline — complex concepts may use more documents, and existing PRDs can be extended with new files when they grow beyond their original scope. Invoke this skill only when the user explicitly asks to create, write, extend, or update PRDs (e.g., "create a PRD", "write PRD documents", "I want PRDs for this feature", "add a document to the existing PRD", "split this PRD file"). Do not invoke for general product ideas, brainstorming, informal planning, or requests to spec a feature unless the user clearly asks for PRD documentation.
model: opus
---

# Write PRD

You are helping the user create or update Product Requirements Documents for a feature or product. The Arantic template starts from a 7-document baseline, but the document count is **not fixed** — complex concepts may need additional documents, and an existing PRD set may need new documents added when it grows. These documents give AI coding assistants complete context upfront, eliminating the cycle of vague prompts and back-and-forth corrections.

## Step 0: Determine the mode

Before anything else, decide which mode you're in:

- **Create mode** — no PRD exists yet for this feature. Proceed from Step 1.
- **Update mode** — a PRD set already exists (the user references an existing `docs/PRDs/<feature>/` directory, says "extend", "add to", "split", "update", or you can see the files). **Jump to the Update Mode section below before touching any file.**

If unclear, ask the user:

> "Is this a new PRD I should create from scratch, or are you asking me to extend/update an existing PRD set? If existing, please point me to the directory."

## Step 1: Understand what they're building

Ask the user to describe what they want to build in plain language. You don't need a perfect picture — you just need enough to assess scope. One or two paragraphs is fine.

If they've already described it in the conversation, use that and confirm rather than asking again.

## Step 2: Determine feature size and document count

Based on their description, classify the feature and propose a document set. **The 7-document baseline is a starting point, not a ceiling.** For complex concepts, add domain-specific documents. For very large topics, split a single document into multiple focused files (e.g., `PRD-04a-API-ENDPOINTS-PUBLIC.md` + `PRD-04b-API-ENDPOINTS-ADMIN.md`).

| Size        | Signals                                                               | Documents to create                                                                                                 |
| ----------- | --------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| **Small**   | Bug fix, minor UI tweak, copy change                                  | No PRD needed — suggest skipping                                                                                    |
| **Medium**  | New CRUD feature, form, simple API                                    | INDEX + OVERVIEW + DATABASE-SCHEMA + IMPLEMENTATION-STEPS                                                           |
| **Large**   | New module, significant user-facing workflow                          | All 7 baseline documents                                                                                            |
| **Complex** | Multi-app, financial calculations, integrations, complex domain logic | All 7 baseline + additional domain-specific docs (e.g., integration spec, state machine, security model, glossary)  |

### When to add or split documents

Add a new document (or split an existing one) when **any** of the following applies:

- A single planned document would exceed ~800 lines or ~25KB. Long files are harder for both humans and AI to navigate.
- A topic is conceptually distinct from its parent (e.g., a third-party integration deserves its own `PRD-0X-INTEGRATION-<name>.md` rather than a buried section).
- Multiple audiences need different slices (e.g., public API vs. internal admin API).
- The feature spans multiple sub-systems that each need their own schema, endpoints, or state diagrams.

Examples of valid extra documents:

- `PRD-07-INTEGRATION-<system>.md` — third-party integration contracts
- `PRD-08-STATE-MACHINE.md` — complex state transitions / workflow rules
- `PRD-09-SECURITY-MODEL.md` — auth, authorization matrix, threat model
- `PRD-10-MIGRATION-PLAN.md` — data migration from a legacy system
- `PRD-11-FRONTEND.md` — wireframes, UX/UI instructions, component inventory, interaction patterns, design tokens (pull out of PRD-01 when the UI is non-trivial or has many screens)
- `PRD-0Xa.md` / `PRD-0Xb.md` — splits of a baseline document that grew too large

### Confirm before generating

Show your reasoning — list the documents you plan to create, why you picked that set, and any extras beyond the baseline. Then **ask the user to confirm** before generating. If they want more, fewer, or differently named documents, follow their lead.

## Step 2.5: Assess feature aspects

A PRD should only describe what's actually in the feature. Before creating documents, figure out which aspects apply. Read the user's description, any existing product vision document, or concept notes and answer each question — then skip sections that don't apply.

| Aspect           | Question                                                                        | If **No**, skip                                            |
| ---------------- | ------------------------------------------------------------------------------- | ---------------------------------------------------------- |
| **Frontend**     | Does a human interact with a UI? (web page, mobile screen, admin panel, CLI UI) | Wireframes / UI flow (PRD-01), Phase 5: UI (PRD-06)        |
| **Persistence**  | Does this feature read/write its own data?                                      | PRD-03 entirely, Phase 1 DB steps (PRD-06)                 |
| **HTTP API**     | Does this expose endpoints callers consume?                                     | PRD-04 entirely, Phase 3 API steps (PRD-06)                |
| **Auth/roles**   | Do different users see or do different things?                                  | User roles table (PRD-01), Authentication section (PRD-04) |
| **Integration**  | Does this talk to an external system (3rd-party API, queue, webhook)?           | Integration points (PRD-05)                                |
| **Domain terms** | Are there non-obvious domain words readers won't know?                          | Domain glossary (PRD-02)                                   |
| **New deps**     | Does this need new packages or services?                                        | Dependencies (PRD-05)                                      |

**Examples:**

- _Pure backend scheduler job (no UI, internal only)_ → skip Frontend + HTTP API + Auth/roles sections. Keep business rules, persistence, integration.
- _Pure library utility (stateless, no DB, no API)_ → skip Persistence + HTTP API + Frontend. Keep business logic and implementation architecture.
- _Simple admin form backed by CRUD_ → keep everything.
- _Ingestion pipeline for a new connector_ → skip Frontend. Emphasize Integration and Persistence.

Before proceeding, list the aspects you've identified and the sections you plan to skip. Confirm with the user. When you later generate each document, omit skipped sections entirely rather than leaving empty placeholders — an empty section is worse than no section.

## Step 3: Establish the directory

Ask where they want the PRD to live. The convention is:

```
docs/PRDs/<feature-name>/
```

Use the feature name as a short kebab-case slug (e.g., `task-management`, `user-onboarding`, `billing-module`). Create the directory:

```bash
mkdir -p docs/PRDs/<feature-name>
```

## Step 4: Create the documents

Create each document as a Markdown file using the templates below. Fill in what you already know from the conversation. Use `[placeholder]` brackets for anything the user still needs to fill in. Add HTML comments (`<!-- ... -->`) where guidance would help them fill in a section later.

**Apply the aspect assessment from Step 2.5** — omit sections that don't apply to this feature. Do not leave empty or placeholder-only sections just because they're in the template. The template is a menu, not a checklist.

Work through documents in order. After creating each one, briefly tell the user what you filled in, what was skipped (and why), and what still needs their input before moving to the next.

---

### PRD-00-INDEX.md

The navigation hub. Every other document links from here.

```markdown
# [Feature Name] — PRD Index

## Feature summary

[2-3 sentence plain-language description of what this feature does and why it exists.]

## Capabilities

- [Capability 1]
- [Capability 2]
- [Capability 3]

## Key decisions

<!-- Record significant architectural or product decisions here as they are made. -->

| Decision   | Choice   | Rationale |
| ---------- | -------- | --------- |
| [Decision] | [Choice] | [Why]     |

## Documents

| Document                                                         | Purpose                           | Status |
| ---------------------------------------------------------------- | --------------------------------- | ------ |
| [PRD-01-OVERVIEW.md](PRD-01-OVERVIEW.md)                         | Vision, scope, wireframes         | Draft  |
| [PRD-02-BUSINESS-LOGIC.md](PRD-02-BUSINESS-LOGIC.md)             | User stories, rules, test cases   | Draft  |
| [PRD-03-DATABASE-SCHEMA.md](PRD-03-DATABASE-SCHEMA.md)           | Tables, indexes, TypeScript types | Draft  |
| [PRD-04-API-ENDPOINTS.md](PRD-04-API-ENDPOINTS.md)               | REST API specification            | Draft  |
| [PRD-05-IMPLEMENTATION.md](PRD-05-IMPLEMENTATION.md)             | Architecture, directory structure | Draft  |
| [PRD-06-IMPLEMENTATION-STEPS.md](PRD-06-IMPLEMENTATION-STEPS.md) | Phased implementation plan        | Draft  |
```

---

### PRD-01-OVERVIEW.md

Vision and scope. Written for both product owners and developers.

````markdown
# [Feature Name] — Overview

## Vision

[One sentence: what this feature achieves for the user.]

## Problem statement

[What problem does this solve? Who has this problem? What happens today without this feature?]

## Scope

### In scope

- [What this feature includes]

### Out of scope

- [What this feature explicitly does not include — be specific]

## Success criteria

<!-- How will you know this feature is working? These become acceptance tests. -->

- [Measurable outcome 1]
- [Measurable outcome 2]

## User roles

<!-- CONDITIONAL: include only if the feature has multiple distinct user roles or permission levels. Skip for single-role or internal-only features. -->

| Role   | Description    | Permissions        |
| ------ | -------------- | ------------------ |
| [Role] | [Who they are] | [What they can do] |

## Wireframes / UI flow

<!-- CONDITIONAL: include only if a human interacts with a UI. Skip entirely for backend-only, scheduled jobs, libraries, or pipeline features. ASCII wireframes are preferred — they give AI assistants a visual spec to implement against. -->

```
[ASCII wireframe here]
```

## Technical notes

<!-- Existing patterns, libraries, or constraints the implementation must follow. -->

- [Note 1]
````

---

### PRD-02-BUSINESS-LOGIC.md

The "what" in detail. Product owners and developers both read this.

```markdown
# [Feature Name] — Business Logic

## Domain glossary

<!-- Define terms specific to this feature's domain. AI assistants will use these definitions. -->

| Term   | Definition   |
| ------ | ------------ |
| [Term] | [Definition] |

## User stories

<!-- Format: As a [role], I want to [action] so that [benefit]. -->

### [Story group name]

**US-01:** As a [role], I want to [action] so that [benefit].

**Acceptance criteria:**

- [Criterion 1]
- [Criterion 2]

## Business rules

<!-- Rules that must always be true. Number them for reference in test cases. -->

**BR-01:** [Rule]
**BR-02:** [Rule]

## Edge cases

- [Edge case and how it should behave]

## Verification test cases

<!-- These map directly to automated tests. Be specific about inputs and expected outputs. -->

| ID    | Scenario   | Input   | Expected result |
| ----- | ---------- | ------- | --------------- |
| TC-01 | [Scenario] | [Input] | [Expected]      |
```

---

### PRD-03-DATABASE-SCHEMA.md

SQL definitions and TypeScript interfaces. Written for developers.

**CONDITIONAL:** skip this entire document if the feature has no persistence (stateless utility, pure business logic, pass-through proxy).

````markdown
# [Feature Name] — Database Schema

## Tables

### [table_name]

```sql
CREATE TABLE [table_name] (
  id          UUID        PRIMARY KEY DEFAULT gen_random_uuid(),
  -- [column]  [type]      [constraints] -- [description]
  created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

## Indexes

```sql
-- [Reason for this index]
CREATE INDEX idx_[table]_[column] ON [table_name]([column]);
```

## TypeScript interfaces

```typescript
interface [EntityName] {
  id: string;
  // [field]: [type]; // [description]
  createdAt: Date;
  updatedAt: Date;
}
```

## Relationships

<!-- Describe foreign keys and cardinality. -->

- `[table_a].id` → `[table_b].[table_a_id]` (one-to-many)
````

---

### PRD-04-API-ENDPOINTS.md

REST API specification. Written for developers.

**CONDITIONAL:** skip this entire document if the feature exposes no HTTP API (internal job, library, CLI tool, background worker).

````markdown
# [Feature Name] — API Endpoints

## Base path

`/api/v1/[resource]`

## Authentication

[Describe auth requirements — JWT, session, API key, etc.]

---

## [Resource] endpoints

### GET /[resource]

**Description:** [What this returns]

**Query parameters:**

| Parameter | Type   | Required | Description   |
| --------- | ------ | -------- | ------------- |
| [param]   | string | No       | [Description] |

**Response 200:**

```json
{
  "[field]": "[value]"
}
```

**Response 404:**

```json
{ "error": "Not found" }
```

---

### POST /[resource]

**Description:** [What this creates]

**Request body:**

```json
{
  "[field]": "[value]"
}
```

**Response 201:**

```json
{
  "id": "[uuid]"
}
```
````

---

### PRD-05-IMPLEMENTATION.md

Code architecture and integration points. Written for developers.

````markdown
# [Feature Name] — Implementation

## Directory structure

```
src/
  [feature]/
    [feature].controller.ts  # Route handlers
    [feature].service.ts     # Business logic
    [feature].schema.ts      # Zod validation schemas
    [feature].types.ts       # TypeScript interfaces
    __tests__/
      [feature].service.test.ts
```

## Architecture decisions

| Decision | Choice   | Rationale |
| -------- | -------- | --------- |
| [Area]   | [Choice] | [Why]     |

## Integration points

<!-- Other parts of the system this feature touches. -->

- **[System/module]:** [How this feature integrates]

## Dependencies

<!-- New packages or services required. -->

- `[package]` — [why needed]
````

---

### PRD-06-IMPLEMENTATION-STEPS.md

Phased task list. AI assistants follow this sequentially.

```markdown
# [Feature Name] — Implementation Steps

<!-- Feed one phase at a time to your AI assistant using /plan mode. -->

## Phase 1: Database & scaffold

- Create migration for [table_name]
- Add TypeScript interfaces to `[path]`
- Scaffold service file at `[path]`
- Scaffold controller file at `[path]`

## Phase 2: Core logic

- Implement [core operation 1]
- Implement [core operation 2]
- Add input validation with Zod schemas

## Phase 3: API endpoints

- Implement GET /[resource]
- Implement POST /[resource]
- Implement PUT /[resource]/:id
- Implement DELETE /[resource]/:id
- Wire routes to [router file]

## Phase 4: Tests

- Unit tests for service layer covering TC-01 through TC-[N]
- Integration tests for API endpoints
- Edge case tests for BR-01 through BR-[N]

<!-- CONDITIONAL: include Phase 5 only if the feature has a UI. Skip entirely for backend-only features. -->

## Phase 5: UI

- [UI task 1]
- [UI task 2]

## Dependencies between phases

Phase 1 must complete before Phase 2. Phases 3 and 4 can run in parallel after Phase 2.
```

---

## Step 5: Offer to generate technical docs from business docs

After creating the INDEX, OVERVIEW, and BUSINESS-LOGIC documents, offer to draft the technical documents (DATABASE-SCHEMA, API-ENDPOINTS, IMPLEMENTATION) automatically:

> "I can draft the database schema, API endpoints, and implementation architecture from the overview and business logic you've filled in. Want me to generate those now, or would you prefer to fill them in yourself?"

If yes, read PRD-01 and PRD-02 and generate the technical documents based on the requirements. Tell the user what assumptions you made and flag anything that needs their input.

## Step 6: Finish

Once all planned documents are created:

1. Tell the user which files were created and their paths
2. Tell them what still needs their input (sections with `[placeholders]`)
3. Show them how to start implementation:

```
/plan Read PRD-00-INDEX.md to understand the feature, then read
PRD-06-IMPLEMENTATION-STEPS.md to see the plan. Research the codebase
and plan Phase 1.
```

## Tips for filling in the documents

- Work through documents in order (INDEX → OVERVIEW → BUSINESS-LOGIC → technical docs)
- Don't aim for perfection on the first pass — fill what you know, leave placeholders for the rest
- For PRD-02 acceptance criteria, write them as if they'll become automated test assertions
- For PRD-06, keep phases small enough that each can be implemented and reviewed independently
- The more specific the business logic doc, the less the AI will have to guess during implementation

---

## Update Mode: extending or modifying an existing PRD

Use this mode when the PRD already exists and the user wants to add documents, split a grown file, or revise content. **The rule: never silently rewrite an existing PRD. Ask first, then act.**

### Step U1: Read the existing PRD set

Before asking anything, read the existing files so your questions are informed, not generic:

1. List the directory (`ls docs/PRDs/<feature-name>/`).
2. Read `PRD-00-INDEX.md` to understand the document set and stated capabilities.
3. Read any document the user mentioned by name.
4. Note file sizes — flag any file over ~800 lines as a split candidate.

### Step U2: Ask the user the necessary questions

**Always run a Q&A round before writing.** The questions below are the minimum; add more if the request is ambiguous. Skip a question only if the user already answered it unprompted.

**Required questions (ask all that apply):**

1. **Intent** — "Do you want to (a) add a brand-new document to the set, (b) split an existing document that has grown too large, (c) revise/expand content inside an existing document, or (d) a combination?"
2. **Scope** — "Which document(s) are in scope? Should I touch anything else, or stay strictly inside the files you named?"
3. **Naming** — "For new documents, what slug/title do you want? (default: `PRD-0X-<TOPIC>.md` where X is the next free number)"
4. **Splits** — "If splitting, how do you want the content divided? By audience (public vs admin), by sub-system, by phase, or another axis?"
5. **Preservation** — "Should I preserve the existing wording verbatim where possible, or am I free to rewrite for clarity?"
6. **Decisions log** — "Should I record this change in the `Key decisions` table in PRD-00-INDEX.md?"
7. **Open questions** — anything in the user's request that you cannot answer from the existing files. Ask explicitly rather than guessing.

Present the questions as a short numbered list. Wait for answers. Do **not** start editing files until the user has responded.

### Step U3: Propose the plan, then confirm

After Q&A, write back a concrete plan in 5–10 lines:

- Files to create (with full paths)
- Files to modify (with a one-line summary of the change per file)
- Files to leave untouched
- Updates to `PRD-00-INDEX.md` (new rows in the Documents table, new entry in Key decisions if requested)

End with: **"Confirm and I'll proceed, or tell me what to adjust."** Wait for explicit confirmation before any write.

### Step U4: Execute the plan

Once confirmed:

1. Create or modify files as agreed. Match the existing PRD's tone, heading style, and section ordering — do not reformat the rest of a file just because you touched part of it.
2. **Always update `PRD-00-INDEX.md`** when adding or splitting documents — add the new row(s) to the Documents table and, if the user requested it, add an entry to the Key decisions table.
3. When splitting a file: move content cleanly, leave a one-line pointer in the original file (e.g., `> Public endpoints moved to [PRD-04a-API-ENDPOINTS-PUBLIC.md](PRD-04a-API-ENDPOINTS-PUBLIC.md).`) if the original is still referenced elsewhere; otherwise delete the original and update all inbound links.
4. After writing, re-read the INDEX and any modified document to verify internal consistency (no broken links, no contradictions, document numbering still makes sense).

### Step U5: Report

Tell the user:

- Which files were created, modified, or deleted (with paths)
- Any sections still containing `[placeholders]` that need their input
- Any inconsistencies you noticed but did not fix (and why)
