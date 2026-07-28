# Chinese Humanize Pass for Technical Documentation

Apply this pass after drafting each major section in Chinese. The goal is to eliminate patterns that make AI-generated technical documentation identifiable as machine-written, without weakening claim status, altering code identifiers, or removing necessary caveats.

This pass is distinct from a translation humanization pass: the source is AI-generated Chinese, not translated English. The dominant artifacts are different — they come from generation patterns, not cross-language calques.

## Tool Invocation: op7418/Humanizer-zh

```bash
# Install if not present
git clone https://github.com/op7418/Humanizer-zh
cd Humanizer-zh && pip install -r requirements.txt

# Process a section
python3 humanizer.py --input <section.md> --output <section-humanized.md>
```

After tool processing, diff the two files and restore any passage where the tool:
- Changed a claim status word (已验证 / 部分验证 / 未验证 / 计划中);
- Removed a stated limitation or caveat;
- Altered a code identifier, route, config key, or exact error string;
- Changed a version number, date, or command.

Log: `section | tool: humanizer-zh vX.Y.Z | passages restored: N | status: complete`.

## Claude-Native Checklist (when tool is absent or as a supplement)

Read through each paragraph and fix every instance of the patterns below.

### Group 1: AI Generation Sentence Patterns

These are the clearest signals of machine-generated Chinese technical prose.

| Pattern | Bad example | Fix |
|---|---|---|
| Starting every paragraph with topic noun | 该系统的认证模块提供了…… / 该模块的功能包括…… | Vary openings; restructure to verb-first or context-first |
| Enumeration without contrast | 首先…… 其次…… 再次…… 最后……（in every section） | Use only when steps are genuinely sequential; otherwise cut the connectors |
| Empty intensifiers | 非常重要的是 / 极为关键的 / 十分显著的 | Cut or replace with the specific fact that makes it important |
| Parallel structure overuse | A负责X，B负责Y，C负责Z（repeated for every item） | Collapse into a table or restructure one non-parallel sentence |
| "本文档将介绍" / "本节将说明" | 本节将详细介绍认证模块的实现方式。 | 认证模块通过以下方式实现。（start with the content, not the announcement） |

### Group 2: Hedging Stacks

Technical documentation requires precise claim status words. These are different from unnecessary hedges.

- **Keep:** 该行为在生产环境中未经验证 / 此功能已部分实现 / 以下配置依赖于外部服务
- **Cut:** 该功能可能在某些情况下或许能够正常运行（three hedges on one claim)
- **Rule:** one hedge per claim. If a claim is uncertain, use the exact status vocabulary locked in `DOCUMENTATION_PLAN.md`.

### Group 3: Connector Overuse

| Overused | Rotate with |
|---|---|
| 此外 (more than twice per section) | 另外 / 同时 / 还需注意 |
| 因此 (sentence-initial) | 于是 / 从而 / 这意味着 |
| 然而 (more than twice per section) | 不过 / 但 / 尽管如此 |
| 需要注意的是 (more than once per page) | 注意 / 值得注意的是 / 应当指出 |

### Group 4: Nominalization of Technical Actions

Chinese prefers verb-phrase forms. Nominalization often comes from following code comment structure.

| Bad (nominalization) | Good (verb form) |
|---|---|
| 对请求的验证的执行 | 执行请求验证 |
| 数据库连接的建立与管理 | 建立并管理数据库连接 |
| 对用户权限的检查 | 检查用户权限 |
| 文件上传的处理流程 | 处理文件上传 |

### Group 5: Subject Pronoun Intrusion

Chinese technical documentation does not use first-person pronouns. Remove them.

| Remove | Replace with |
|---|---|
| 我们可以通过以下命令…… | 通过以下命令…… |
| 用户可以看到系统将…… | 系统将…… |
| 开发者需要注意，系统会…… | 系统会…… |

### Group 6: Long Sentences from Code Structure

When documenting a function with five parameters, the temptation is to describe all five in one sentence. In Chinese, this reads as breathless.

- If a sentence exceeds ~50 Chinese characters and contains more than two embedded clauses, split it.
- After a colon (：), the following text should be a complete thought, not a noun phrase fragment.
- Avoid 逗号 splices where a 句号 or 分号 belongs.

### Group 7: Code-Adjacent Prose Transitions

These bridge prose and code blocks and are commonly over-formal or over-casual.

| Bad | Good |
|---|---|
| 以下代码演示了上述功能的具体实现方式。 | 实现如下： |
| 运行该命令将会输出以下内容。 | 运行后输出如下。 |
| 如上面的代码片段所示，系统使用了…… | 该实现使用了…… |

### Group 8: Claim Status Vocabulary Consistency

Before finishing the pass, verify that every instance of the following terms matches the vocabulary locked in `DOCUMENTATION_PLAN.md`:

- 已验证 / 实现但未验证 / 部分验证 / 已计划 / 已废弃
- 不适用 / 依赖外部环境 / 暂无测试覆盖

Do not paraphrase these status terms in humanized passages. They are contracts, not prose.

## Rigor Gate Regression Check

After the humanize pass for each section, verify:

1. All claim status markers are present and use the exact locked vocabulary.
2. Hypothesis and interpretation labels have not been softened into assertions.
3. Stated limitations, missing evidence, and unverified external gates are unchanged.
4. Code identifiers, routes, config keys, and exact error strings are identical to the pre-humanize draft.
5. Version numbers, dates, and commands are unchanged.

If a regression is found: restore the precise pre-humanize text for that passage and log the restoration.

## Log Entry Format

```
date | section | gate: humanize | tool: humanizer-zh vX.Y | passages restored: N | status: complete
date | section | gate: humanize | tool: claude-native | passages restored: N | status: complete
```

The humanize pass for a section is complete only when its log entry exists and no regression is open.
