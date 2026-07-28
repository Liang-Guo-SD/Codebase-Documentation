# Evidence and First-Principles Analysis

## Evidence hierarchy

Use the strongest available source and disclose conflicts:

1. observed runtime behavior with a reproducible command or recording;
2. passing automated test that exercises the claim;
3. executable implementation and configuration;
4. schema, migration, or interface contract;
5. product requirement, design, or comment;
6. inference or assumption.

Levels 5 and 6 describe intent, not proof of current behavior. A test that only mocks the relevant boundary proves less than an integration test. A code path that cannot run due to missing infrastructure is **implemented, not operationally verified**.

## Claim statuses

- **Verified:** directly supported by a passing relevant test or reproducible observation.
- **Implemented, not verified:** code/configuration exists, but required environment or evidence was unavailable.
- **Partially verified:** happy path is proven but an important alternate, security, or failure path is not.
- **Contradicted:** sources disagree; document the discrepancy and do not choose silently.
- **Planned/absent:** no implementation was found.

## MECE reconciliation checks

Reconcile the capability map against four independent inventories:

- all user-facing routes/pages/templates;
- all domain models, migrations, and state-changing services;
- all background jobs, CLI commands, and external integrations;
- all tests and release/operations checks.

Every item must map to one primary capability and one status. Investigate items with no map, capabilities with no implementation, implementation with no user explanation, and tests with no documented claim.

## Risk-first review

Prioritize workflows that can cause irreversible data loss, privilege changes, financial/academic/legal consequences, disclosure of personal data, external messages, or cross-tenant access. For each, require a positive test and at least one negative or boundary test. Document authorization before mutation, transaction/rollback behavior, retries, duplicate submission behavior, and audit evidence.

## Code excerpt rules

Include only the smallest excerpt that proves the point. Annotate it with file and symbol, explain inputs/outputs and why it matters, and link to the test. Prefer pseudocode or a line reference when an excerpt would expose secrets, overwhelm readers, or become stale. Never include credentials, production identifiers, personal data, or generated bundles.

## Gap register fields

Use: `ID | gap/claim | affected audience | risk | current status | missing evidence or implementation | exact closure action | owner | target/version`. A vague note such as “needs more testing” is insufficient.
