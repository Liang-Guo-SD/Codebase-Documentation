# Codebase Documentation Skill

Generates detailed, evidence-backed documentation for an existing software project. Serves three audiences simultaneously:

- **users** who need to operate the system;
- **auditors** who need to trace behavior and controls to evidence;
- **maintainers** who need to understand architecture, invariants, tests, and operations.

Supports English and Chinese output. The full pipeline is:

```
Codegraph indexes the repo  →  skill analyzes code & drafts documentation
                            →  humanizer-zh polishes Chinese prose (if ZH output)
                            →  final document
```

---

## Before you begin

Two tools are optional but strongly recommended depending on your scenario. **Install them before invoking the skill** — the skill detects their presence at startup and adjusts its workflow automatically.

| Your situation | Install |
|---|---|
| Repo has > ~150 files | Codegraph |
| Output language is Chinese | humanizer-zh |
| Both | Both |
| Small English-only repo | Neither — invoke the skill directly |

### Step 1 — Install Codegraph (large repos)

Codegraph pre-indexes the repository's symbol graph and exposes it to the skill via MCP. It replaces 40+ grep/Read discovery calls with 2–4 targeted queries, cutting token usage ~60–70% on medium-to-large codebases.

```bash
# One-time machine install (requires Node 22 or 24)
npm install -g @colbymchenry/codegraph
codegraph install          # registers the MCP server with Claude Code
```

Restart Claude Code after `codegraph install`. Then, from your **project root**:

```bash
codegraph init             # builds the index; writes to .codegraph/
codegraph status           # verify: "Backend: native" with non-zero symbol count
```

If `codegraph status` shows zero symbols, re-run `codegraph init`. You only need to do this once per project; the index stays up to date automatically via a file watcher.

### Step 2 — Install humanizer-zh (Chinese output)

op7418/Humanizer-zh removes AI-writing artifacts from Chinese technical prose — over-formal connectors, nominalized actions, pronoun intrusion, long breathless sentences — without altering claim status labels or code identifiers.

```bash
git clone https://github.com/op7418/Humanizer-zh
cd Humanizer-zh && pip install -r requirements.txt
```

Place the cloned directory somewhere accessible (e.g. `~/tools/Humanizer-zh`). The skill will locate it via `python3 -c "import humanizer"` and invoke it automatically after drafting each major section. If it is absent, the skill applies a built-in Claude-native ZH checklist instead.

---

## First invocation

Once setup is complete, open Claude Code in your project root and invoke the skill:

**English documentation (default):**

```text
Use $codebase-documentation to document this project. Produce a complete system
document for users, maintainers, and auditors. Verify every runnable claim and
list all unverified operational gates.
```

**Chinese documentation:**

```text
Use $codebase-documentation to document this project in Chinese. Produce a complete
system document for users, maintainers, and auditors.
```

The skill will:
1. Detect whether Codegraph and humanizer-zh are available, and log the result.
2. Create harness files under `docs/` to track progress across sessions.
3. Lock terminology and language decisions in `docs/DOCUMENTATION_PLAN.md` before drafting.
4. Draft the document, then apply humanizer-zh (or the built-in ZH checklist) if Chinese output.

**Focused scope example:**

```text
Use $codebase-documentation to document authentication, enrollment, grading,
email notifications, backups, and database deployment in this project, in Chinese.
```

---

## Output language policy

The skill defaults to **English**. To get Chinese output, say "in Chinese" in your request — the skill then drafts fully in Chinese, keeps code identifiers, routes, config keys, and exact error strings in their original form, and applies the humanize pass before completing each section.

Chinese and English use entirely separate post-processing paths. Codegraph operates the same way regardless of output language.

---

## What "complete" means

The skill must not announce completion merely because prose was written. Completion requires:

1. every material capability is mapped exactly once using a MECE partition;
2. high-risk workflows have positive and negative/boundary evidence;
3. code, tests, configuration, and runtime claims are traceable to file and symbol;
4. runnable checks have exact commands and results;
5. external gates — live database, SMTP credentials, browser checks, backup restore — are explicitly marked if not verified in this environment;
6. no secrets, personal data, or production identifiers are copied into the documentation;
7. a new session can resume from the harness files without relying on conversation memory;
8. if Chinese output: every section has a completed humanize log entry with no open claim-status regressions.

---

## Harness files

For multi-session work the skill creates and maintains these files under `docs/`:

```
docs/DOCUMENTATION_PLAN.md       — audience, scope, language, terminology decisions
docs/DOCUMENTATION_PROGRESS.md   — current phase, next unit, blockers
docs/GLOSSARY.md                  — locked terms, UI labels, code identifiers
docs/DOCUMENTATION_LOG.md         — append-only log of every action and result
```

The primary deliverable is:

```
docs/system-documentation.md
```

If a session is interrupted, start a new session and the skill will read `DOCUMENTATION_PROGRESS.md` first and resume from exactly where it stopped.

---

## Skill contents

| File | Purpose |
|---|---|
| `SKILL.md` | Operating instructions for Claude |
| `references/document-schema.md` | Default 14-section document structure |
| `references/evidence-and-analysis.md` | Evidence hierarchy, risk analysis, MECE checks |
| `references/harness-files.md` | Harness file templates |
| `references/codegraph-guide.md` | Codegraph installation, MCP call sequences, per-phase usage |
| `references/humanize-zh-pass.md` | humanizer-zh invocation, Claude-native ZH checklist, regression gate |
| `agents/openai.yaml` | UI metadata and default prompt |
