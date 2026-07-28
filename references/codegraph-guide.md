# Codegraph Integration Guide

Codegraph builds a pre-indexed knowledge graph of a codebase using tree-sitter AST parsing and stores it in a local SQLite database. AI agents query this graph through two MCP tools instead of running dozens of grep/glob/Read operations. Across real-world codebases it reduces tool calls by ~70–89% and token usage by ~60–70%.

Use Codegraph when: the repo has more than ~150 files, you are documenting across multiple sessions, or the codebase spans multiple modules with complex call relationships. Skip it for small repos, purely runtime-behavior questions, or when the MCP integration is not active.

## Installation (once per machine)

```bash
npm install -g @colbymchenry/codegraph   # requires Node 22 or 24
codegraph install                         # registers the MCP server with Claude Code
```

Restart Claude Code after `codegraph install`.

## Per-project initialization

```bash
cd <project-root>
codegraph init        # builds the full index; writes to .codegraph/
codegraph status      # verify: should show "Backend: native" and non-zero symbol count
```

If `codegraph status` shows zero symbols, re-run `codegraph init`. The index auto-syncs via a file watcher (2-second debounce) so you do not need to re-run `init` after code changes during a documentation session.

## MCP Tools

| MCP tool | Purpose |
|---|---|
| `codegraph_context <query>` | Returns relevant symbols, source code, and call paths for a natural-language query. Primary discovery tool. |
| `codegraph_explore <symbol-or-query>` | Examines a specific symbol with its callers, callees, and source. Use for tracing a workflow. |

## CLI Commands (for fallback or verification)

| Command | When to use |
|---|---|
| `codegraph query <term>` | Full-text search across symbol names — find all symbols related to a capability area |
| `codegraph node <symbol>` | Retrieve one symbol with its callers and callees |
| `codegraph callers <symbol>` | Find everything that calls a specific function or method |
| `codegraph impact <symbol>` | Map the blast radius of changing a symbol — all downstream dependents |
| `codegraph affected <file> [<file>...]` | Identify test files linked to source files under review |
| `codegraph explore <query>` | Return connected call paths across files for a natural-language query |
| `codegraph sync` | Force an incremental index update if auto-sync seems stale |

## Per-Phase Usage in the Documentation Workflow

### Phase 1 — Establish the evidence boundary

Replace the broad grep/glob/find survey with two MCP calls:

```
codegraph_context("entry points routes controllers services models migrations jobs CLI")
codegraph_context("background workers scheduled tasks integrations external APIs")
```

Each call returns grouped source code with line numbers. Use the output to build the file and symbol inventory that Phase 1 requires. Supplement with a targeted `Read` only for files the tool returns that need full-context inspection.

### Phase 2 — MECE capability inventory

For each capability area, query by domain keyword to find all related symbols:

```
codegraph_context("authentication authorization login session")
codegraph_context("enrollment assignment submission grading")
```

Cross-check the returned symbol list against the route table and model list. Symbols with no documentation candidate are orphaned code — record them in the open-gaps register.

### Phase 3 — Trace the system from interface to effect (Code View)

For each important workflow, start at the route or entry point and trace downward:

```
codegraph_explore("<route-handler-function>")
```

The tool returns the full call path across files. For each intermediate symbol, use:

```
codegraph_explore("<service-or-domain-function>")
```

To verify all callers of a shared function (e.g., a permission check used in multiple routes):

```bash
codegraph callers <function-name>
```

This ensures you document every context in which a critical function is invoked.

### Phase 3.5 — Reconstruct user-facing procedures

Codegraph output from Phase 3 gives you the code path. Phase 3.5 reverses the lens to produce what the user does. For each route found in Phase 3:

1. **Map form fields to procedure steps.** Every `request.form.get(...)` and `request.files.get(...)` in the POST handler is a user action. List them as numbered steps using the field's label from the template, not the form key name.

2. **Extract exact error messages.** Search the code path for `flash(..., "error")` and non-2xx `return`/`abort()` calls. Record the exact string. Codegraph `codegraph_explore` already returns these inline — do not paraphrase them.

3. **Identify the happy-path endpoint.** Find the success-branch `redirect(url_for(...))` or `render_template(...)`. This tells you which page the user lands on after a successful operation.

4. **Identify preconditions from auth gates.** Every `@require_role(...)`, `abort(403)`, `abort(404)`, and early `if not` guard in the code path is a precondition. Translate it to user language (e.g., "You must be enrolled in this course" rather than "`_ensure_enrolled` must return True").

**No Codegraph query specifically maps to procedure generation.** The input comes from the code paths already captured in Phase 3. Use `codegraph_explore` a second time on the same route if the first capture was truncated and you need the error paths.

### Phase 4 — Analyze data, trust, and risk

Before writing a risk assessment for a data model or auth function, map its blast radius:

```bash
codegraph impact <model-class-or-auth-function>
```

The output lists all downstream symbols affected by a change. A large blast radius signals a high-risk component that warrants positive and boundary tests in Phase 5.

### Phase 5 — Verify claims

To identify which tests are linked to a set of source files you are documenting:

```bash
codegraph affected <path/to/service.py> <path/to/model.py>
```

This surfaces test files that grep cannot correlate without reading every test's imports. Use the output to narrow the `pytest` or `unittest` run to the relevant subset.

## Verification

After initialization, confirm the integration is active:

```bash
codegraph status
```

Expected output: `Backend: native` with a non-zero symbol count. If you see only grep/glob/Read calls during a session with no Codegraph MCP calls, the MCP integration is not active — re-run `codegraph install` and restart Claude Code.

## Supported Languages

Codegraph supports 14+ languages including Python, TypeScript, JavaScript, Go, Rust, Java, C++, Ruby, PHP, Swift, and Kotlin. Language detection is automatic; `.gitignore` patterns are honored.

## Privacy

The index is stored entirely in `.codegraph/` within your project directory. No code or metadata is sent to external services. The `.codegraph/` directory should be added to `.gitignore` for most projects.
