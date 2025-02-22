### **Live Search Functionality using AJAX in Laravel** 🚀  
This guide will show you how to implement **real-time search functionality** using **AJAX in Laravel**, where the results update dynamically as the user types.

---

## **1. Set Up Routes**  
Modify `routes/web.php`:

```php
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\SearchController;

Route::get('/', function () {
    return view('search');
});

Route::get('/search', [SearchController::class, 'search'])->name('search');
```

---

## **2. Create Controller**  
Run:

```bash
php artisan make:controller SearchController
```

Modify `app/Http/Controllers/SearchController.php`:

```php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Models\Product;

class SearchController extends Controller
{
    public function search(Request $request)
    {
        $query = $request->input('query');

        $products = Product::where('name', 'LIKE', "%{$query}%")->get();

        return response()->json($products);
    }
}
```

---

## **3. Create Blade View**  
Create `resources/views/search.blade.php`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Live Search in Laravel</title>
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
    </style>
</head>
<body>
    <h2>Live Search in Laravel with AJAX</h2>

    <input type="text" id="search-input" placeholder="Search products..." />

    <table id="search-results">
        <thead>
            <tr>
                <th>ID</th>
                <th>Name</th>
            </tr>
        </thead>
        <tbody></tbody>
    </table>

    <script>
        $(document).ready(function () {
            $("#search-input").on("keyup", function () {
                let query = $(this).val().trim();

                $.ajax({
                    url: "{{ route('search') }}",
                    type: "GET",
                    data: { query: query },
                    success: function (data) {
                        let tableBody = $("#search-results tbody");
                        tableBody.empty();
                        
                        if (data.length > 0) {
                            data.forEach(item => {
                                tableBody.append(`<tr><td>${item.id}</td><td>${item.name}</td></tr>`);
                            });
                        } else {
                            tableBody.append("<tr><td colspan='2'>No results found</td></tr>");
                        }
                    }
                });
            });
        });
    </script>
</body>
</html>
```

---

## **4. Create Database & Model**  
Run:

```bash
php artisan make:model Product -m
```

Modify `database/migrations/YYYY_MM_DD_create_products_table.php`:

```php
public function up()
{
    Schema::create('products', function (Blueprint $table) {
        $table->id();
        $table->string('name');
        $table->timestamps();
    });
}
```

Run:

```bash
php artisan migrate
```

Seed the database:  
Modify `database/seeders/DatabaseSeeder.php`:

```php
use App\Models\Product;

public function run()
{
    Product::insert([
        ['name' => 'iPhone'],
        ['name' => 'Samsung Galaxy'],
        ['name' => 'MacBook Pro'],
        ['name' => 'Dell XPS'],
        ['name' => 'Sony Headphones'],
        ['name' => 'AirPods'],
        ['name' => 'Apple Watch'],
        ['name' => 'Google Pixel'],
        ['name' => 'OnePlus Nord'],
    ]);
}
```

Run:

```bash
php artisan db:seed
```

---

## **5. Run Laravel Server**
Start your application:

```bash
php artisan serve
```

Visit **[http://127.0.0.1:8000/](http://127.0.0.1:8000/)** in your browser.

---

## **6. Features & Testing**
✅ **AJAX-based real-time search**  
✅ **Dynamically updates without page reload**  
✅ **Handles empty search results**  
✅ **Simple and efficient implementation**  

---

### **🔥 Bonus Enhancements**
- Add a **loading spinner** during AJAX requests  
- Implement **pagination for large results**  
- Highlight the **matched search text**  
