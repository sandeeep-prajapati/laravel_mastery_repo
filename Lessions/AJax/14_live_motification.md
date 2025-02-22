# **🔥 Build a Live Notification System Using WebSockets in Laravel 🚀**  

This guide will help you create a **real-time notification system** using **Laravel WebSockets, Pusher, and Echo**.

---

## **📌 1. Install Laravel WebSockets Package**
Run:

```bash
composer require beyondcode/laravel-websockets
```

Then, publish the WebSockets configuration:

```bash
php artisan vendor:publish --provider="BeyondCode\LaravelWebSockets\WebSocketsServiceProvider" --tag="config"
```

---

## **📌 2. Configure WebSockets**
Modify `.env`:

```env
BROADCAST_DRIVER=pusher
PUSHER_APP_ID=local
PUSHER_APP_KEY=local
PUSHER_APP_SECRET=local
PUSHER_HOST=127.0.0.1
PUSHER_PORT=6001
PUSHER_SCHEME=http
```

Modify `config/broadcasting.php`:

```php
'connections' => [
    'pusher' => [
        'driver' => 'pusher',
        'key' => env('PUSHER_APP_KEY'),
        'secret' => env('PUSHER_APP_SECRET'),
        'app_id' => env('PUSHER_APP_ID'),
        'options' => [
            'host' => env('PUSHER_HOST', '127.0.0.1'),
            'port' => env('PUSHER_PORT', 6001),
            'scheme' => env('PUSHER_SCHEME', 'http'),
            'encrypted' => false,
            'useTLS' => false,
        ],
    ],
]
```

Modify `config/websockets.php`:

```php
'apps' => [
    [
        'id' => env('PUSHER_APP_ID'),
        'name' => env('APP_NAME'),
        'key' => env('PUSHER_APP_KEY'),
        'secret' => env('PUSHER_APP_SECRET'),
        'enable_client_messages' => true,
        'enable_statistics' => true,
    ],
],
```

Run WebSockets server:

```bash
php artisan websockets:serve
```

---

## **📌 3. Create Event for Notifications**
Run:

```bash
php artisan make:event NewNotification
```

Modify `app/Events/NewNotification.php`:

```php
namespace App\Events;

use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Broadcasting\PresenceChannel;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class NewNotification implements ShouldBroadcast
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    public $message;

    public function __construct($message)
    {
        $this->message = $message;
    }

    public function broadcastOn()
    {
        return new Channel('notifications');
    }

    public function broadcastAs()
    {
        return 'new-notification';
    }
}
```

---

## **📌 4. Create Route to Trigger Notification**
Modify `routes/web.php`:

```php
use App\Events\NewNotification;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;

Route::get('/', function () {
    return view('notifications');
});

Route::post('/send-notification', function (Request $request) {
    event(new NewNotification($request->message));
    return response()->json(['success' => 'Notification sent']);
});
```

---

## **📌 5. Create Blade View for Notifications**
Modify `resources/views/notifications.blade.php`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Live Notifications</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/axios/1.2.2/axios.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/laravel-echo/1.11.1/echo.min.js"></script>
    <script src="https://js.pusher.com/7.0/pusher.min.js"></script>
</head>
<body>

    <h2>Live Notifications</h2>
    <div id="notificationArea"></div>

    <form id="notificationForm">
        <input type="text" id="message" placeholder="Enter notification message" required>
        <button type="submit">Send Notification</button>
    </form>

    <script>
        const form = document.getElementById("notificationForm");
        const messageInput = document.getElementById("message");
        const notificationArea = document.getElementById("notificationArea");

        form.addEventListener("submit", function(event) {
            event.preventDefault();
            axios.post("/send-notification", {
                message: messageInput.value
            }).then(response => {
                console.log("Notification sent");
            }).catch(error => {
                console.log("Error sending notification");
            });
        });

        window.Echo = new Echo({
            broadcaster: "pusher",
            key: "local",
            wsHost: window.location.hostname,
            wsPort: 6001,
            forceTLS: false,
            disableStats: true
        });

        Echo.channel("notifications")
            .listen(".new-notification", (data) => {
                notificationArea.innerHTML += `<p><strong>New Notification:</strong> ${data.message}</p>`;
            });
    </script>

</body>
</html>
```

---

## **📌 6. Run the Application**
Start Laravel server:

```bash
php artisan serve
```

Start WebSockets server:

```bash
php artisan websockets:serve
```

Visit:  
🔗 **http://127.0.0.1:8000/**

---

## **🚀 Features & Benefits**
✅ **Real-Time Notifications** without page refresh  
✅ **Efficient Communication** using WebSockets  
✅ **Scalable & Lightweight**  

🎉 **Congratulations! You've built a live notification system using WebSockets in Laravel.** 🚀 Let me know if you need enhancements!