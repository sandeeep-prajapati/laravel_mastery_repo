### 🚀 **Implement API Rate Limiting in Laravel**  

**Rate limiting** helps prevent abuse by restricting the number of requests a user can make within a given time. Laravel provides built-in support for rate limiting using **throttles** in middleware.

---

## **1️⃣ Basic API Rate Limiting**
Laravel includes rate limiting by default in `api.php`. Modify it to limit API requests:

📂 **Modify `routes/api.php`**  
```php
Route::middleware(['auth:api', 'throttle:60,1'])->group(function () {
    Route::get('/profile', [UserController::class, 'profile']);
});
```
🔹 This allows **60 requests per minute** per user.

---

## **2️⃣ Customizing Rate Limits**
Modify **app/Http/Kernel.php** in the `$middlewareAliases` section:
  
📂 **Modify `app/Http/Kernel.php`**  
```php
protected $middlewareAliases = [
    'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
];
```

Define custom throttles in the `boot()` method of **AppServiceProvider.php**:

📂 **Modify `app/Providers/AppServiceProvider.php`**  
```php
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\RateLimiter;

public function boot()
{
    RateLimiter::for('api', function (Request $request) {
        return Limit::perMinute(100)->by($request->user()?->id ?: $request->ip());
    });
}
```
🔹 This limits users to **100 requests per minute** based on user ID or IP address.

---

## **3️⃣ Creating Multiple Rate Limits**
You can create different rate limits for different endpoints.

📂 **Modify `AppServiceProvider.php`**  
```php
RateLimiter::for('login', function (Request $request) {
    return Limit::perMinute(5)->by($request->ip())->response(function() {
        return response()->json(['message' => 'Too many login attempts. Try again later.'], 429);
    });
});

RateLimiter::for('data-fetch', function (Request $request) {
    return Limit::perMinute(200)->by($request->user()?->id ?: $request->ip());
});
```

📂 **Modify `routes/api.php`**  
```php
Route::post('/login', [AuthController::class, 'login'])->middleware('throttle:login');
Route::get('/data', [DataController::class, 'fetch'])->middleware('throttle:data-fetch');
```
🔹 `login` API allows **5 attempts per minute**  
🔹 `data-fetch` API allows **200 requests per minute**  

---

## **4️⃣ Advanced: Rate Limiting with Redis**
For better performance, use **Redis** instead of default cache.

📂 **Modify `.env`**  
```env
CACHE_DRIVER=redis
```
Ensure **Redis is installed** and Laravel is configured properly.

---

## **5️⃣ Testing Rate Limits**
Test using **Postman** or `cURL`:  
```sh
curl -X GET http://127.0.0.1:8000/api/profile -H "Authorization: Bearer your_token"
```
After exceeding the limit, Laravel returns:
```json
{
    "message": "Too many requests",
    "retry_after": 30
}
```
---

### **✅ Summary**
| **Method**        | **Use Case** |
|------------------|-------------|
| `throttle:60,1` | Basic rate limiting (60 requests/min) |
| Custom `RateLimiter::for()` | Different limits for different APIs |
| Redis Backend | Faster rate limiting in production |

🔥 **Done! Your Laravel API is now secured with rate limiting!** 🚀 Let me know if you need improvements. 😃