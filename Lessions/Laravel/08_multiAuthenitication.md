### **Multi-Authentication in Laravel (Admin & User)**
Laravel provides a flexible way to set up **multi-authentication** using guards and middleware. Follow this guide to implement multi-authentication for **Admin and User**.

---

## **1. Install Laravel Authentication**
If not installed, set up Laravel authentication:
```sh
composer require laravel/ui
php artisan ui bootstrap --auth
npm install && npm run dev
```

---

## **2. Modify `config/auth.php`**
Update the **guards** and **providers**:
```php
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],
    'admin' => [  // Custom guard for admin
        'driver' => 'session',
        'provider' => 'admins',
    ],
],

'providers' => [
    'users' => [
        'driver' => 'eloquent',
        'model' => App\Models\User::class,
    ],
    'admins' => [
        'driver' => 'eloquent',
        'model' => App\Models\Admin::class,
    ],
],
```

---

## **3. Create Admin Model and Migration**
Generate an `Admin` model and migration:
```sh
php artisan make:model Admin -m
```
Edit `database/migrations/xxxx_xx_xx_create_admins_table.php`:
```php
Schema::create('admins', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('email')->unique();
    $table->string('password');
    $table->rememberToken();
    $table->timestamps();
});
```
Run the migration:
```sh
php artisan migrate
```

---

## **4. Define Admin Model**
Edit `app/Models/Admin.php`:
```php
namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class Admin extends Authenticatable
{
    use Notifiable;

    protected $fillable = ['name', 'email', 'password'];
    protected $hidden = ['password', 'remember_token'];
}
```

---

## **5. Create Authentication Controllers**
Generate controllers:
```sh
php artisan make:controller Auth/AdminLoginController
```

#### **User Authentication (`LoginController.php`)**
Edit `app/Http/Controllers/Auth/LoginController.php`:
```php
namespace App\Http\Controllers\Auth;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

class LoginController extends Controller
{
    public function showLoginForm()
    {
        return view('auth.login');
    }

    public function login(Request $request)
    {
        $credentials = $request->only('email', 'password');

        if (Auth::guard('web')->attempt($credentials)) {
            return redirect()->route('user.dashboard');
        }
        return back()->withErrors(['email' => 'Invalid credentials']);
    }

    public function logout()
    {
        Auth::guard('web')->logout();
        return redirect('/');
    }
}
```

#### **Admin Authentication (`AdminLoginController.php`)**
Edit `app/Http/Controllers/Auth/AdminLoginController.php`:
```php
namespace App\Http\Controllers\Auth;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

class AdminLoginController extends Controller
{
    public function showLoginForm()
    {
        return view('auth.admin-login');
    }

    public function login(Request $request)
    {
        $credentials = $request->only('email', 'password');

        if (Auth::guard('admin')->attempt($credentials)) {
            return redirect()->route('admin.dashboard');
        }
        return back()->withErrors(['email' => 'Invalid credentials']);
    }

    public function logout()
    {
        Auth::guard('admin')->logout();
        return redirect('/admin/login');
    }
}
```

---

## **6. Create Middleware for Authentication**
Run:
```sh
php artisan make:middleware AdminMiddleware
```
Edit `app/Http/Middleware/AdminMiddleware.php`:
```php
namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

class AdminMiddleware
{
    public function handle(Request $request, Closure $next)
    {
        if (!Auth::guard('admin')->check()) {
            return redirect('/admin/login');
        }
        return $next($request);
    }
}
```
Register it in `app/Http/Kernel.php`:
```php
protected $routeMiddleware = [
    'admin' => \App\Http\Middleware\AdminMiddleware::class,
];
```

---

## **7. Define Routes**
Edit `routes/web.php`:
```php
use App\Http\Controllers\Auth\AdminLoginController;
use App\Http\Controllers\Auth\LoginController;

Route::get('/', function () {
    return view('welcome');
});

// User Routes
Route::get('/login', [LoginController::class, 'showLoginForm'])->name('user.login');
Route::post('/login', [LoginController::class, 'login']);
Route::get('/dashboard', function () {
    return view('user.dashboard');
})->middleware('auth:web')->name('user.dashboard');
Route::post('/logout', [LoginController::class, 'logout'])->name('user.logout');

// Admin Routes
Route::get('/admin/login', [AdminLoginController::class, 'showLoginForm'])->name('admin.login');
Route::post('/admin/login', [AdminLoginController::class, 'login']);
Route::get('/admin/dashboard', function () {
    return view('admin.dashboard');
})->middleware('admin')->name('admin.dashboard');
Route::post('/admin/logout', [AdminLoginController::class, 'logout'])->name('admin.logout');
```

---

## **8. Create Login Views**
### **User Login View (`resources/views/auth/login.blade.php`)**
```html
<form method="POST" action="{{ route('user.login') }}">
    @csrf
    <input type="email" name="email" placeholder="Email">
    <input type="password" name="password" placeholder="Password">
    <button type="submit">Login</button>
</form>
```

### **Admin Login View (`resources/views/auth/admin-login.blade.php`)**
```html
<form method="POST" action="{{ route('admin.login') }}">
    @csrf
    <input type="email" name="email" placeholder="Admin Email">
    <input type="password" name="password" placeholder="Password">
    <button type="submit">Login</button>
</form>
```

---

## **9. Run Your Application**
```sh
php artisan serve
```

---

## **10. Testing Multi-Authentication**
1. **Register an Admin**
   ```sh
   php artisan tinker
   ```
   Then, create an admin:
   ```php
   App\Models\Admin::create([
       'name' => 'Admin User',
       'email' => 'admin@example.com',
       'password' => bcrypt('password123')
   ]);
   ```
2. **Login as a User**
   - Visit: `http://localhost:8000/login`
   - Use **user credentials**.
   - It will redirect to `user/dashboard`.

3. **Login as an Admin**
   - Visit: `http://localhost:8000/admin/login`
   - Use **admin credentials**.
   - It will redirect to `admin/dashboard`.

---

## **🎯 Summary**
✅ **User and Admin authentication configured**  
✅ **Separate login pages & dashboards**  
✅ **Protected routes using middleware**  
✅ **Session-based authentication for multiple guards**

Now, you have successfully implemented **multi-authentication** in Laravel! 🚀 Let me know if you have any doubts. 😊