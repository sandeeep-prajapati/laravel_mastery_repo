### **Filter Data in a Multi-Level Table using AJAX in Laravel**  
This guide will show you how to **filter hierarchical data dynamically** in a **multi-level table using AJAX**. The table will **load data from a Laravel backend** and allow filtering based on user input.

---

## **1. Set Up Routes**  
Modify `routes/web.php`:

```php
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\MultiLevelFilterController;

Route::get('/', function () {
    return view('multi-level-filter');
});

Route::get('/fetch-data', [MultiLevelFilterController::class, 'fetchData'])->name('fetch.data');
Route::get('/filter-data', [MultiLevelFilterController::class, 'filterData'])->name('filter.data');
```

---

## **2. Create Controller**  
Run:

```bash
php artisan make:controller MultiLevelFilterController
```

Modify `app/Http/Controllers/MultiLevelFilterController.php`:

```php
namespace App\Http\Controllers;

use Illuminate\Http\Request;

class MultiLevelFilterController extends Controller
{
    private $data = [
        ['id' => 1, 'name' => 'Electronics', 'parent_id' => null],
        ['id' => 2, 'name' => 'Mobile Phones', 'parent_id' => 1],
        ['id' => 3, 'name' => 'Laptops', 'parent_id' => 1],
        ['id' => 4, 'name' => 'Samsung Galaxy', 'parent_id' => 2],
        ['id' => 5, 'name' => 'iPhone', 'parent_id' => 2],
        ['id' => 6, 'name' => 'Dell XPS', 'parent_id' => 3],
        ['id' => 7, 'name' => 'MacBook', 'parent_id' => 3]
    ];

    public function fetchData()
    {
        return response()->json($this->data);
    }

    public function filterData(Request $request)
    {
        $query = strtolower($request->query('search'));

        $filteredData = array_filter($this->data, function ($item) use ($query) {
            return strpos(strtolower($item['name']), $query) !== false;
        });

        return response()->json(array_values($filteredData));
    }
}
```

---

## **3. Create Blade View**  
Create `resources/views/multi-level-filter.blade.php`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Multi-Level Table with AJAX Filtering</title>
    <meta name="csrf-token" content="{{ csrf_token() }}">
    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
        }
        input {
            padding: 8px;
            margin-bottom: 15px;
            width: 300px;
            font-size: 16px;
        }
        table {
            width: 60%;
            margin: 0 auto;
            border-collapse: collapse;
        }
        th, td {
            border: 1px solid #ddd;
            padding: 10px;
            text-align: left;
        }
        th {
            background-color: #f4f4f4;
        }
        .expand-btn {
            cursor: pointer;
            color: blue;
        }
        .nested {
            display: none;
        }
    </style>
</head>
<body>
    <h2>Multi-Level Table with AJAX Filtering</h2>

    <input type="text" id="search-input" placeholder="Search categories..." />

    <table id="multi-level-table">
        <thead>
            <tr>
                <th>ID</th>
                <th>Name</th>
                <th>Expand</th>
            </tr>
        </thead>
        <tbody></tbody>
    </table>

    <script>
        $(document).ready(function () {
            function fetchData(query = "") {
                let url = query ? "{{ route('filter.data') }}?search=" + query : "{{ route('fetch.data') }}";
                
                $.ajax({
                    url: url,
                    type: "GET",
                    success: function (data) {
                        let groupedData = {};
                        data.forEach(item => {
                            if (!groupedData[item.parent_id]) {
                                groupedData[item.parent_id] = [];
                            }
                            groupedData[item.parent_id].push(item);
                        });

                        let tableBody = $("#multi-level-table tbody");
                        tableBody.empty();
                        renderRows(groupedData, null, tableBody);
                    }
                });
            }

            function renderRows(data, parentId, tableBody, level = 0) {
                if (data[parentId]) {
                    data[parentId].forEach(item => {
                        let row = `<tr data-id="${item.id}" data-parent="${parentId}" class="${level > 0 ? 'nested' : ''}" style="padding-left: ${level * 20}px;">
                            <td>${item.id}</td>
                            <td>${item.name}</td>
                            <td><span class="expand-btn" data-id="${item.id}">+</span></td>
                        </tr>`;
                        tableBody.append(row);

                        renderRows(data, item.id, tableBody, level + 1);
                    });
                }
            }

            $(document).on("click", ".expand-btn", function () {
                let id = $(this).data("id");
                let rows = $(`tr[data-parent='${id}']`);
                if (rows.is(":visible")) {
                    rows.hide();
                    $(this).text("+");
                } else {
                    rows.show();
                    $(this).text("-");
                }
            });

            $("#search-input").on("keyup", function () {
                let query = $(this).val().trim();
                fetchData(query);
            });

            fetchData();
        });
    </script>
</body>
</html>
```

---

## **4. Run Laravel Server**
Start your application:

```bash
php artisan serve
```

Visit **[http://127.0.0.1:8000/](http://127.0.0.1:8000/)** in your browser.

---

## **5. Features & Testing**
✅ **AJAX-based data fetching**  
✅ **Real-time filtering using input search**  
✅ **Expandable & collapsible nested rows**  
✅ **Hierarchical multi-level structure**  
✅ **Dynamic rendering without page reload**  

---

### **🔥 Bonus Enhancements**
- Use **icons (🔽/🔼) instead of `+/-` buttons**  
- Implement **sorting & pagination**  
- Integrate **database using Eloquent models**  
