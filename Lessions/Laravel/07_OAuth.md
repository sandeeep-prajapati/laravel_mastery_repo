To implement **OAuth authentication** in Laravel using **Laravel Passport**, follow these steps:

---

## **1. Install Laravel Passport**
Run the following command:
```sh
composer require laravel/passport
```
After installation, run:
```sh
php artisan migrate
php artisan passport:install
```
This creates the necessary database tables and generates encryption keys.

---

## **2. Configure Passport in `AuthServiceProvider`**
Edit `app/Providers/AuthServiceProvider.php` and update the `boot` method:
```php
use Laravel\Passport\Passport;

public function boot()
{
    $this->registerPolicies();
    Passport::routes();
}
```

---

## **3. Update `config/auth.php`**
Set Passport as the default API guard:
```php
'guards' => [
    'api' => [
        'driver' => 'passport',
        'provider' => 'users',
    ],
],
```

---

## **4. Add `HasApiTokens` to the User Model**
Edit `app/Models/User.php`:
```php
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, Notifiable;
}
```

---

## **5. Create OAuth Client Credentials**
Run:
```sh
php artisan passport:client --personal
```
This creates a personal access client, which is useful for API authentication.

---

## **6. Setup API Authentication Routes**
Edit `routes/api.php`:
```php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\AuthController;

Route::post('/register', [AuthController::class, 'register']);
Route::post('/login', [AuthController::class, 'login']);
Route::middleware('auth:api')->get('/user', function (Request $request) {
    return $request->user();
});
```

---

## **7. Create `AuthController`**
Run:
```sh
php artisan make:controller AuthController
```
Edit `app/Http/Controllers/AuthController.php`:
```php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Models\User;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Hash;

class AuthController extends Controller
{
    public function register(Request $request)
    {
        $request->validate([
            'name' => 'required|string|max:255',
            'email' => 'required|string|email|max:255|unique:users',
            'password' => 'required|string|min:6',
        ]);

        $user = User::create([
            'name' => $request->name,
            'email' => $request->email,
            'password' => Hash::make($request->password),
        ]);

        $token = $user->createToken('Personal Access Token')->accessToken;

        return response()->json(['token' => $token], 201);
    }

    public function login(Request $request)
    {
        $request->validate([
            'email' => 'required|string|email',
            'password' => 'required|string',
        ]);

        if (Auth::attempt(['email' => $request->email, 'password' => $request->password])) {
            $user = Auth::user();
            $token = $user->createToken('Personal Access Token')->accessToken;

            return response()->json(['token' => $token], 200);
        } else {
            return response()->json(['error' => 'Unauthorized'], 401);
        }
    }
}
```

---

## **8. Protect API Routes**
Edit `routes/api.php`:
```php
Route::middleware('auth:api')->group(function () {
    Route::get('/profile', function (Request $request) {
        return response()->json(['user' => $request->user()]);
    });
});
```

---

## **9. Test with Postman**
1. **Register a new user**  
   - `POST http://localhost:8000/api/register`  
   - Body (JSON):
     ```json
     {
       "name": "John Doe",
       "email": "john@example.com",
       "password": "password123"
     }
     ```
   - Response:
     ```json
     {
       "token": "eyJ0eXAiOiJK..."
     }
     ```

2. **Login**  
   - `POST http://localhost:8000/api/login`
   - Body (JSON):
     ```json
     {
       "email": "john@example.com",
       "password": "password123"
     }
     ```
   - Response:
     ```json
     {
       "token": "eyJ0eXAiOiJK..."
     }
     ```

3. **Access Protected Route**  
   - `GET http://localhost:8000/api/profile`
   - Headers:
     ```
     Authorization: Bearer YOUR_ACCESS_TOKEN
     ```
   - Response:
     ```json
     {
       "user": {
         "id": 1,
         "name": "John Doe",
         "email": "john@example.com"
       }
     }
     ```

---

## **10. Run Your Application**
```sh
php artisan serve
```
Your **OAuth authentication system** with Laravel Passport is now fully functional! 🚀

Let me know if you need any clarifications. 😊