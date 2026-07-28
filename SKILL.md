---
name: codebase-documentation
description: Produces detailed, evidence-backed documentation for an existing software project — user guide, system architecture, workflows, data model, security controls, and operations. Use when a user says "document this project", "write system docs", "create technical documentation", "document this codebase", "produce a user guide", "write developer docs", "audit this system", or "create a system document". Supports English and Chinese output. Optionally integrates Codegraph (MCP symbol-graph) for large repos and op7418/Humanizer-zh for Chinese post-processing.
license: MIT
compatibility: Claude Code and Claude.ai. Codegraph integration requires Node 22 or 24 and `npm install -g @colbymchenry/codegraph` (optional; only beneficial for repos over ~150 files). Humanizer-zh integration requires Python 3 and `op7418/Humanizer-zh` (optional; only needed for Chinese-language output).
metadata:
  author: Liang Guo
  version: 2.1.0
  category: documentation
---

# Codebase Documentation

Produce trustworthy documentation grounded in the actual code, tests, and configuration of an existing project. The output serves three audiences simultaneously: users who need to operate the system, auditors who need to trace behavior to evidence, and maintainers who need to understand architecture, invariants, and operations.

## First Principles

1. **Code is the behavioral authority.** Treat PRDs, READMEs, and comments as declared intent. When they conflict with executable behavior, document the code's actual behavior and name the discrepancy.
2. **Discovery is the most expensive part of documentation.** Use Codegraph's pre-indexed symbol graph when available to answer "what calls what" in two MCP calls instead of forty grep operations. Fall back to grep/glob/Read only when Codegraph is absent or the repo is small.
3. **Audience language is a first-class decision.** Default to English. If the product is Chinese-first — Chinese UI, Chinese users, Chinese operations team — produce Chinese documentation and apply a humanize pass to eliminate AI-writing artifacts.
4. **Humanization removes AI tells without sacrificing precision.** For Chinese output, a second pass with op7418/Humanizer-zh or the Claude-native ZH checklist eliminates translationese. Never soften claim status, weaken hedges, or remove necessary caveats during humanization.
5. **Session continuity requires harness files.** Documentation of any serious codebase crosses context boundaries. The harness files — not conversation memory — are the authoritative state.

## Detect Phase and Resume

At the start of every invocation, check for `DOCUMENTATION_PROGRESS.md` in the project's docs directory. If present, read it first and continue only from its recorded phase and next unit. If absent, create the harness files (see [harness-files.md](references/harness-files.md)) and enter `SETUP`.

Never redo completed units because the conversation was compacted.

## Harness Files

Create and maintain these files under the project's `docs/` directory:

- `DOCUMENTATION_PLAN.md` — audience, scope, output language, evidence standard, terminology decisions, and acceptance criteria.
- `DOCUMENTATION_PROGRESS.md` — current phase, next unit, source revision, capabilities complete, blockers.
- `GLOSSARY.md` — domain terms, UI labels, state names, code identifiers, and locked synonyms.
- `DOCUMENTATION_LOG.md` — append-only record of every action, command, result, and open gap.

Read [harness-files.md](references/harness-files.md) for templates. Do not pre-populate empty logs.

## Tool Availability Check

At the start of a new documentation effort, run these checks once and record results in `DOCUMENTATION_LOG.md`:

```bash
# Codegraph — optional, recommended for repos over ~150 files
codegraph status 2>/dev/null || echo "codegraph: absent"

# Humanizer-zh — optional, for Chinese documentation output
python3 -c "import sys; sys.path.insert(0, 'Humanizer-zh'); import humanizer" 2>/dev/null \
  && echo "humanizer-zh: available" || echo "humanizer-zh: absent"
```

If Codegraph is **available** and the repo has more than ~150 files: use it in Phases 1–4 as described in [codegraph-guide.md](references/codegraph-guide.md). If `codegraph status` shows zero symbols, run `codegraph init` first.

If Humanizer-zh is **available** and the output language is Chinese: invoke it in Phase 7.5. If **absent**, apply Claude-native ZH humanization using [humanize-zh-pass.md](references/humanize-zh-pass.md).

## Governing Principles

The document is a claim-to-evidence map, not a prose rewrite of the repository.

1. **Separate audiences without duplicating facts.** Explain the same capability first as a user outcome, then as an auditable implementation contract.
2. **Make scope explicit.** Distinguish implemented, partially implemented, configured-but-unverified, deprecated, and planned behavior.
3. **Never promote inference to fact.** Mark assumptions, unknowns, environment-dependent behavior, and evidence gaps.
4. **Reproducibility is part of documentation.** Every material claim needs a source path, symbol/route/config key, test, command, or observed runtime result.

Read [evidence-and-analysis.md](references/evidence-and-analysis.md) when deciding what counts as evidence, how to analyze risk, or how to handle gaps.

## Authority Order

Resolve conflicts in this order:

1. current user instructions and explicitly approved scope;
2. the requested documentation plan and locked terminology decisions;
3. observed runtime behavior and reproducible verification results;
4. executable code, configuration, schemas, and migrations;
5. tests and CI definitions, interpreted according to what they actually exercise;
6. product requirements, architecture documents, comments, and README files;
7. assumptions and inference.

When a lower-ranked source conflicts with a higher-ranked source, document the conflict and current behavior; do not silently rewrite the code's behavior to match an aspirational document.

## Workflow

### Phase 1: Establish the Evidence Boundary

Inspect the repository before writing. Identify:

- product requirements, ADRs, README files, release notes, and existing documentation;
- application entry points, routes/controllers, services, models, migrations, jobs, CLI commands, templates, assets, and integrations;
- dependency manifests, environment examples, deployment files, permissions, secrets handling, and data stores;
- unit, integration, end-to-end, security, migration, and operational tests;
- CI scripts and available local commands.

**With Codegraph:** Run `codegraph_context` with a broad query such as "entry points routes services models" to obtain the primary symbol graph. Use `codegraph explore <module>` for each major subsystem. This replaces most grep/glob/find calls. See [codegraph-guide.md](references/codegraph-guide.md) for the exact call sequence.

**Without Codegraph:** Use grep, glob, and Read to build the same inventory manually.

Record the commit/version, date, inspected paths, commands run, and environment limitations in `DOCUMENTATION_LOG.md`. Do not hide untracked or generated files that materially affect behavior.

### Phase 2: Build a MECE Capability Inventory

Classify every user-visible or operational capability exactly once in a primary area. Use this partition unless the product clearly requires a better one:

1. identity, authentication, authorization, and account lifecycle;
2. primary domain workflows and state transitions;
3. content/data creation, import, export, and storage;
4. collaboration, communication, and notifications;
5. search, navigation, reporting, and analytics;
6. administration, configuration, and tenancy;
7. integrations, scheduled work, and external effects;
8. reliability, security, privacy, accessibility, localization, and observability;
9. deployment, operations, recovery, and maintenance.

For each capability, capture actors, preconditions, happy path, alternate paths, validation, failure behavior, authorization boundary, persistence effects, external effects, and evidence. Check that no feature is missing, duplicated across areas, or left as an unexplained orphan.

**With Codegraph:** Use `codegraph query <term>` to locate every symbol associated with each capability area. Cross-check routes, models, and jobs against the symbol graph to find orphaned code with no documented capability.

### Phase 3: Trace the System from Interface to Effect (Code View)

For each important workflow, trace the implementation path end to end:

`route or entry point → validation/auth → domain/service logic → persistence → side effect → response/UI`

Name relevant files, symbols, and the exact line range for each hop. For each hop, capture:

- **Auth/validation gate**: which decorator or check runs, what happens when it fails (404/403/redirect/message)
- **Domain logic**: state transitions, invariants, idempotency, transaction boundaries
- **Persistence**: what is written, when `commit()` is called, whether `rollback()` exists on failure
- **Side effects**: emails, audit logs, scheduled tasks, external calls
- **Response**: redirect, template render, flash message, HTTP status code

Include short, targeted code excerpts that show the security boundary, business rule, data transformation, or integration contract. Never dump whole files or secrets. The output of this phase is a **code-path inventory** — a structured record of exactly what happens when each route is invoked. This inventory feeds Phase 3.5.

**With Codegraph:** Use `codegraph_explore <route-or-handler>` to receive the call path across files in a single MCP call. Use `codegraph callers <symbol>` to verify all callers of a shared service. Use `codegraph node <symbol>` for callees at each hop.

### Phase 3.5: Reconstruct User-Facing Procedures

The code-path inventory from Phase 3 is implementation-facing. This phase reverses the lens and produces user-facing procedures — what a person does, sees, and recovers from. **Do not skip any route or operation discovered in Phase 2.** For every route that serves a user action, produce:

#### For each operation, four mandatory elements

**1. Preconditions** — what the user must have done or satisfied before starting:
- Login state (which role, must be `active`?)
- Data prerequisites (must be enrolled in the course? must be the chapter owner?)
- System state prerequisites (course must be `published`? chapter must not be `superseded`?)
- Derive these from the auth gate, validation checks, and early `abort()` / `redirect()` calls in the code path

**2. Step-by-step operations** — numbered actions the user performs:
- Use concrete UI labels in the product's language (e.g., "点击蓝色的'提交'按钮" not "click submit")
- If the code reads from `request.form` or `request.files`, each field is a separate step
- If the code has conditionals that change the user experience (e.g., `mode == "self-study"` vs `"assessment"`), branch the steps
- Derive these from route methods (GET vs POST), form fields, and template names

**3. Expected visible result** — what the user sees on the happy path:
- Which page/template renders next (`redirect(url_for(...))` or `render_template(...)`)
- What `flash("...", "success")` message appears
- What data is visible (scores, answers, status changes)
- Derive these from the success-branch `return` statements and `flash()` calls

**4. At least one alternate or error path** — what happens when something goes wrong:
- Use the **exact error string** from the code (e.g., `"Invalid credentials or inactive account."`, not "login fails")
- State the HTTP status code if the route returns one
- If the route has multiple failure branches (400, 403, 404, 409), cover each distinct error the user can encounter
- Derive these from every `flash(..., "error")`, `abort()`, and non-2xx `return` in the code path

#### Procedure generation rule

Procedures must be grounded in what the user sees and does — not in what the code calls. Use code excerpts only to verify auth checks, data mutations, and the exact error strings. The procedure itself should be understandable by someone who has never read the source code.

#### Cross-check

After generating procedures for all routes, reconcile against the Phase 2 MECE inventory. Every user-visible capability must have a procedure. Log any capability that lacks a procedure in `DOCUMENTATION_LOG.md` with the reason.

### Phase 4: Analyze Data, Trust, and Risk

Document the domain model and lifecycle of important records: fields, relationships, constraints, ownership, retention/deletion behavior, and sensitive data. For every trust boundary, explain input validation, authorization, output encoding, secret handling, file handling, rate limiting, auditability, and known residual risk. Identify high-risk logic and point to tests that exercise it.

**With Codegraph:** Use `codegraph impact <symbol>` to map the blast radius of every high-risk data model, auth function, or permission check. This reveals which routes and services are affected by a single model change — a direct measure of risk scope.

Read [evidence-and-analysis.md](references/evidence-and-analysis.md) for the risk-first review procedure.

### Phase 5: Verify Claims

Run the repository's documented checks where possible. Prefer the narrowest command that proves each claim, then run the full relevant suite. Inspect failures rather than suppressing them. For claims requiring external services, production data, credentials, or a browser, label them **not verified in this environment** and state the exact rehearsal needed.

**With Codegraph:** Use `codegraph affected <changed-files>` to identify which test files are linked to each source file under review. This narrows verification scope and surfaces tests that grep cannot easily correlate.

Maintain a coverage matrix: claim | audience | implementation source | evidence command/test | result | status | gap/owner. A claim without evidence is a documentation defect.

### Phase 6: Lock Terminology and Documentation Decisions

Before large-scale drafting, create or update `DOCUMENTATION_PLAN.md`. Lock:

- audience, scope, and **output language** (English or Chinese);
- status vocabulary: Verified / Implemented-not-verified / Partially-verified / Contradicted / Planned;
- citation style (`path/to/file.py:123` or `Class.method`);
- code-excerpt policy (minimum size, redaction rules);
- privacy redaction rules and output format.

Maintain `GLOSSARY.md` for domain terms, UI labels, state names, and code identifiers. Do not introduce competing synonyms for a locked term without recording a decision.

### Phase 7: Draft the Document

Use the structure in [document-schema.md](references/document-schema.md). Write user-facing explanations in the declared output language, then provide an implementation and audit view. Preserve product terminology.

**English output:** Write in clear technical English. If the UI or source uses another language, include a glossary section and preserve exact UI labels in their original language for accuracy.

**Chinese output:** Draft fully in Chinese. Code identifiers, routes, configuration keys, exact error strings, and log messages remain in their original language. Apply the humanize pass immediately after each major section (Phase 7.5) rather than as a final batch step.

Use stable links such as `path/to/file.py:123`. Include version/date scope on the title page. Prefer tables for matrices and concise sequence diagrams for multi-step flows.

### Phase 7.5: Humanize (Chinese Output Only)

Skip this phase entirely if the output language is English.

**With Humanizer-zh:**
```bash
# If not already installed
git clone https://github.com/op7418/Humanizer-zh
cd Humanizer-zh && pip install -r requirements.txt

# Process each major section
python3 humanizer.py --input <section-file.md> --output <section-humanized.md>
```

After tool processing: diff the original and humanized output. Restore any passage where the tool weakened a claim status, removed a stated limitation, collapsed a Partially-verified label, or altered a code identifier. Log the tool name, version, and sections processed.

**Without Humanizer-zh:** Apply the Claude-native ZH humanization checklist in [humanize-zh-pass.md](references/humanize-zh-pass.md). This eliminates the eight most common patterns that make AI-generated Chinese technical documentation identifiable as machine-written.

Humanization does not reopen evidence or claim-status gates. If a humanized passage collapses a "Partially-verified" label into a bare assertion, restore the precise status and log the conflict.

### Phase 8: Validate

Before declaring completion, verify:

- Can a new user complete every supported workflow from the document **using only the numbered procedures from Phase 3.5**?
- Does every user-facing route have a procedure with all four elements (preconditions, steps, expected result, error path)?
- Do error paths use the exact error strings from the source code (not paraphrased versions)?
- Can an auditor trace each material claim to code and executable evidence?
- Are permissions, negative paths, data mutations, and external side effects covered?
- Are configuration, deployment, backup/restore, monitoring, and incident procedures covered?
- Are limitations and unverified gates conspicuous?
- Does the MECE inventory reconcile against routes, models, jobs, commands, templates, and tests?
- Are all examples safe, current, and free of real credentials or personal data?
- If Chinese output: is the humanize pass complete for every section?

After each major capability area, validate links, code references, terminology, table consistency, and evidence status before proceeding. Append the checklist result to `DOCUMENTATION_LOG.md`.

If any answer is no, either close the gap with more inspection/evidence or list it explicitly in the open-gaps register. Never call the document complete while hidden gaps remain.

## Output Contract

Produce, in the declared output language:

- one primary document (normally `docs/system-documentation.md` or the requested format);
- a claim/evidence coverage matrix;
- an open-gaps and verification register;
- a short change log identifying the source revision and verification date.

For multi-session work, also maintain the harness files in [harness-files.md](references/harness-files.md). They are working controls, not substitutes for the primary document.

Do not modify application code unless the user explicitly asks. Do not invent behavior to make documentation look complete. When a generated artifact is requested, render or validate it using the relevant artifact skill and report the verification result.

## Completion Gate

Only state that documentation is complete when:

1. the document exists and covers every MECE capability area;
2. the MECE reconciliation is recorded in `DOCUMENTATION_LOG.md`;
3. all high-risk workflows have positive and at least one boundary/negative evidence entry;
4. all tests/checks that were runnable pass or are explained;
5. every remaining external dependency is explicitly marked with a concrete verification procedure;
6. if Chinese output: every section has a humanize log entry and no open claim-status regressions remain.

## Portability and Safety

Use repository-relative references where possible so the document remains useful after cloning. Do not depend on a particular shell, editor, model, or MCP server unless the project requires it. Avoid writing secrets, personal data, tokens, private URLs, or production identifiers into excerpts, fixtures, screenshots, logs, or examples. If sensitive evidence is necessary, describe its existence and location without copying the value.
