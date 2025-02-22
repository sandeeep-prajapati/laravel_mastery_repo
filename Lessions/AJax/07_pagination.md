### **AJAX-Based Pagination in Laravel** 🚀  

This guide will walk you through implementing **AJAX pagination** in Laravel, where data loads dynamically without page reloads.

---

## **1. Set Up Routes**  
Modify `routes/web.php`:  

```php
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\PaginationController;

Route::get('/users', [PaginationController::class, 'index']);
Route::get('/fetch-users', [PaginationController::class, 'fetchUsers']);
```

---

## **2. Create Controller**  
Run:

```bash
php artisan make:controller PaginationController
```

Modify `app/Http/Controllers/PaginationController.php`:

```php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Models\User;

class PaginationController extends Controller
{
    public function index()
    {
        return view('users');
    }

    public function fetchUsers(Request $request)
    {
        $users = User::orderBy('id', 'desc')->paginate(5);

        if ($request->ajax()) {
            return view('user_data', compact('users'))->render();
        }

        return view('users', compact('users'));
    }
}
```

---

## **3. Create Blade Views**  

### **(a) Main View - `resources/views/users.blade.php`**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AJAX Pagination</title>
    <meta name="csrf-token" content="{{ csrf_token() }}">
    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
    <style>
        body { font-family: Arial, sans-serif; text-align: center; margin-top: 30px; }
        table { width: 60%; margin: auto; border-collapse: collapse; }
        th, td { padding: 10px; border: 1px solid #ddd; }
        th { background-color: #f4f4f4; }
        .pagination { margin-top: 15px; display: flex; justify-content: center; }
        .pagination a { padding: 8px 12px; margin: 0 5px; border: 1px solid #ddd; text-decoration: none; color: #007bff; cursor: pointer; }
        .pagination a.active { background-color: #007bff; color: white; }
        .pagination a.disabled { pointer-events: none; color: #ccc; }
    </style>
</head>
<body>

    <h2>AJAX Pagination in Laravel</h2>

    <div id="user-data">
        @include('user_data')
    </div>

    <script>
        $(document).ready(function () {
            $(document).on('click', '.pagination a', function (event) {
                event.preventDefault();
                let page = $(this).attr('href').split('page=')[1];
                fetchUsers(page);
            });

            function fetchUsers(page) {
                $.ajax({
                    url: "/fetch-users?page=" + page,
                    success: function (data) {
                        $("#user-data").html(data);
                    }
                });
            }
        });
    </script>

</body>
</html>
```

---

### **(b) Partial View - `resources/views/user_data.blade.php`**
```html
<table>
    <thead>
        <tr>
            <th>ID</th>
            <th>Name</th>
            <th>Email</th>
        </tr>
    </thead>
    <tbody>
        @foreach ($users as $user)
            <tr>
                <td>{{ $user->id }}</td>
                <td>{{ $user->name }}</td>
                <td>{{ $user->email }}</td>
            </tr>
        @endforeach
    </tbody>
</table>

<div class="pagination">
    {{ $users->links() }}
</div>
```

---

## **4. Database & Model**  
Ensure you have a `users` table. If not, run:  

```bash
php artisan make:model User -m
```

Modify `database/migrations/YYYY_MM_DD_create_users_table.php`:

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up()
    {
        Schema::create('users', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('email')->unique();
            $table->timestamps();
        });
    }

    public function down()
    {
        Schema::dropIfExists('users');
    }
};
```

Run:

```bash
php artisan migrate
```

To generate dummy users, run:

```bash
php artisan tinker
```

Inside Tinker:

```php
\Illuminate\Support\Facades\DB::table('users')->insert([
    ['name' => 'John Doe', 'email' => 'john@example.com'],
    ['name' => 'Jane Smith', 'email' => 'jane@example.com'],
    ['name' => 'Alice Brown', 'email' => 'alice@example.com'],
    ['name' => 'Bob Martin', 'email' => 'bob@example.com'],
    ['name' => 'Charlie White', 'email' => 'charlie@example.com'],
]);
```

---

## **5. Run Laravel Server**
Start your application:

```bash
php artisan serve
```

Visit **[http://127.0.0.1:8000/users](http://127.0.0.1:8000/users)** in your browser.

---

## **6. Features & Testing**
✅ **AJAX pagination without page reload**  
✅ **Pagination links update dynamically**  
✅ **Optimized for large datasets**  
✅ **Easy integration with Laravel Eloquent**  

---

### **🔥 Bonus Enhancements**
- Add **search functionality** along with pagination  
- Display **loading spinner** during AJAX requests  
- Implement **filtering options** for better UX  
