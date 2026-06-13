---
name: laravel-sweatalert-installation
description: Use this skill when installing and setting up SweetAlert assets in a Laravel project.
---

# Laravel SweetAlert Installation

Use this skill when the user wants to install or set up SweetAlert in a Laravel project.

## Goal

Install and configure SweetAlert assets so they can be used in Laravel pages, especially for list/table views.

## When to Use

Use this skill when the user asks to:

- Install SweetAlert in Laravel
- Add SweetAlert CSS and JavaScript assets
- Configure SweetAlert assets manually

## Assets

SweetAlert assets can be downloaded from:

https://github.com/taupikpirdian/assets

Required files:

sweatalert/css/sweetalert2.min.css
sweatalert/js/sweetalert2.min.js

## Installation Steps
1. Download Assets

Download the required SweetAlert assets from the asset repository.

2. Place Assets in Laravel Public Directory

Place the files inside the Laravel public directory.

Recommended structure:

public/
└── assets/
    └── libs/
        └── sweatalert/
            ├── js/
            │   └── sweetalert2.min.js
            └── css/
                └── sweetalert2.min.css

3. Register CSS Asset

Add the SweetAlert CSS file in the main layout file.

Example:

<link rel="stylesheet" href="{{ asset('assets/libs/sweatalert/css/sweetalert2.min.css') }}">

4. Register JavaScript Asset

Add the SweetAlert JavaScript file before the closing </body> tag.

Example:

<script src="{{ asset('assets/libs/sweatalert/js/sweetalert2.min.js') }}"></script>

5. Setup SweetAlert

Add SweetAlert usage after the JavaScript asset is loaded.

Example for Laravel flash messages:

```blade
@if (session('success'))
    <script>
        Swal.fire({
            icon: 'success',
            title: 'Berhasil',
            text: @json(session('success')),
            timer: 2000,
            showConfirmButton: false
        });
    </script>
@endif

@if (session('error'))
    <script>
        Swal.fire({
            icon: 'error',
            title: 'Gagal',
            text: @json(session('error'))
        });
    </script>
@endif
```

Example controller redirect:

```php
return redirect()
    ->route('users.index')
    ->with('success', 'Data berhasil disimpan.');
```

Example delete confirmation for table/list actions:

```blade
<form action="{{ route('users.destroy', $user->id) }}" method="POST" class="delete-form">
    @csrf
    @method('DELETE')
    <button type="submit">Hapus</button>
</form>

<script>
    document.querySelectorAll('.delete-form').forEach((form) => {
        form.addEventListener('submit', function (event) {
            event.preventDefault();

            Swal.fire({
                title: 'Hapus data?',
                text: 'Data yang sudah dihapus tidak dapat dikembalikan.',
                icon: 'warning',
                showCancelButton: true,
                confirmButtonText: 'Ya, hapus',
                cancelButtonText: 'Batal'
            }).then((result) => {
                if (result.isConfirmed) {
                    form.submit();
                }
            });
        });
    });
</script>
```
