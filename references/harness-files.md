# Documentation Harness Files

Use these files for documentation efforts that are large, iterative, or likely to cross context boundaries. Keep them in the project documentation directory, normally `docs/`.

## `DOCUMENTATION_PLAN.md`

The locked execution plan. Include:

- purpose, audiences, scope, exclusions, and target output;
- source revision and evidence boundary;
- capability partition and reconciliation method;
- terminology and localization decisions;
- evidence standard and status vocabulary;
- high-risk workflows and required negative tests;
- expected commands, environments, and external prerequisites;
- acceptance criteria and document owner.

Change decisions deliberately. Append a dated decision note when a locked choice changes.

## `DOCUMENTATION_PROGRESS.md`

The minimal cold-resume state. Keep it short and machine-readable where practical:

```text
phase: inventory | analysis | drafting | verification | complete
next_unit: <capability, workflow, or review item>
source_revision: <commit/tag/date>
capabilities_complete: <number>/<number>
high_risk_complete: <number>/<number>
last_verified: <ISO date>
blockers: <none or explicit list>
```

Update it after each bounded work unit, including failures and external blockers. Never mark a unit complete without the corresponding evidence entry.

## `GLOSSARY.md`

Use a table with `Term | Meaning | Preferred English | UI/localized label | Code identifier | Source`. Lock terms that affect user instructions, state transitions, permissions, or audit interpretation. Preserve exact code identifiers and exact user-visible strings when they are contracts.

## `DOCUMENTATION_LOG.md`

Use an append-only record:

`date | phase/unit | action or command | result | evidence references | unresolved gap`

Record commands exactly enough for another maintainer to rerun them. Include environment versions and external-service limitations. Never replace a failed result with a later success; append the later result.

## Optional generated artifacts

For large systems, create a route/model/test inventory and a link-check report, but keep generated inventories subordinate to the primary document. Do not add scripts solely for one-off prose editing; add a script only when deterministic validation is repeated or fragile.
