---
name: laravel-datatable-installation
description: Use this skill when installing and setting up DataTables assets in a Laravel project.
---

# Laravel DataTable Installation

Use this skill when the user wants to install or set up DataTables in a Laravel project.

## Goal

Install and configure DataTables assets so they can be used in Laravel pages, especially for list/table views.

## When to Use

Use this skill when the user asks to:

- Install DataTables in Laravel
- Add DataTables CSS and JavaScript assets
- Configure DataTables assets manually
- Prepare Laravel views to use DataTables
- Set up list pages using DataTables

## Assets

DataTables assets can be downloaded from:

https://github.com/taupikpirdian/assets

Required files:

datatables/js/jquery.datatables.min.js
datatables/css/bootstrap.datatable.min.css

## Installation Steps
1. Download Assets

Download the required DataTables assets from the asset repository.

2. Place Assets in Laravel Public Directory

Place the files inside the Laravel public directory.

Recommended structure:

public/
└── assets/
    └── libs/
        └── datatables/
            ├── js/
            │   └── jquery.datatables.min.js
            └── css/
                └── bootstrap.datatable.min.css

3. Register CSS Asset

Add the DataTables CSS file in the main layout file.

Example:

<link rel="stylesheet" href="{{ asset('assets/libs/datatables/css/bootstrap.datatable.min.css') }}">

4. Register JavaScript Asset

Add the DataTables JavaScript file before the closing </body> tag.

Example:

<script src="{{ asset('assets/libs/datatables/js/jquery.datatables.min.js') }}"></script>