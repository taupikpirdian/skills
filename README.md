# Claude Code Skills

This repository contains a collection of **Claude Code skills** used as working guidelines for building or improving Laravel and Go projects.

## Repository Contents

```text
skills/
├── crud-speckit-generator/
├── golang-clean-architecture/
├── laravel-datatable-implementation/
├── laravel-installation-ckeditor/
├── laravel-installation-datatable/
├── laravel-installation-datepicker/
├── laravel-installation-sweatalert/
├── laravel-service-repository/
├── laravel-validation/
├── salt-httpmanager/
└── salt-logmanager/
```

Each skill folder contains the main file:

```text
SKILL.md
```

## Installation

Make sure Node.js and npm are installed:

```bash
node -v
npm -v
```

Then add this skills repository using `npx`:

```bash
npx skills add https://github.com/taupikpirdian/skills
```

## Usage

After the skills are installed, ask Claude Code to use the relevant skill for your task.

Example prompts:

```text
Use the laravel-service-repository skill to create a Product CRUD feature.
```

```text
Use the laravel-validation skill to create a User Form Request validation.
```

```text
Use the golang-clean-architecture skill to structure this Go project.
```

```text
Use the crud-speckit-generator skill to create a CRUD specification for Customer.
```

## Available Skills

| Skill | Purpose |
| --- | --- |
| `crud-speckit-generator` | Creates spec, plan, and task documents for CRUD features. |
| `golang-clean-architecture` | Provides guidance for Go Clean Architecture structure. |
| `laravel-service-repository` | Guides Laravel Controller, Service, Repository, and Model structure. |
| `laravel-validation` | Guides Laravel Form Request validation. |
| `laravel-datatable-implementation` | Guides Laravel server-side DataTable implementation. |
| `laravel-installation-ckeditor` | Guides CKEditor installation in Laravel. |
| `laravel-installation-datatable` | Guides DataTable installation in Laravel. |
| `laravel-installation-datepicker` | Guides datepicker installation in Laravel. |
| `laravel-installation-sweatalert` | Guides SweetAlert installation in Laravel. |
| `salt-httpmanager` | Guides Go REST API development using `httpmanager`. |
| `salt-logmanager` | Guides Go logging and tracing using `logmanager`. |

## Adding a New Skill

Create a new folder inside `skills/`:

```bash
mkdir -p skills/skill-name
```

Create the `SKILL.md` file:

```bash
touch skills/skill-name/SKILL.md
```

Basic format:

```markdown
---
name: skill-name
description: Use this skill when ...
---

# Skill Name

Write the skill instructions here.
```

Use `kebab-case` for skill names to keep them consistent.
