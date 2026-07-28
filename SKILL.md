---
name: codebase-documentation
description: Generate detailed, evidence-backed English documentation from a project's source code, tests, configuration, and runtime behavior for end users, maintainers, and auditors.
---

# Codebase Documentation

Use this skill when a project needs a trustworthy documentation package that explains what users can do and how the system actually works. It is for existing codebases, not speculative product briefs. The output must be in English even when the product UI or source comments use another language. It is composable with artifact-specific skills: use those when the requested deliverable is DOCX, PDF, spreadsheet, or presentation.

## Concrete use cases and success criteria

This skill supports three related use cases:

1. **User enablement:** a new user can complete supported workflows, understand prerequisites, and recover from common failures without reading source code.
2. **Audit and assurance:** an auditor can trace material behavior, controls, data mutations, and operational claims to source and reproducible evidence.
3. **Engineering continuity:** a maintainer can locate ownership, invariants, integration boundaries, tests, configuration, and known gaps from a cold session.

Success requires all three audiences to be served by one consistent source of truth. Assess the result against these observable criteria: every material capability is mapped exactly once; every high-risk claim has implementation and verification references; every unverified external dependency is named with a closure procedure; runnable checks are recorded with exact results; and a new session can resume from the harness files without relying on conversation memory.

## Governing principles

The document is a claim-to-evidence map, not a prose rewrite of the repository.

1. **Code is the behavioral authority.** Treat PRDs, README files, and comments as intent; resolve conflicts against executable behavior and tests.
2. **Separate audiences without duplicating facts.** Explain the same capability first as a user outcome, then as an auditable implementation contract.
3. **Make scope explicit.** Distinguish implemented, partially implemented, configured-but-unverified, deprecated, and planned behavior.
4. **Never promote inference to fact.** Mark assumptions, unknowns, environment-dependent behavior, and evidence gaps.
5. **Reproducibility is part of documentation.** Every material claim needs a source path, symbol/route/config key, test, command, or observed runtime result.

Read [document-schema.md](references/document-schema.md) before drafting. Read [evidence-and-analysis.md](references/evidence-and-analysis.md) when deciding what counts as evidence, how to analyze risk, or how to handle gaps. Use [harness-files.md](references/harness-files.md) when the documentation effort spans multiple sessions or large repositories.

## Authority order

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

### 0. Detect phase and resume safely

At the start of every invocation, look for `DOCUMENTATION_PROGRESS.md`. If present, read it before inspecting or editing outputs. Continue only from its recorded phase and next unit of work. If absent, create the harness files described in [harness-files.md](references/harness-files.md) and enter `SETUP`. Never redo completed units merely because the conversation was compacted.

### 1. Establish the evidence boundary

Inspect the repository before writing. Identify:

- product requirements, ADRs, README files, release notes, and existing documentation;
- application entry points, routes/controllers, services, models, migrations, jobs, CLI commands, templates, assets, and integrations;
- dependency manifests, environment examples, deployment files, permissions, secrets handling, and data stores;
- unit, integration, end-to-end, security, migration, and operational tests;
- CI scripts and available local commands.

Record the commit/version, date, inspected paths, commands run, and environment limitations in the log. Do not hide untracked or generated files that materially affect behavior.

### 2. Build a MECE capability inventory

Classify every user-visible or operational capability exactly once in a primary capability area. Use this partition unless the product clearly requires a better one:

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

### 3. Trace the system from interface to effect

For each important workflow, follow the actual path:

`user/trigger → route or entry point → validation/auth → domain/service logic → persistence → side effect → response/UI`

Name relevant files and symbols. Explain invariants, state transitions, idempotency, transaction boundaries, retries, and failure recovery. Include short, targeted code excerpts only when they clarify a security boundary, business rule, data transformation, or integration contract; never dump whole files or secrets.

### 4. Analyze data, trust, and risk

Document the domain model and lifecycle of important records: fields, relationships, constraints, ownership, retention/deletion behavior, and sensitive data. For every trust boundary, explain input validation, authorization, output encoding, secret handling, file handling, rate limiting, auditability, and known residual risk. Identify high-risk logic and point to tests that exercise it.

### 5. Verify claims

Run the repository's documented checks where possible. Prefer the narrowest command that proves each claim, then run the full relevant suite. Inspect failures rather than suppressing them. For claims that require external services, production data, credentials, a live database, or a browser, label them **not verified in this environment** and state the exact rehearsal needed.

Maintain a coverage matrix with columns: claim, audience, implementation source, evidence command/test, result, status, and gap/owner. A claim without evidence is a documentation defect.

### 6. Lock terminology and documentation decisions

Before large-scale drafting, create or update `DOCUMENTATION_PLAN.md`. Lock the audience, scope, status vocabulary, citation style, code-excerpt policy, localization policy, privacy redaction rules, and output format. Maintain `GLOSSARY.md` for domain terms, UI labels, state names, and code identifiers whose wording must remain consistent. Do not introduce competing synonyms for a locked term without recording a decision.

### 7. Draft the English document

Use the structure in [document-schema.md](references/document-schema.md). Write user-facing explanations in plain language, then provide an implementation and audit view. Preserve product terminology and include a glossary. If the UI is localized, document the UI language separately; do not translate code identifiers, routes, configuration keys, or exact error strings inaccurately.

Use stable links such as `path/to/file.py:123` or `Class.method`. Include version/date scope on the title page. Prefer tables for matrices and concise sequence diagrams for multi-step flows.

### 8. Perform bounded validation and independent completeness review

Before declaring completion, ask:

- Can a new user complete every supported task from the document?
- Can an auditor trace each material claim to code and executable evidence?
- Are permissions, negative paths, data mutations, and external side effects covered?
- Are configuration, deployment, backup/restore, monitoring, and incident procedures covered?
- Are limitations and unverified gates conspicuous?
- Does the MECE inventory reconcile against routes, models, jobs, commands, templates, and tests?
- Are all examples safe, current, and free of real credentials or personal data?

After each major capability area, validate links, code references, terminology, table consistency, and evidence status before proceeding. At the end, perform the checklist below and append the result to `DOCUMENTATION_LOG.md`.

If any answer is no, either close the gap with more inspection/evidence or list it explicitly in the open-gaps register. Never call the document complete while hidden gaps remain.

## Output contract

Produce, in English:

- one primary document (normally `docs/system-documentation.md` or the requested format);
- a claim/evidence coverage matrix;
- an open-gaps and verification register;
- a short change log identifying the source revision and verification date.

For multi-session work, also maintain the harness files in [harness-files.md](references/harness-files.md). They are working controls, not substitutes for the primary document.

Do not modify application code unless the user explicitly asks. Do not invent behavior to make documentation look complete. When a generated artifact is requested, render or validate it using the relevant artifact skill and report the verification result.

## Completion gate

Only state that documentation is complete when the document exists, the MECE reconciliation is recorded, all high-risk workflows have evidence, all tests/checks that were runnable pass or are explained, and every remaining external dependency is explicitly marked with a concrete verification procedure.

## Portability and safety

Use repository-relative references where possible so the document remains useful after cloning. Do not depend on a particular shell, editor, model, or MCP server unless the project requires it. Avoid writing secrets, personal data, tokens, private URLs, or production identifiers into excerpts, fixtures, screenshots, logs, or examples. If sensitive evidence is necessary, describe its existence and location without copying the value.
