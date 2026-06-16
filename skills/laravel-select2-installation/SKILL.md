---
name: laravel-select2-installation
description: Use this skill when installing and setting up select2 assets in a Laravel project.
---

# Laravel Select2 Installation

Use this skill when the user wants to install or set up select2 in a Laravel project.

## Goal

Install and configure Select2 assets so they can be used in Laravel from select option.

## When to Use

Use this skill when the user asks to:

- Install Select2 in Laravel
- Add Select2 CSS and JavaScript assets
- Configure Select2 assets manually
- Prepare Laravel form to use Select2
- Set up select option use Select2

## Assets

Select2 assets can be downloaded from:

https://github.com/taupikpirdian/assets

Required files:

select2/js/select2.min.js
select2/css/select2.min.css

## Installation Steps
1. Download Assets

Download the required Select2 assets from the asset repository.

2. Place Assets in Laravel Public Directory

Place the files inside the Laravel public directory.

Recommended structure:

public/
└── assets/
    └── libs/
        └── select2/
            ├── js/
            │   └── select2.min.js
            └── css/
                └── select2.min.css

3. Register CSS Asset

Add the select2 CSS file in the main layout file.

Example:

<link rel="stylesheet" href="{{ asset('assets/libs/select2/css/...') }}">

4. Register JavaScript Asset

Add the select2 JavaScript file before the closing </body> tag.

Example:

<script src="{{ asset('assets/libs/select2/js/...') }}"></script>

5. How to use

Put this code on global js file
```js
$(".select2-global").select2();
```

and put on all class form select option