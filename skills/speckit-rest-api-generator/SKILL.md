---
name: speckit-rest-api-generator
description: Use this skill when the user asks to create a single REST API specification using Spec Kit / Speckit style. This includes Indonesian prompts such as "buatkan speckit REST API", "buatkan API speckit", "buatkan spesifikasi REST API", "buatkan dokumen API", or English prompts such as "generate REST API spec", "create REST API specification", and "make REST API feature spec". Clarify incomplete REST API requirements before creating files. The final output must be created under docs/speckit/<feature-name>/ and must describe only one requested API operation.
---

# Speckit REST API Generator

Use this skill when the user wants to create a Speckit specification for one REST API operation before implementation.

The output must be written under:

```text
docs/speckit/<feature-name>/
```

Use kebab-case for `<feature-name>`.

## Goal

Create concise specs for exactly one REST API operation so the implementation agent understands the URL, headers, payload, success response, error response, optional validation rules, and implementation tasks.

Do not turn one requested API into a full CRUD feature. Do not add List, Detail, Create, Update, Delete, pagination, search, filter, sort, or relationship endpoints unless the user explicitly includes that behavior in the single API contract.

## Workflow

Follow this order:

```text
clarify -> specify -> plan -> tasks
```

## 1. Clarify

Do not create directories or files during clarification.

Before generating files, verify that the API contract is clear enough to implement. Ask targeted clarification questions for missing or ambiguous requirements, then wait for the user's answer.

Required clarity:

* HTTP method.
* URL or endpoint path.
  * Example: `POST /api/job-vacancies/apply`
* Purpose of the API operation.
* Required request headers, or explicit confirmation that no custom headers are needed.
* Authentication or authorization expectations, or explicit confirmation that none are needed.
* Request payload, query parameters, and path parameters, or explicit confirmation that each unused category is not needed.
* Success response example or schema.
* Error response example or schema.
* Status codes.
* Custom validation rules, or explicit confirmation that no custom validation is needed.
* Used database tables, or explicit confirmation that database access is not needed.

If any required clarity item is missing, ask for only the missing information. Group related questions so the user can answer quickly, but do not generate `spec.md`, `plan.md`, or `tasks.md` yet.

Derive the feature name from the HTTP method and URL when the user does not provide one. Example: `POST /api/job-vacancies/apply` becomes `post-job-vacancies-apply`.

If the HTTP method is not included, ask for it. Do not infer the method from the payload.

Do not use `TBD` for missing requirements. Do not proceed with assumptions while any required clarity item is missing.

## 2. Output Files

Create these files:

```text
docs/speckit/<feature-name>/
  spec.md
  plan.md
  tasks.md
```

## 3. spec.md

Focus on what the single REST API operation should do and why.

Must include:

* Feature name
* Purpose
* Target consumer
* Scope
* Out of scope
* Single API operation
* Method
* URL
* Request headers
* Request path parameters
* Request query parameters
* Request payload
* Response fields
* Success response format
* Error response format
* Custom validation expectations
* Status codes
* Success criteria

For the single API operation, include:

* Method
* Path
* Purpose
* Headers
* Query parameters
* Path parameters
* Request payload
* Success response
* Error response
* Custom validation
* Status codes

Keep `spec.md` scoped to this one operation. Do not invent extra API behavior.

## 4. plan.md

Focus on how the single REST API operation will be built.

Must include:

* Tech stack
* Architecture pattern
* Single endpoint design
* Header handling
* Payload or parameter mapping
* Success response mapping
* Error response mapping
* Custom validation approach
* Error handling approach
* Authentication approach, only if identifiable from headers or user input
* Authorization/policy approach, only if provided by the user

Do not include implementation plans for endpoints that were not requested.

## 5. tasks.md

Break the plan into ordered implementation tasks.

Use checkbox format.

Example:

```md
# Tasks

- [ ] Define one API operation only
- [ ] Define method and URL
- [ ] Define request headers
- [ ] Define request payload, query parameters, and path parameters
- [ ] Define success response format
- [ ] Define error response format
- [ ] Define custom validation rules, if provided
- [ ] Map request data into the handler or service
- [ ] Map success response from the handler or service
- [ ] Map error responses and status codes
- [ ] Implement validation, only for provided validation rules
- [ ] Add tests for this single API operation
- [ ] Review speckit completeness against the provided input
```

## Agent Behavior

When applying this skill:

1. Ask required clarification questions first when any required clarity item is missing or ambiguous.
2. Wait for user answers before creating `docs/speckit/<feature-name>/`.
3. Do not create placeholder speckit files with `TBD` values.
4. Normalize feature name to kebab-case after the method and URL are known.
5. Generate only `spec.md`, `plan.md`, and `tasks.md`.
6. Save files under `docs/speckit/<feature-name>/`.
7. Keep each file concise, implementation-ready, and limited to one REST API operation.
8. After creating files, summarize created files and the confirmed API contract.
