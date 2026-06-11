---

name: crud-speckit-generator
description: Use this skill when creating CRUD feature specifications using a Spec Kit style workflow under docs/speckit/<feature-name>/.
-----------------------------------------------------------------------------------------------------------------------------------------

# CRUD Speckit Generator

Use this skill when the user wants to create a CRUD feature specification before implementation.

The output must be written under:

```text
docs/speckit/<feature-name>/
```

Use kebab-case for `<feature-name>`.

## Goal

Create concise CRUD specs so the agent understands the feature, model, fields, validation, form behavior, and implementation tasks.

## Workflow

Follow this order:

```text
clarify → specify → plan → tasks
```

## 1. Clarify

Before generating files, ask the user:

1. What is the feature name?
2. Which model will be used?
3. Which fields from the model are shown in the list page?
4. Which fields from the model are mandatory?
5. What form/input type should each field use?
6. Which CRUD actions are needed: create, read/detail, update, delete, list?
7. Are there relationships with other models?
8. Are there special validation rules?
9. Are there role/permission rules?

Do not generate the final speckit until the model, mandatory fields, and form/input types are clear.

When asking about form/input types, suggest examples:

* text
* textarea
* number
* email
* password
* date
* datetime
* select
* radio
* checkbox
* file
* image
* hidden
* switch
* rich text editor

## 2. Output Files

Create these files:

```text
docs/speckit/<feature-name>/
  spec.md
  plan.md
  tasks.md
```

## 3. spec.md

Focus on what the CRUD feature should do and why.

Must include:

* Feature name
* Purpose
* Target user
* Scope
* Out of scope
* Related model
* CRUD actions
* List page fields
* Form fields
* Required fields
* Form/input type per field
* Relationships
* Validation expectations
* Permission expectations
* Success criteria

Avoid API endpoint details in `spec.md`.

## 4. plan.md

Focus on how the CRUD feature will be built.

Must include:

* Tech stack
* Architecture pattern
* Model/table used
* Field mapping
* List page behavior
* Form behavior
* Validation approach
* Controller plan
* Service plan
* Repository plan
* Resource/View plan
* Permission/policy plan if needed

For Laravel projects, include:

* Model
* Migration if needed
* Form Request
* Controller
* Service
* Repository
* View or Resource depending on project style
* Route
* Policy/Permission if needed

Avoid API endpoint details unless the user explicitly requests API CRUD.

## 5. tasks.md

Break the plan into ordered implementation tasks.

Use checkbox format.

Example:

```md
# Tasks

- [ ] Review existing model and table
- [ ] Create or update Form Request
- [ ] Create or update Controller actions
- [ ] Create or update Service methods
- [ ] Create or update Repository methods
- [ ] Create or update View/Resource
- [ ] Register routes
- [ ] Add permission/policy if needed
- [ ] Add tests if needed
```

## Rules

* Focus only on CRUD feature behavior.
* Do not include API endpoint specs unless requested.
* Keep files concise and implementation-ready.
* Do not assume mandatory fields without user confirmation.
* Do not assume form/input types without user confirmation.
* Use `TBD` for missing information instead of guessing.
* Use tables for field mapping when helpful.
* Use checklists for tasks.
* Do not generate unrelated features.
* Follow existing project conventions if available.

## Agent Behavior

When applying this skill:

1. Ask required clarification questions first.
2. Wait for user answers before generating files.
3. Normalize feature name to kebab-case.
4. Generate only `spec.md`, `plan.md`, and `tasks.md`.
5. Save files under `docs/speckit/<feature-name>/`.
6. Keep each file concise but complete.
7. Explain assumptions clearly if the user allows assumptions.
8. After creating files, summarize created files and suggested next step.

## UI Component Defaults

When generating CRUD specs, use these default UI components unless the user says otherwise:

* List data must use DataTable.
* Option/select fields must use Select2.
* Date fields must use Datepicker.
* Long text or rich text fields must use CKEditor.

Apply these defaults in `spec.md`, `plan.md`, and `tasks.md` when relevant.

## Field Component Mapping

Use this mapping when defining form fields:

| Field Type                       | Default Component            |
| -------------------------------- | ---------------------------- |
| list/table                       | DataTable                    |
| select / option / relationship   | Select2                      |
| date                             | Datepicker                   |
| datetime                         | Datepicker or DateTimePicker |
| textarea / long text / rich text | CKEditor                     |
| text / short string              | input text                   |
| number                           | input number                 |
| email                            | input email                  |
| password                         | input password               |
| file / image                     | file upload                  |
| boolean                          | switch or checkbox           |

If the field type is unclear, mark it as `TBD` and ask the user.
