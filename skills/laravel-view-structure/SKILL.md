---
name: c
description: Use when creating, refactoring, or standardizing Laravel Blade folder structure under resources/views for admin panels, CRUD pages, modules, layouts, components, partials, and reusable view conventions across Laravel projects.
---

# Laravel View Structure

Gunakan skill ini ketika membuat atau merapikan struktur folder `resources/views` pada project Laravel. Tujuannya adalah membuat susunan Blade yang konsisten, mudah dicari, dan mudah dipakai ulang di project lain.

## Prinsip Utama

- Ikuti struktur view yang sudah ada di project jika project sudah punya pola jelas.
- Pakai folder berdasarkan domain atau modul, bukan berdasarkan tipe halaman saja.
- Simpan layout global di `layouts/`.
- Simpan komponen kecil yang reusable di `components/`.
- Simpan potongan Blade yang hanya dipakai dalam satu modul di folder `_partials/` pada modul tersebut.
- Gunakan nama file Blade yang jelas, kecil, dan konsisten.
- Jangan mencampur view admin, public, auth, dan email dalam satu folder datar.

## Struktur Default

Gunakan struktur berikut untuk project Laravel berbasis dashboard/admin dan CRUD:

```text
resources/views/
├── layouts/
│   ├── app.blade.php
│   ├── auth.blade.php
│   ├── guest.blade.php
│   └── partials/
│       ├── head.blade.php
│       ├── navbar.blade.php
│       ├── sidebar.blade.php
│       ├── footer.blade.php
│       └── scripts.blade.php
├── components/
│   ├── alert.blade.php
│   ├── breadcrumb.blade.php
│   ├── button.blade.php
│   ├── card.blade.php
│   ├── form-error.blade.php
│   ├── modal.blade.php
│   └── pagination.blade.php
├── auth/
│   ├── login.blade.php
│   ├── register.blade.php
│   ├── forgot-password.blade.php
│   └── reset-password.blade.php
├── dashboard/
│   └── index.blade.php
├── modules/
│   └── example-module/
│       ├── index.blade.php
│       ├── create.blade.php
│       ├── edit.blade.php
│       ├── show.blade.php
│       └── _partials/
│           ├── form.blade.php
│           ├── filter.blade.php
│           └── table.blade.php
├── errors/
│   ├── 403.blade.php
│   ├── 404.blade.php
│   └── 500.blade.php
└── emails/
    └── default.blade.php
```

## Struktur Modul CRUD

Untuk setiap fitur CRUD, buat folder di `resources/views/modules/<nama-modul>/`.

```text
resources/views/modules/alumni/
├── index.blade.php
├── create.blade.php
├── edit.blade.php
├── show.blade.php
└── _partials/
    ├── form.blade.php
    ├── filter.blade.php
    └── table.blade.php
```

Gunakan mapping berikut:

- `index.blade.php`: halaman daftar data.
- `create.blade.php`: halaman tambah data.
- `edit.blade.php`: halaman ubah data.
- `show.blade.php`: halaman detail data.
- `_partials/form.blade.php`: form yang dipakai bersama oleh create dan edit.
- `_partials/filter.blade.php`: filter pencarian untuk index.
- `_partials/table.blade.php`: tabel daftar data untuk index.

## Konvensi Penamaan

- Gunakan `kebab-case` untuk nama folder dan file.
- Gunakan nama domain bisnis untuk folder modul, misalnya `alumni`, `lowongan-kerja`, `tracer-study`, atau `pengumuman`.
- Gunakan `_partials` untuk partial lokal modul.
- Hindari nama terlalu umum seperti `data`, `page`, `view`, atau `content`.
- Hindari file Blade dengan nama campuran bahasa jika project sudah memilih satu bahasa.

## Konvensi Blade

Gunakan layout dengan pola berikut:

```blade
@extends('layouts.app')

@section('title', 'Alumni')

@section('content')
    {{-- Page content --}}
@endsection
```

Gunakan include partial modul dengan pola berikut:

```blade
@include('modules.alumni._partials.filter')
@include('modules.alumni._partials.table')
```

Untuk create dan edit, gunakan satu partial form:

```blade
@include('modules.alumni._partials.form', [
    'alumni' => $alumni ?? null,
])
```

## Checklist Saat Membuat View Baru

1. Identifikasi area view: `auth`, `dashboard`, `modules`, `emails`, atau `errors`.
2. Jika fitur berupa CRUD, buat folder di `modules/<nama-modul>/`.
3. Buat file utama sesuai action: `index`, `create`, `edit`, dan `show`.
4. Pindahkan bagian yang berulang ke `_partials/`.
5. Pakai `@extends('layouts.app')` untuk halaman dashboard/admin.
6. Pakai `@section('title', ...)` agar judul halaman konsisten.
7. Pastikan route/controller mengarah ke path view yang sama.
8. Jalankan pemeriksaan manual pada halaman terkait setelah struktur dibuat.

## Contoh Mapping Controller ke View

```php
return view('modules.alumni.index', compact('alumni'));
return view('modules.alumni.create');
return view('modules.alumni.edit', compact('alumni'));
return view('modules.alumni.show', compact('alumni'));
```

## Kapan Membuat Component

Buat file di `resources/views/components/` jika potongan UI:

- Dipakai oleh lebih dari satu modul.
- Tidak bergantung kuat pada satu domain.
- Cocok dipanggil sebagai komponen Blade.

Contoh:

```blade
<x-alert />
<x-breadcrumb :items="$breadcrumbs" />
<x-button type="submit">Simpan</x-button>
```

## Kapan Membuat Partial

Buat file di `_partials/` jika potongan UI:

- Hanya dipakai oleh satu modul.
- Bergantung pada data modul tersebut.
- Terlalu spesifik untuk dijadikan komponen global.

Contoh:

```blade
@include('modules.alumni._partials.form')
@include('modules.alumni._partials.table')
```

## Aturan Adaptasi Project

- Jika project memakai `admin/` sebagai root view, tempatkan modul di `resources/views/admin/modules/`.
- Jika project memakai tema seperti AdminLTE atau template custom, ikuti layout bawaan project dan hanya rapikan struktur folder modul.
- Jika project sudah memakai Laravel Blade anonymous components secara intensif, prioritaskan `components/` untuk UI reusable dan gunakan `_partials/` hanya untuk potongan domain.
- Jika project memiliki frontend public dan admin, pisahkan menjadi `public/` dan `admin/`.

Contoh struktur untuk pemisahan public dan admin:

```text
resources/views/
├── admin/
│   ├── layouts/
│   ├── dashboard/
│   └── modules/
└── public/
    ├── layouts/
    ├── home.blade.php
    └── pages/
```

## Output yang Diharapkan

Saat skill ini digunakan, hasilkan:

- Struktur folder Blade yang konsisten dengan project.
- File Blade utama untuk halaman yang diminta.
- Partial lokal untuk form, filter, table, atau bagian berulang lain.
- Penamaan view yang cocok dengan controller dan route Laravel.
- Catatan singkat jika ada penyesuaian dari struktur default karena mengikuti pola project.
