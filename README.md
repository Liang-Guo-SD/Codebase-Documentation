# Codebase Documentation Skill

This skill creates detailed, evidence-backed English documentation from an existing software project. It is designed for three audiences:

- users who need to operate the system;
- auditors who need to trace behavior and controls to evidence;
- maintainers who need to understand architecture, invariants, tests, and operations.

## Use it in `Edu_System`

The skill package is located at:

```text
/Users/liangguo/Edu_System/skills/codebase-documentation/
```

To use it with Codex, invoke it explicitly:

```text
$codebase-documentation
```

Then provide a task such as:

```text
Use $codebase-documentation to document this Edu_System project.

Analyze the source code, templates, tests, configuration, deployment files,
and existing docs. Produce a complete English system document for users,
maintainers, and auditors. The UI may remain Chinese, but all development and
audit documentation must be English. Verify every runnable claim and list all
unverified operational gates.
```

You can also ask for a focused scope:

```text
Use $codebase-documentation to document authentication, enrollment, grading,
SMTP configuration, backups, and PostgreSQL deployment in this project.
```

## Recommended Edu_System workflow

Run the skill from the project root:

```text
/Users/liangguo/Edu_System
```

The skill should inspect, at minimum:

- `docs/PRD.md`, `docs/acceptance-matrix.md`, and existing acceptance evidence;
- `edu_system/` routes, services, models, templates, CLI, mailer, and operations;
- `tests/` and the dependency/deployment files;
- environment configuration, migrations, backups, and security controls.

For a large or multi-session documentation effort, ask the skill to create or maintain these files under `docs/`:

```text
docs/DOCUMENTATION_PLAN.md
docs/DOCUMENTATION_PROGRESS.md
docs/GLOSSARY.md
docs/DOCUMENTATION_LOG.md
```

The primary deliverable is normally:

```text
docs/system-documentation.md
```

The document should include the user guide, capability map, architecture, workflows, data model, interface contracts, security/privacy, operations, testing/evidence, troubleshooting, glossary, and open gaps.

## What “complete” means

The skill must not announce completion merely because prose was written. Completion requires:

1. every material capability is mapped exactly once using a MECE partition;
2. high-risk workflows have positive and negative/boundary evidence;
3. code, tests, configuration, and runtime claims are traceable;
4. runnable checks have exact commands and results;
5. external gates—such as live PostgreSQL, SMTP credentials, browser checks, or backup restore—are explicitly marked if not verified;
6. no secrets, personal data, or production identifiers are copied into the documentation;
7. a new session can resume from the documentation harness files.

## Important distinction

This skill documents the existing implementation. It does not silently implement missing features or convert planned behavior into stated behavior. If the PRD and code disagree, the documentation must show the discrepancy and identify current behavior.

## Skill contents

- `SKILL.md` — operating instructions for Codex;
- `references/document-schema.md` — default document structure;
- `references/evidence-and-analysis.md` — evidence hierarchy, risk analysis, and MECE checks;
- `references/harness-files.md` — multi-session planning, progress, glossary, and logging conventions;
- `agents/openai.yaml` — UI metadata and default invocation prompt.
