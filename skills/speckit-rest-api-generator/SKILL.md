---
name: speckit-rest-api-generator
description: Use this skill when the user asks to create REST API specifications using Spec Kit / Speckit style. This includes Indonesian prompts such as "buatkan speckit REST API", "buatkan API speckit", "buatkan spesifikasi REST API", "buatkan dokumen API", or English prompts such as "generate REST API spec", "create REST API specification", and "make REST API feature spec". The output must be created under docs/speckit/<feature-name>/.
---

# Speckit REST API Generator

Use this skill when the user wants to create a REST API feature specification before implementation.

The output must be written under:

```text
docs/speckit/<feature-name>/
```

Use kebab-case for `<feature-name>`.

## Goal

Create concise REST API specs so the agent understands the resource, endpoints, request payload, response format, validation, authentication, authorization, error handling, and implementation tasks.

## Workflow

Follow this order:

```text
clarify → specify → plan → tasks
```

## 1. Clarify

Before generating files, ask the user:

- What is the REST API feature name?
- What resource or model or design table will this API use?
- What base endpoint should be used?
   * Example: `/api/job-vacancies`
- Which HTTP methods are needed?
   * GET
   * POST
   * PUT
   * PATCH
   * DELETE
- Are there any required request headers?
   * Example: `Authorization`, `Accept`, `Content-Type`, `api_key`, `x-signature`
- If the method is `POST`, what request fields are accepted for create?
- If the method is `PUT` or `PATCH`, what request fields are accepted for update?
- Which request fields are mandatory?
- What validation rules are needed?
- What response fields should be returned?
- Is authentication required?
- What authorization or permission rules are needed?
- If the method is `GET`, is pagination needed for the list endpoint?
- If the method is `GET`, is search needed?
- If the method is `GET`, are filters needed?
- If the method is `GET`, is sorting needed?
- What success response format should be used?
- What error response format should be used?

Do not generate the final speckit until these API details are clear:

If information is missing, use `TBD` instead of guessing.

## 2. Output Files

Create these files:

```text
docs/speckit/<feature-name>/
  spec.md
  plan.md
  tasks.md
```

## 3. spec.md

Focus on what the REST API should do and why.

Must include:

* Feature name
* Purpose
* Target consumer
* Scope
* Out of scope
* Related resource/model
* Base endpoint
* Authentication requirement
* REST actions
* Endpoint list
* Request query parameters
* Request body
* Required request fields
* Response fields
* Success response format
* Error response format
* Validation expectations
* Pagination expectations
* Search/filter/sort expectations
* Relationship response expectations
* Status codes
* Success criteria

For each endpoint, include:

* Method
* Path
* Purpose
* Auth requirement
* Query parameters
* Path parameters
* Request body
* Success response
* Error response
* Status codes

## 4. plan.md

Focus on how the REST API feature will be built.

Must include:

* Tech stack
* Architecture pattern
* Resource/model used
* Table used
* Endpoint design
* Request field mapping
* Response field mapping
* Request validation approach
* Response/resource approach
* Error handling approach
* Authentication approach
* Authorization/policy approach
* Pagination/search/filter/sort approach

## 5. tasks.md

Break the plan into ordered implementation tasks.

Use checkbox format.

Example:

```md
# Tasks

- [ ] Define REST API feature name
- [ ] Define resource, model, or design table used by the API
- [ ] Define base endpoint
- [ ] Define required HTTP methods
- [ ] Define required request headers if needed
- [ ] Define request fields for `POST` create endpoint if needed
- [ ] Define request fields for `PUT` or `PATCH` update endpoint if needed
- [ ] Define mandatory request fields
- [ ] Define validation rules for request fields
- [ ] Define response fields
- [ ] Define authentication requirement
- [ ] Define authorization or permission rules
- [ ] Define pagination behavior for `GET` list endpoint if needed
- [ ] Define search behavior for `GET` endpoint if needed
- [ ] Define filter behavior for `GET` endpoint if needed
- [ ] Define sorting behavior for `GET` endpoint if needed
- [ ] Define success response format
- [ ] Define error response format
- [ ] Review REST API specification completeness
```
