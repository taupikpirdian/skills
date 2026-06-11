---

name: laravel-validation
description: Use this skill when creating or refactoring Laravel Form Request validation for CRUD/API features, including rules, authorization, custom messages, request preparation, and validated data usage.
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Laravel Validation Request

Use this skill to keep Laravel request validation clean, reusable, and separated from controllers and services.

## Purpose

Form Request is responsible for validating and authorizing incoming HTTP input before it reaches the controller or service layer.

## Core Rules

* Use Form Request for endpoint validation.
* Do not put long validation rules directly in Controller.
* Controller should receive validated input from Form Request.
* Service should receive clean data, not raw request objects.
* Business rules belong in Service, not Form Request.
* Database persistence must not happen inside Form Request.

### rules

Put validation rules in `rules()`.

Prefer array syntax for complex rules.

```php
public function rules(): array
{
    return [
        'name' => ['required', 'string', 'max:100'],
        'email' => ['required', 'email', 'max:150'],
        'phone' => ['nullable', 'string', 'max:20'],
    ];
}
```

### messages

Use `messages()` only when default messages are not clear enough.

```php
public function messages(): array
{
    return [
        'email.required' => 'Email is required.',
        'email.email' => 'Email format is invalid.',
    ];
}
```

## Controller Usage

Controller should use `$request->validated()` or `$request->safe()`.

```php
public function store(StoreUserRequest $request)
{
    $result = $this->userService->create($request->validated());

    return new UserResource($result);
}
```

Avoid passing raw request to Service.

Bad:

```php
$this->userService->create($request);
```

Good:

```php
$this->userService->create($request->validated());
```
