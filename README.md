# Claude Code Skills

Repository ini berisi kumpulan **skills untuk Claude Code** yang digunakan sebagai panduan kerja saat membuat atau merapikan project Laravel dan Go.

## Isi Repository

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

Setiap folder skill memiliki file utama:

```text
SKILL.md
```

## Cara Menggunakan

Jalankan Claude Code menggunakan `npx` dari root repository ini:

```bash
npx skills add https://github.com/taupikpirdian/skills
```

Lalu minta Claude menggunakan skill yang sesuai dengan kebutuhan.

Contoh prompt:

```text
Gunakan skill laravel-service-repository untuk membuat fitur CRUD Product.
```

```text
Gunakan skill laravel-validation untuk membuat Form Request validasi user.
```

```text
Gunakan skill golang-clean-architecture untuk menyusun struktur project Go ini.
```

```text
Gunakan skill crud-speckit-generator untuk membuat spesifikasi CRUD Customer.
```

## Daftar Skill Singkat

| Skill | Kegunaan |
| --- | --- |
| `crud-speckit-generator` | Membuat dokumen spec, plan, dan tasks untuk fitur CRUD. |
| `golang-clean-architecture` | Panduan struktur Go Clean Architecture. |
| `laravel-service-repository` | Panduan pola Controller, Service, Repository, Model di Laravel. |
| `laravel-validation` | Panduan membuat Laravel Form Request validation. |
| `laravel-datatable-implementation` | Panduan implementasi DataTable server-side di Laravel. |
| `laravel-installation-ckeditor` | Panduan instalasi CKEditor di Laravel. |
| `laravel-installation-datatable` | Panduan instalasi DataTable di Laravel. |
| `laravel-installation-datepicker` | Panduan instalasi datepicker di Laravel. |
| `laravel-installation-sweatalert` | Panduan instalasi SweetAlert di Laravel. |
| `salt-httpmanager` | Panduan membuat REST API Go dengan `httpmanager`. |
| `salt-logmanager` | Panduan logging/tracing Go dengan `logmanager`. |

## Menambahkan Skill Baru

Buat folder baru di dalam `skills/`:

```bash
mkdir -p skills/nama-skill
```

Buat file `SKILL.md`:

```bash
touch skills/nama-skill/SKILL.md
```

Format dasar:

```markdown
---
name: nama-skill
description: Use this skill when ...
---

# Nama Skill

Isi instruksi skill di sini.
```

Gunakan nama skill dengan format `kebab-case` agar konsisten.
