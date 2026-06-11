---

name: laravel-datatable-implementation
description: Use this skill when implementing Laravel CRUD list pages using DataTable with Controller, Service, Repository, Blade, AJAX, filters, action buttons, and server-side processing.
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Laravel DataTable Implementation

Use this skill when implementing Laravel list pages with DataTable.

## Goal

Implement DataTable consistently using this flow:

```text
Blade → AJAX Route → Controller → Service → Repository → JSON Response
```

## Required Files

For a CRUD feature, create or update:

```text
routes/web.php
app/Http/Controllers/<Feature>Controller.php
app/Services/<Feature>Service.php
app/Repositories/<Feature>Repository.php
resources/views/<feature>/index.blade.php
```

If the project uses repository interfaces, also create:

```text
app/Repositories/Contracts/<Feature>RepositoryInterface.php
```

## Route Pattern

Use separate routes for page view and DataTable AJAX.

```php
Route::get('/products', [ProductController::class, 'index'])->name('products.index');
Route::get('/products/datatable', [ProductController::class, 'datatable'])->name('products.datatable');
```

## Controller Pattern

Controller should only render view or return DataTable response.

```php
public function index()
{
    return view('products.index');
}

public function datatable(Request $request)
{
    return response()->json(
        $this->productService->datatable($request->all())
    );
}
```

## Service Pattern

Service handles DataTable request mapping.

```php
public function datatable(array $params): array
{
    $start = (int) ($params['start'] ?? 0);
    $length = (int) ($params['length'] ?? 10);
    $search = $params['search']['value'] ?? null;
    $orderColumnIndex = $params['order'][0]['column'] ?? 0;
    $orderDirection = $params['order'][0]['dir'] ?? 'desc';

    $columns = [
        0 => 'id',
        1 => 'name',
        2 => 'status',
        3 => 'created_at',
    ];

    $orderColumn = $columns[$orderColumnIndex] ?? 'created_at';

    $result = $this->productRepository->datatable([
        'start' => $start,
        'length' => $length,
        'search' => $search,
        'order_column' => $orderColumn,
        'order_direction' => $orderDirection,
        'filters' => $params['filters'] ?? [],
    ]);

    return [
        'draw' => (int) ($params['draw'] ?? 1),
        'recordsTotal' => $result['total'],
        'recordsFiltered' => $result['filtered'],
        'data' => $result['data'],
    ];
}
```

## Repository Pattern

Repository handles query, search, filter, sorting, and pagination.

```php
public function datatable(array $params): array
{
    $baseQuery = Product::query()
        ->select([
            'id',
            'name',
            'status',
            'created_at',
        ]);

    $total = (clone $baseQuery)->count();

    if (!empty($params['search'])) {
        $baseQuery->where(function ($query) use ($params) {
            $search = $params['search'];

            $query->where('name', 'like', "%{$search}%")
                ->orWhere('status', 'like', "%{$search}%");
        });
    }

    if (!empty($params['filters']['status'])) {
        $baseQuery->where('status', $params['filters']['status']);
    }

    $filtered = (clone $baseQuery)->count();

    $rows = $baseQuery
        ->orderBy($params['order_column'], $params['order_direction'])
        ->skip($params['start'])
        ->take($params['length'])
        ->get();

    return [
        'total' => $total,
        'filtered' => $filtered,
        'data' => $rows->map(function ($row) {
            return [
                'id' => $row->id,
                'name' => e($row->name),
                'status' => e($row->status),
                'created_at' => optional($row->created_at)->format('Y-m-d H:i:s'),
                'action' => view('products.partials.action', [
                    'product' => $row,
                ])->render(),
            ];
        })->toArray(),
    ];
}
```

## Blade Pattern

Blade should render table, filters, and DataTable initialization.

```blade
<table id="product-table" class="table table-bordered table-striped">
    <thead>
        <tr>
            <th>ID</th>
            <th>Name</th>
            <th>Status</th>
            <th>Created At</th>
            <th width="120">Action</th>
        </tr>
    </thead>
</table>

<script>
$(function () {
    $('#product-table').DataTable({
        processing: true,
        serverSide: true,
        ajax: {
            url: '{{ route('products.datatable') }}',
            data: function (d) {
                d.filters = {
                    status: $('#filter-status').val()
                };
            }
        },
        columns: [
            { data: 'id', name: 'id' },
            { data: 'name', name: 'name' },
            { data: 'status', name: 'status' },
            { data: 'created_at', name: 'created_at' },
            { data: 'action', name: 'action', orderable: false, searchable: false }
        ]
    });
});
</script>
```

## Filter Pattern

When filters exist, reload DataTable on filter change.

```blade
<select id="filter-status" class="form-control">
    <option value="">All Status</option>
    <option value="active">Active</option>
    <option value="inactive">Inactive</option>
</select>

<script>
$('#filter-status').on('change', function () {
    $('#product-table').DataTable().ajax.reload();
});
</script>
```

## Action Button Pattern

Action buttons must be handled in backend.

The DataTable response should include an `action` column that contains rendered HTML buttons.

Recommended location:

```text
resources/views/<feature>/partials/action.blade.php
```

Repository or Service may render the action partial depending on existing project convention.

Preferred:

* Repository handles query and raw row data.
* Service formats DataTable rows and renders action buttons.
* Blade DataTable only displays the `action` column.

Example Service formatting:

```php
$data = $rows->map(function ($row) {
    return [
        'id' => $row->id,
        'name' => e($row->name),
        'status' => e($row->status),
        'created_at' => optional($row->created_at)->format('Y-m-d H:i:s'),
        'action' => view('products.partials.action', [
            'product' => $row,
        ])->render(),
    ];
})->toArray();
```

Example action partial:

```blade
<a href="{{ route('products.show', $product->id) }}" class="btn btn-sm btn-info">
    Detail
</a>

<a href="{{ route('products.edit', $product->id) }}" class="btn btn-sm btn-warning">
    Edit
</a>

<form action="{{ route('products.destroy', $product->id) }}" method="POST" class="d-inline delete-form">
    @csrf
    @method('DELETE')
    <button type="submit" class="btn btn-sm btn-danger">
        Delete
    </button>
</form>
```

DataTable column config:

```js
{
    data: 'action',
    name: 'action',
    orderable: false,
    searchable: false
}
```

Rules:

* Do not build action buttons in frontend JavaScript.
* Do not hardcode URLs in JavaScript.
* Use named routes in backend.
* Apply permission checks before rendering each button.
* Keep action button HTML in a partial Blade file.
* Mark `action` column as not searchable and not orderable.

## Implementation Rules

* Use server-side DataTable.
* Use separate route for DataTable AJAX.
* Keep Controller thin.
* Put DataTable parameter mapping in Service.
* Put query logic in Repository.
* Use partial Blade for action buttons.
* Escape displayed values with `e()`.
* Do not put query logic in Blade.
* Do not put large HTML action buttons in Repository unless project convention requires it.
* Do not hardcode URLs; use route names.
* Validate allowed order columns to avoid unsafe ordering.
* Use deterministic default ordering.
* Apply soft delete filters when needed.
* Use eager loading when displaying relationship data.
* Reload DataTable when filters change.

## Agent Behavior

When applying this skill:

1. Inspect existing DataTable patterns first.
2. Follow existing route, service, repository, and Blade conventions.
3. Generate code using the same feature naming style.
4. Add page route and DataTable AJAX route.
5. Implement Controller, Service, Repository, Blade table, filters, and action partial.
6. Keep implementation small and consistent.
7. Explain any deviation from this pattern.
