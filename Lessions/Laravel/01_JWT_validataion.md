Here's how to **implement and validate JWT tokens** in Laravel:

---

## **🔐 Implement & Validate JWT in Laravel**
JWT (JSON Web Token) is used for API authentication, ensuring secure user access.

---

## **📌 Steps to Implement JWT Authentication**

### **1️⃣ Install JWT Package**
Run the command to install `tymon/jwt-auth`:
```sh
composer require tymon/jwt-auth
```

Publish the JWT configuration:
```sh
php artisan vendor:publish --provider="Tymon\JWTAuth\Providers\LaravelServiceProvider"
```

Generate a JWT secret key:
```sh
php artisan jwt:secret
```
This will update your `.env` file with `JWT_SECRET`.

---

### **2️⃣ Configure `config/auth.php`**
Modify the `guards` section in **`config/auth.php`**:
```php
'defaults' => [
    'guard' => 'api',
    'passwords' => 'users',
],

'guards' => [
    'api' => [
        'driver' => 'jwt',
        'provider' => 'users',
    ],
],
```

---

### **3️⃣ Modify the `User` Model**
Update `app/Models/User.php` to implement `JWTSubject`:
```php
namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Tymon\JWTAuth\Contracts\JWTSubject;

class User extends Authenticatable implements JWTSubject
{
    protected $fillable = [
        'name', 'email', 'password',
    ];

    protected $hidden = [
        'password',
    ];

    public function getJWTIdentifier()
    {
        return $this->getKey();
    }

    public function getJWTCustomClaims()
    {
        return [];
    }
}
```

---

### **4️⃣ Create `AuthController` for JWT Authentication**
Run:
```sh
php artisan make:controller AuthController
```

Modify **`app/Http/Controllers/AuthController.php`**:
```php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Models\User;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Validator;
use Tymon\JWTAuth\Facades\JWTAuth;

class AuthController extends Controller
{
    public function register(Request $request)
    {
        $validator = Validator::make($request->all(), [
            'name' => 'required|string|max:255',
            'email' => 'required|string|email|max:255|unique:users',
            'password' => 'required|string|min:6',
        ]);

        if ($validator->fails()) {
            return response()->json($validator->errors(), 400);
        }

        $user = User::create([
            'name' => $request->name,
            'email' => $request->email,
            'password' => bcrypt($request->password),
        ]);

        return response()->json(['message' => 'User registered successfully!'], 201);
    }

    public function login(Request $request)
    {
        $credentials = $request->only('email', 'password');

        if (!$token = Auth::attempt($credentials)) {
            return response()->json(['error' => 'Unauthorized'], 401);
        }

        return $this->respondWithToken($token);
    }

    public function logout()
    {
        Auth::logout();
        return response()->json(['message' => 'Successfully logged out']);
    }

    public function refresh()
    {
        return $this->respondWithToken(Auth::refresh());
    }

    public function userProfile()
    {
        return response()->json(Auth::user());
    }

    protected function respondWithToken($token)
    {
        return response()->json([
            'access_token' => $token,
            'token_type' => 'bearer',
            'expires_in' => auth()->factory()->getTTL() * 60
        ]);
    }
}
```

---

### **5️⃣ Define API Routes**
Modify **`routes/api.php`**:
```php
use App\Http\Controllers\AuthController;

Route::post('/register', [AuthController::class, 'register']);
Route::post('/login', [AuthController::class, 'login']);

Route::middleware('auth:api')->group(function () {
    Route::post('/logout', [AuthController::class, 'logout']);
    Route::post('/refresh', [AuthController::class, 'refresh']);
    Route::get('/profile', [AuthController::class, 'userProfile']);
});
```

---

### **6️⃣ Test with Postman**
#### **Register**
**POST** `http://127.0.0.1:8000/api/register`
```json
{
    "name": "John Doe",
    "email": "john@example.com",
    "password": "password123"
}
```

#### **Login**
**POST** `http://127.0.0.1:8000/api/login`
```json
{
    "email": "john@example.com",
    "password": "password123"
}
```
✅ **Response:**
```json
{
    "access_token": "your-jwt-token",
    "token_type": "bearer",
    "expires_in": 3600
}
```

#### **Access Protected Routes**
Use the **Authorization Header**:
```
Authorization: Bearer your-jwt-token
```

**GET** `http://127.0.0.1:8000/api/profile`
✅ **Response:**
```json
{
    "id": 1,
    "name": "John Doe",
    "email": "john@example.com"
}
```

#### **Logout**
**POST** `http://127.0.0.1:8000/api/logout`

#### **Refresh Token**
**POST** `http://127.0.0.1:8000/api/refresh`

---

## **🎯 JWT Token Validation Middleware**
Create middleware:
```sh
php artisan make:middleware JwtMiddleware
```

Modify **`app/Http/Middleware/JwtMiddleware.php`**:
```php
namespace App\Http\Middleware;

use Closure;
use Tymon\JWTAuth\Facades\JWTAuth;
use Exception;

class JwtMiddleware
{
    public function handle($request, Closure $next)
    {
        try {
            $user = JWTAuth::parseToken()->authenticate();
        } catch (Exception $e) {
            return response()->json(['error' => 'Unauthorized'], 401);
        }

        return $next($request);
    }
}
```

Register middleware in **`app/Http/Kernel.php`**:
```php
protected $routeMiddleware = [
    'jwt.auth' => \App\Http\Middleware\JwtMiddleware::class,
];
```

Now, secure routes:
```php
Route::middleware(['jwt.auth'])->group(function () {
    Route::get('/profile', [AuthController::class, 'userProfile']);
});
```

---

## **🔥 Final Steps**
✅ Start Laravel server:
```sh
php artisan serve
```
✅ **Test using Postman or cURL** 🚀  

---

## **🎉 Summary**
✔️ Installed and configured JWT in Laravel  
✔️ Created authentication routes for **register, login, logout, refresh**  
✔️ Secured API routes using JWT middleware  
✔️ Implemented JWT validation and token management  

Now, your Laravel API is protected using **JWT authentication**! 🔐💡