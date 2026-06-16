---
name: laravel-datepicker-installation
description: Use this skill when installing and setting up datepicker assets in a Laravel project.
---

# Laravel Datepicker Installation

Use this skill when the user wants to install or set up datepicker in a Laravel project.

## Goal

Install and configure datepicker assets so they can be used in Laravel from date input.

## When to Use

Use this skill when the user asks to:

- Install datepicker in Laravel
- Add datepicker CSS and JavaScript assets
- Configure datepicker assets manually
- Prepare Laravel form to use datepicker
- Set up select option use datepicker

## Assets

datepicker assets can be downloaded from:

https://github.com/taupikpirdian/assets/datepicker

Required files:

datepicker/js/bootstrap-datepicker.min.js
datepicker/css/bootstrap-datepicker.min.css

## Installation Steps
1. Download Assets

Download the required datepicker assets from the asset repository.

2. Place Assets in Laravel Public Directory

Place the files inside the Laravel public directory.

Recommended structure:

public/
└── assets/
    └── libs/
        └── datepicker/
            ├── js/
            │   └── bootstrap-datepicker.min.js
            └── css/
                └── bootstrap-datepicker.min.css

3. Register CSS Asset

Add the datepicker CSS file in the main layout file.

Example:

<link rel="stylesheet" href="{{ asset('assets/libs/datepicker/css/...') }}">

4. Register JavaScript Asset

Add the datepicker JavaScript file before the closing </body> tag.

Example:

<script src="{{ asset('assets/libs/datepicker/js/...') }}"></script>

5. How to use

Put this code on global js file
```js
$(function() {
    $(".datepicker").datepicker({
        format: "dd/mm/yyyy",
    });
});
```

and put on all class form input date