# RuleWeaver

Versioned, explainable business rules with predictable evaluation.

> Planning repository — documentation and Git initialization only. No application, package, solution, database, tests, CI workflow, or runnable demo has been implemented yet. Everything below describes the proposed implementation, unless explicitly marked as verified.

## Goals

Let developers define bounded decision logic outside application code while retaining validation, version history, and a clear explanation of each result. Emphasize an auditable rules language over unrestricted scripting.

## Target users

Application developers implementing eligibility, routing, validation, or configurable decision tables; analysts reviewing why a result occurred.

## MVP scope

- JSON rule sets containing typed comparisons and all/any/not composition.
- Schema validation and explicit handling for missing fields, null values, and incompatible types.
- Immutable rule versions with draft validation and publish operations.
- Pure evaluation with match results and a structured explanation tree.
- A playground to evaluate fixtures against two versions and inspect behavior changes.

## Stretch goals

- Decision-table editor and conflict analysis.
- Time-limited activation windows and controlled rollout comparisons.
- A packaged evaluation SDK once its semantics stabilize.

## Proposed tech stack

C# / .NET 10, a dependency-light evaluation core, ASP.NET Core API, Blazor playground, SQLite for version history, xUnit and property-based tests.

Select compatible stable dependency versions during milestone 1, record them in project files, and pin the SDK/toolchain after the compatibility spike. No packages or development workloads are installed by this documentation task.

## Architecture

A parser converts JSON DTOs into a typed expression tree. Validation resolves field references against an input schema. A pure evaluator produces a decision and explanation using an explicitly supplied context and clock. Version storage and HTTP/UI adapters sit outside the evaluator; published versions cannot be edited in place.

## Proposed folder structure

Only README.md and .gitignore exist initially, alongside Git metadata. The following tree is a plan; create directories when their implementation begins.

```text
RuleWeaver/
  README.md
  .gitignore
  RuleWeaver.slnx
  src/RuleWeaver.Core/
  src/RuleWeaver.Application/
  src/RuleWeaver.Infrastructure/
  src/RuleWeaver.Api/
  src/RuleWeaver.Web/
  tests/RuleWeaver.UnitTests/
  tests/RuleWeaver.ContractTests/
  samples/rules/
  docs/language/
```

## Data models and invariants

RuleSet(Id, Version, InputSchemaVersion, Rules, DefaultOutcome); Rule(Id, Priority, Condition, Outcome); Condition(Field, Operator, TypedValue) or BooleanGroup; EvaluationResult(Version, Outcome, MatchedRuleIds, Explanation). Use decimal for explicit decimal fields and ordinal string comparisons by default. Define first-match behavior with stable priority/ID ordering.

## Proposed API and contracts

POST /api/rulesets/{id}/versions validates a draft; POST /api/rulesets/{id}/versions/{version}/publish makes it available; POST /api/evaluations accepts a version and facts; POST /api/comparisons evaluates fixtures against two versions. Unknown operators and unsupported types are validation errors, not false matches.

Version persisted/exported formats from their first implementation. Validate at boundaries and return structured errors rather than leaking internal exceptions. The contracts above remain proposals until backed by code and contract tests.

## Implementation milestones

1. Write language reference and golden decision/explanation fixtures before evaluator code.
2. Implement typed evaluation and resource limits; validate deterministic ordering.
3. Add immutable storage, evaluation API, and version comparison.
4. Build playground with sample routing/eligibility cases and publish a compatibility policy.

Each milestone should end with a small demonstrable result, a written acceptance check, and tests for its important failure modes. Work through a vertical slice before expanding the stretch scope.

## Testing and CI plan

Test boundary values, null/missing distinction, decimal precision, nesting limits, explanation stability, and repeated deterministic evaluation. Property tests check Boolean identities where the missing-value policy permits them. CI runs unit and API contract tests, schema validation, format checks, and packaged-core compatibility checks.

CI is planned, not configured. Once a solution exists, add .github/workflows/ci.yml with read-only default permissions, explicit toolchain setup, dependency restore, build, and the checks above. Keep ordinary CI offline from real services and hardware. Store test reports as artifacts; do not publish packages, create releases, or deploy from the first workflow. Record hardware/manual results separately from automated passes.

## Setup and development

### Current checkout

Open this README in a Markdown viewer. Git is initialized locally; no build or run step exists yet. Check the working tree with `git status --short --branch`. Documentation files start untracked; no initial commit or remote is created.

### Prerequisites and first implementation

After scaffolding, create the core first, add sample schemas and rule sets, then implement API and playground hosts. Use a local SQLite database for draft history and publish state. Validate every example through the same parser used by the API.

Install only the dependencies needed for the selected milestone. Add a checked-in sample configuration with harmless defaults; keep local databases, recordings, generated datasets, credentials, and device identifiers outside version control. Pick a license before public distribution; no license is implied by this plan.

### Future commands

These commands are examples for after the referenced projects/files have been created; they will not build this documentation-only repository.

```powershell
dotnet restore RuleWeaver.slnx
dotnet test RuleWeaver.slnx
dotnet run --project src/RuleWeaver.Api
```

## Supporting libraries and boundaries

ConfigLens may validate host settings; DataSynth produces input fixtures. Rule semantics stay local and should not depend on ConfigLens configuration-merge behavior.

Consume stable library versions through local package artifacts initially. Avoid absolute ProjectReference paths into sibling repositories and avoid Git submodules for the MVP. Keep a consumer's first vertical slice runnable using a small local interface or fixture while a supporting package is under development.

## Realistic limitations

MVP has no arbitrary C#, reflection, network calls, or user-supplied functions. A decision engine cannot establish whether rules are fair, lawful, or correct for a business; domain owners must review policy. Limit expression depth, rule count, and input size to bound evaluation work.

Scope and performance targets are hypotheses until measured. Document reproducible dataset sizes, hardware, dependency versions, and failure cases alongside any benchmark claim. Do not present planned capabilities as shipped features.

## Documentation provenance

Prepared 2026-10-23. The previous chat supplied the project names and the AutoDiag platform change, but its detailed assistant responses were unavailable content references. The user confirmed the seven proposed application themes in this session; individual designs here are newly proposed plans, not a recovered transcript.

