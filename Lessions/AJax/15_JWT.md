It looks like you want to secure your Laravel API using **JWT (JSON Web Token) or OAuth (Laravel Passport).** Below is a step-by-step guide for both methods.

---

## **🔐 Option 1: Secure Laravel API with JWT**
**Best for:**  
✅ Mobile apps  
✅ Single Page Applications (SPAs) (React, Vue, Angular)  
✅ Lightweight APIs  

### **1️⃣ Install JWT Package**
Run:  
```bash
composer require tymon/jwt-auth
```

Publish the configuration:  
```bash
php artisan vendor:publish --provider="PHPOpenSourceSaver\JWTAuth\Providers\LaravelServiceProvider"
```

Generate the JWT secret key:  
```bash
php artisan jwt:secret
```

---

### **2️⃣ Configure `config/auth.php`**
Modify:

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

### **3️⃣ Create Authentication Controller**
Run:  
```bash
php artisan make:controller AuthController
```

Modify `app/Http/Controllers/AuthController.php`:

```php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Models\User;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Facades\Validator;
use PHPOpenSourceSaver\JWTAuth\Facades\JWTAuth;

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
            'password' => Hash::make($request->password),
        ]);

        $token = JWTAuth::fromUser($user);

        return response()->json(['token' => $token]);
    }

    public function login(Request $request)
    {
        $credentials = $request->only('email', 'password');

        if (!$token = JWTAuth::attempt($credentials)) {
            return response()->json(['error' => 'Unauthorized'], 401);
        }

        return response()->json(['token' => $token]);
    }

    public function profile()
    {
        return response()->json(auth()->user());
    }

    public function logout()
    {
        auth()->logout();
        return response()->json(['message' => 'Successfully logged out']);
    }
}
```

---

### **4️⃣ Define API Routes**
Modify `routes/api.php`:

```php
use App\Http\Controllers\AuthController;

Route::post('register', [AuthController::class, 'register']);
Route::post('login', [AuthController::class, 'login']);

Route::middleware(['auth:api'])->group(function () {
    Route::get('profile', [AuthController::class, 'profile']);
    Route::post('logout', [AuthController::class, 'logout']);
});
```

---

### **5️⃣ Test API with Postman**
#### **Register User**
**POST** `http://127.0.0.1:8000/api/register`  
**Body:**
```json
{
    "name": "John Doe",
    "email": "johndoe@example.com",
    "password": "password"
}
```

#### **Login**
**POST** `http://127.0.0.1:8000/api/login`  
Returns:
```json
{
    "token": "your_jwt_token_here"
}
```

#### **Access Protected Route**
**GET** `http://127.0.0.1:8000/api/profile`  
Headers:
```text
Authorization: Bearer your_jwt_token_here
```

---

## **🔐 Option 2: Secure Laravel API with OAuth (Laravel Passport)**
**Best for:**  
✅ Enterprise applications  
✅ Multi-client APIs  
✅ Secure token-based authentication  

---

### **1️⃣ Install Laravel Passport**
Run:  
```bash
composer require laravel/passport
```

Run migrations:  
```bash
php artisan migrate
php artisan passport:install
```

---

### **2️⃣ Configure `config/auth.php`**
Modify:

```php
'guards' => [
    'api' => [
        'driver' => 'passport',
        'provider' => 'users',
    ],
],
```

---

### **3️⃣ Setup Passport in `App\Models\User.php`**
Modify:

```php
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, Notifiable;
}
```

---

### **4️⃣ Modify `app/Providers/AuthServiceProvider.php`**
Modify:

```php
use Laravel\Passport\Passport;

public function boot()
{
    $this->registerPolicies();
    Passport::routes();
}
```

---

### **5️⃣ Create OAuth AuthController**
Run:  
```bash
php artisan make:controller OAuthController
```

Modify `app/Http/Controllers/OAuthController.php`:

```php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Models\User;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Facades\Validator;
use Laravel\Passport\Client;
use Illuminate\Support\Facades\Auth;

class OAuthController extends Controller
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
            'password' => Hash::make($request->password),
        ]);

        $token = $user->createToken('API Token')->accessToken;

        return response()->json(['token' => $token]);
    }

    public function login(Request $request)
    {
        if (!Auth::attempt($request->only('email', 'password'))) {
            return response()->json(['message' => 'Unauthorized'], 401);
        }

        $user = Auth::user();
        $token = $user->createToken('API Token')->accessToken;

        return response()->json(['token' => $token]);
    }

    public function profile()
    {
        return response()->json(auth()->user());
    }

    public function logout(Request $request)
    {
        $request->user()->token()->revoke();
        return response()->json(['message' => 'Successfully logged out']);
    }
}
```

---

### **6️⃣ Define API Routes**
Modify `routes/api.php`:

```php
use App\Http\Controllers\OAuthController;

Route::post('register', [OAuthController::class, 'register']);
Route::post('login', [OAuthController::class, 'login']);

Route::middleware('auth:api')->group(function () {
    Route::get('profile', [OAuthController::class, 'profile']);
    Route::post('logout', [OAuthController::class, 'logout']);
});
```

---

### **7️⃣ Test OAuth API**
#### **Register User**
**POST** `http://127.0.0.1:8000/api/register`

#### **Login**
**POST** `http://127.0.0.1:8000/api/login`  
Returns:
```json
{
    "token": "your_access_token_here"
}
```

#### **Access Protected Route**
**GET** `http://127.0.0.1:8000/api/profile`  
Headers:
```text
Authorization: Bearer your_access_token_here
```

---

## **🚀 Summary: JWT vs. OAuth**
| Feature        | JWT | OAuth (Passport) |
|---------------|-----|-----------------|
| Best for | SPAs, Mobile Apps | Enterprise APIs |
| Storage | LocalStorage, Cookies | Database |
| Token Expiry | Short-lived | Long-lived (Refresh Tokens) |
| Security | Moderate | High (Scopes, Clients) |

---

🎉 **Done! Your Laravel API is secured using JWT or OAuth.** 🚀 Let me know if you need enhancements!