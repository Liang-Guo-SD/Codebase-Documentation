# Documentation Schema

Use this as the default table of contents. Adapt names to the product, but preserve the evidence and gap sections.

## 1. Document control

Title, product/version or commit, scope, audience, language policy, owner, last verified date, source boundary, and status. State whether the document describes current behavior, a target design, or both; never mix them without labels.

## 2. Executive summary

Purpose, users, business outcomes, principal workflows, deployment shape, and the most important limitations. Keep this readable by a non-technical reviewer.

## 3. User guide

For each actor and capability: goal, prerequisites, exact steps, visible result, alternate/empty/error states, permissions, and troubleshooting. Include navigation labels and localized UI terms where relevant.

## 4. Capability map

Provide the MECE inventory and actor-to-capability matrix. Every capability must have an implementation reference and evidence status.

## 5. System architecture

Context diagram, components, request/job flows, trust boundaries, runtime dependencies, configuration, and deployment topology. Identify where state is authoritative.

## 6. Workflow and business rules

Use one subsection per high-value workflow. For each: trigger, preconditions, state machine, validation, authorization, persistence, side effects, retries/idempotency, failure modes, and relevant code/test references.

## 7. Data and domain model

Explain entities, fields that matter to users or auditors, relationships, constraints, ownership, lifecycle, migrations, retention, deletion, export, and sensitive-data classification. Add an entity relationship diagram when relationships are not clear in a table.

## 8. Interface contracts

Document web routes, APIs, forms, CLI commands, jobs, imports/exports, file formats, status codes, validation errors, pagination, versioning, and compatibility assumptions. Do not expose secrets or tokens.

## 9. Security, privacy, and abuse resistance

Cover authentication, sessions, authorization, CSRF, injection/output encoding, uploads, secrets, encryption, rate limits, logging, personal data, tenant isolation, threat assumptions, and residual risks. Distinguish tested controls from asserted controls.

## 10. Operations

Installation, configuration, startup, migrations, health checks, logs, monitoring, scheduled tasks, backups, restore rehearsal, upgrades, rollback, incident response, and capacity assumptions. Mark environment-dependent steps.

## 11. Quality and verification

Test strategy, test inventory, commands, fixtures, coverage limitations, static checks, browser/manual checks, external-service checks, and a claim/evidence matrix. Record exact results and dates.

## 12. Failure modes and troubleshooting

Organize by symptom, likely cause, diagnostic evidence, safe remediation, escalation, and data-safety warning. Include expected behavior for empty, duplicate, stale, unauthorized, and unavailable states.

## 13. Glossary and terminology

Define domain terms, actor names, state names, code identifiers, localized labels, and abbreviations. Flag terms with different user and implementation meanings.

## 14. Open gaps and change log

List missing evidence, planned features, known defects, assumptions, external prerequisites, owner, risk, and exact closure test. End with the source revision and verification record.
