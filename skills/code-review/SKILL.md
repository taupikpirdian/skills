---
name: code-review
description: Review current code changes, GitLab merge requests, and code diffs with a security-first, issue-only stance. Use when Codex is asked to review local git changes, an MR URL, GitLab merge request, code diff, or code changes against OWASP, Clean Architecture, DDD domain/application layer rules, Go code standards, and testing requirements; focuses findings on the changed code and includes GitLab MCP workflow, severity classification, summary comments, and inline finding guidance.
---

# GitLab Code Review

## Review Stance

Act as a code reviewer. Report only problems, bugs, risks, missing tests, and concrete improvements. Do not include compliments, positive feedback, or "looks good" comments.

Default to reviewing the code changes in front of you. For local repository reviews, inspect the current Git working tree and staged changes first; for MR reviews, inspect the MR diff first; for pasted diffs, review only the pasted diff. Read unchanged files only as supporting context needed to understand the changed code.

Tie every finding to changed code. Do not report pre-existing issues in untouched code unless the current change directly exposes, worsens, or depends on that issue.

Prioritize findings in this order:

1. Security vulnerabilities
2. Architectural or dependency-rule violations
3. Business logic in the wrong layer
4. DDD modeling and encapsulation issues
5. Correctness bugs and error handling problems
6. Missing or weak tests
7. Minor naming or code quality issues

Ignore these unless they hide a real bug:

- Missing newline at end of file
- Pure formatting or style-only issues
- Functions or methods called but not defined in the diff; assume they exist unless surrounding context proves otherwise

## GitLab MCP Workflow

When reviewing a GitLab MR with MCP tools available:

1. Always call `get_merge_request_summary(mr_url)` first.
2. Use the summary to check file count, changed file list, and MR size.
3. For small MRs under 10 files, call `get_merge_request_changes(mr_url)`.
4. For MRs with 10 or more files, fetch diffs by focus area with `file_patterns`, such as `domain/`, `application/`, `internal/usecase/`, `*.go`, or `*_test.go`.
5. For very large MRs, paginate with `start_index` and `end_index`.
6. Review by layer and risk, not merely by file order.
7. Post or return the review only after checking the relevant diffs and any needed surrounding context.

Use these MCP tools when available:

- `get_merge_request_summary`: MR overview, file count, statistics, and file list. This is mandatory before fetching changes.
- `get_merge_request_files`: changed files with status; use only when detailed file status is needed beyond the summary.
- `get_merge_request_changes`: diffs, with filtering and pagination.
- `get_file_content`: full file content from source branch, target branch, or commit when context is needed.
- `create_merge_request_note`: main MR comments and inline review comments.

If MCP tools are not available, review the provided diff or local branch changes directly and clearly state the scope reviewed.

## Local Changes Workflow

When reviewing the current repository instead of a GitLab MR:

1. Run `git status --short` to identify changed, staged, untracked, and deleted files.
2. Run `git diff --staged` when staged changes exist.
3. Run `git diff` for unstaged tracked changes.
4. Inspect untracked files directly when they are part of the requested review.
5. If a base branch is obvious from context, use `git diff <base>...HEAD` only to supplement the working-tree diff; do not let it replace current staged or unstaged changes.
6. Read surrounding unchanged code only when needed to validate behavior, architecture boundaries, or tests.
7. State the exact review scope, including whether staged, unstaged, untracked, or branch diff changes were reviewed.

## Posting Strategy

When the user asks to post comments to GitLab:

- Post the overall summary as a main MR comment with `create_merge_request_note` without `file_path`.
- Post each actionable finding as a separate inline comment with `file_path` and the correct `new_line` or `old_line`.
- Keep inline comments concise: issue, impact, and recommended fix.
- Do not post duplicate comments for the same root cause; group related line-level evidence in the summary when needed.

When responding in chat:

- Lead with findings, ordered by severity.
- Include file and line references whenever available.
- Add open questions or assumptions only after findings.
- Add a short summary only after findings.
- If no issues are found, say that directly and mention any remaining review scope or test gaps.

Suggested finding format:

```text
[Severity] file.go:123 - Short finding title
Impact: What can go wrong.
Recommendation: Specific fix or design change.
```

## Severity

Use these severity levels consistently:

| Severity | Use For |
| --- | --- |
| Critical | Security vulnerabilities, OWASP issues, dependency rule violations, business logic placed in the wrong architecture layer |
| Major | DDD violations, missing encapsulation, wrong domain building block, missing constructor pattern, correctness bugs with meaningful user or data impact |
| Minor | Typos, mixed concerns, missing or weak test coverage, local maintainability issues without direct correctness impact |

## Security Checklist

Security is the highest-priority review area. Check OWASP Top 10 risks:

- A01 Broken Access Control: missing authorization, role validation, resource ownership, tenant boundary checks.
- A02 Cryptographic Failures: sensitive data exposure, weak crypto, hardcoded secrets, insecure token handling.
- A03 Injection: SQL, NoSQL, OS command, LDAP, template, or path injection.
- A04 Insecure Design: missing security control, unsafe trust boundary, flawed abuse-case handling.
- A05 Security Misconfiguration: default credentials, debug features, permissive CORS, unsafe headers.
- A06 Vulnerable Components: outdated or vulnerable dependencies when dependency changes are in scope.
- A07 Authentication Failures: weak credential/session flow, missing expiry, session fixation risks.
- A08 Data Integrity Failures: insecure deserialization, unsigned data, unverified callbacks or payloads.
- A09 Logging Failures: missing audit logs for security-sensitive operations, or sensitive data in logs.
- A10 SSRF: unvalidated URLs, internal network access, unsafe redirects, metadata-service access.

Flag sensitive data in logs, error messages, metrics, traces, or API responses as security findings.

## Architecture Rules

Enforce SOLID:

- Single Responsibility: each struct or interface has one reason to change.
- Open/Closed: extension should not require risky modification of stable code.
- Liskov Substitution: implementations must satisfy interface expectations.
- Interface Segregation: prefer small focused interfaces.
- Dependency Inversion: depend on abstractions, not concrete infrastructure.

Enforce Clean Architecture dependencies:

```text
Delivery -> Application -> Domain <- Infrastructure
```

Layer responsibilities:

- Domain: entities, value objects, repository interfaces, domain services, events, enums. It must have no outer-layer dependencies.
- Application: use cases, DTOs, orchestration, transactions, mapping. It depends on domain abstractions.
- Infrastructure: repository implementations, database, external services, clients. It implements domain/application interfaces.
- Delivery: handlers, routes, middleware, request/response mapping. It depends on application.

Critical architecture rules:

- Inner layers must not know outer layers.
- Dependencies must point inward.
- Domain must have zero external dependencies.
- Application must not import `infrastructure/` packages directly.
- Infrastructure should implement domain interfaces and be wired by dependency injection.

## DDD Domain Layer Rules

Review for rich domain models:

- Business logic must live in domain entities, value objects, or justified domain services.
- Application services/usecases must not contain business validation, calculations, or state-transition rules.
- Avoid anemic entities that only hold data while services manipulate their fields.

Entity construction:

- Use `New<Entity>()` for creating new entities with full business validation.
- Use `Reconstruct<Entity>()` for loading trusted persisted data without creation validation.
- Flag empty struct plus `Rehydrate`/`Populate`, a single constructor with optional ID, or mixed creation/reconstitution logic.

Ubiquitous language:

- Domain methods should express business operations, not generic setters.
- Flag names such as `ChangeStatus(string)`, `SetActive(bool)`, `UpdateField(value)`, or `ChangeXxxID(id)`.
- Prefer business verbs such as `Activate()`, `Suspend()`, `Archive()`, `AssignTo...()`, or `MoveTo...()`.

Domain services:

- First ask whether the logic belongs in an entity.
- If not, ask whether it belongs in a value object.
- Use a domain service only for cross-aggregate coordination, stateless domain algorithms, domain-defined external interfaces, complex aggregate factories, or policy/strategy rules.
- Flag services that only delegate to one entity, services for single-entity behavior, generic `<Entity>Service` names, or services that exist because entities are anemic.

Building blocks:

- Use value objects for concepts defined by attributes and equality, such as email, money, address, quantity, percentage, date range, or URL.
- Use entities for objects with identity and lifecycle.
- Use aggregate roots as transactional consistency boundaries.
- Only aggregate roots should have repositories.
- Aggregates should reference other aggregates by ID, not direct object reference.

Encapsulation:

- Entity fields must be unexported.
- Read access should go through getters.
- State changes must go through domain methods that protect invariants.
- Flag plain data structs in `domain/entity/`; these are DTOs unless they own behavior.

Repository placement:

- Repository interfaces belong in `domain/repository/`.
- Repository implementations belong in `infrastructure/`.

## Application Layer Rules

Application usecases/services must be thin orchestrators:

1. Fetch entities from repositories.
2. Delegate validation to entity methods.
3. Delegate business rules to entities or domain services.
4. Persist changes through repositories.
5. Map domain objects to response DTOs.

Allowed in application:

- Workflow coordination
- Transaction or unit-of-work handling
- Repository calls
- Domain method and domain service invocation
- DTO mapping
- Cross-cutting concerns such as logging or metrics, without leaking sensitive data

Not allowed in application:

- Business validation logic
- Price, discount, or domain calculations
- Status transition rules
- Complex business `if/else` chains
- Direct entity field manipulation
- Direct imports from concrete infrastructure packages

Usecase constructor pattern:

- Use unexported structs.
- Provide `New<Usecase>()` constructor functions.
- Return the usecase interface when that is the project convention.
- Keep dependencies unexported.
- Flag exported dependency fields and direct struct initialization that bypasses the constructor.

## Go Code Standards

Naming:

- Packages: lowercase, single word when practical.
- Interfaces: descriptive nouns.
- Structs: PascalCase.
- Exported methods: PascalCase, imperative or present tense.
- Private methods: camelCase.
- Files: snake_case.
- Usecases: `<action><noun>Usecase`, such as `createOrderUsecase`.

Method and service names:

- Use imperative/present tense: `Send`, `Create`, `Validate`, not `Sent`, `Created`.
- Use ubiquitous language for domain methods.
- Service names must reflect actual responsibility.
- Avoid generic names such as `<Entity>Service`; prefer names like `PricingCalculator` or `OTPSender`.
- Split services that mix validation, sending, fetching, and unrelated responsibilities.

Error handling:

- Use domain-specific errors with `errors.New()`.
- Wrap errors with context using `fmt.Errorf("context: %w", err)`.
- Never swallow errors silently.
- Return errors; do not panic for expected failures.

Interface design:

- Prefer small interfaces with 1-3 methods.
- Keep each interface single-purpose.
- Compose small interfaces when needed.
- Define interfaces where they are consumed.
- Flag large catch-all interfaces such as repositories or services with unrelated methods.

## Testing Rules

Required unit test layers:

- `domain/`: required.
- `application/`: required.
- `delivery/`: not required as unit tests.
- `infrastructure/`: not required as unit tests.

Testing expectations:

- Use dependency injection through interfaces.
- Use mocks for external dependencies; mockery is acceptable when used by the project.
- Prefer table-driven tests for multiple scenarios.
- Name tests as `Test<Function>_<Scenario>` or `Test<Struct>_<Method>`.
- Prefer `_test` package suffix, such as `package usecase_test`.
- Use Arrange-Act-Assert structure.

Test domain behavior:

- Entity creation and validation.
- Entity state transitions.
- Value object equality and validation.
- Domain service calculations.
- Aggregate invariants.

Test application behavior:

- Usecase happy path.
- Error paths.
- DTO mapping.
- Workflow coordination.

Do not require unit tests for:

- Private implementation details.
- Third-party library internals.
- Database queries; use integration tests when needed.
- HTTP routing; use integration tests when needed.

Flag tests that access private methods through type assertions. Tests should validate behavior through public interfaces, not internal implementation details.

## Review Checklist

Before finalizing, check:

- Security and OWASP risks first.
- Authorization and ownership checks for protected operations.
- No sensitive data in logs or responses.
- Domain has no outer-layer dependencies.
- Application does not import infrastructure directly.
- Business logic is in domain, not usecases.
- Entities are encapsulated with unexported fields.
- Entity construction separates `New<Entity>()` and `Reconstruct<Entity>()`.
- Domain methods use business language rather than setters.
- Value objects are used for meaningful domain concepts.
- Domain services are justified.
- Repository interfaces live in domain and implementations in infrastructure.
- Usecases follow Fetch -> Delegate -> Persist -> Map.
- Usecases use constructor pattern and unexported dependencies.
- Errors are returned, wrapped, and not silently swallowed.
- Interfaces are small and consumer-focused.
- Domain and application changes have meaningful tests.

## Summary Comment Template

Use this when posting or returning an overall MR summary:

```markdown
## Code Review Summary

| Severity | Count |
| --- | ---: |
| Critical | 0 |
| Major | 0 |
| Minor | 0 |

### Findings

1. [Critical] `path/file.go:123` - Finding title.
2. [Major] `path/file.go:456` - Finding title.

### Notes

- Review scope: files/layers reviewed.
- Remaining risk: any files or behavior not reviewed.
```

Omit empty sections when unnecessary. Do not include positive feedback.

## Inline Comment Template

Use this for one inline finding:

```markdown
[Severity] Short finding title.

Impact: state the concrete risk.
Recommendation: state the specific change needed.
```
