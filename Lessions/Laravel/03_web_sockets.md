### **📡 Implementing WebSockets for Real-Time Data in Laravel**  

WebSockets enable real-time communication in Laravel using **Laravel Echo** and **Pusher**. This guide walks you through setting up WebSockets and sending real-time data.

---

## **1️⃣ Install Laravel WebSockets Package**  
Run the following command to install the package:
```sh
composer require beyondcode/laravel-websockets
```

Now, publish the package configuration:
```sh
php artisan vendor:publish --provider="BeyondCode\LaravelWebSockets\WebSocketsServiceProvider"
```

---

## **2️⃣ Configure `.env` File**  
Modify `.env` to use WebSockets instead of Pusher:
```ini
BROADCAST_DRIVER=pusher

PUSHER_APP_ID=my-app-id
PUSHER_APP_KEY=my-app-key
PUSHER_APP_SECRET=my-app-secret
PUSHER_APP_CLUSTER=mt1
PUSHER_HOST=127.0.0.1
PUSHER_PORT=6001
PUSHER_SCHEME=http
```
---

## **3️⃣ Configure WebSockets in Laravel**  
Modify `config/broadcasting.php`:
```php
'default' => env('BROADCAST_DRIVER', 'log'),

'connections' => [
    'pusher' => [
        'driver' => 'pusher',
        'key' => env('PUSHER_APP_KEY'),
        'secret' => env('PUSHER_APP_SECRET'),
        'app_id' => env('PUSHER_APP_ID'),
        'options' => [
            'cluster' => env('PUSHER_APP_CLUSTER'),
            'host' => env('PUSHER_HOST'),
            'port' => env('PUSHER_PORT'),
            'scheme' => env('PUSHER_SCHEME'),
            'useTLS' => false,
        ],
    ],
],
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

---

## **4️⃣ Set Up WebSockets Provider**
Modify `config/app.php`:
```php
App\Providers\BroadcastServiceProvider::class,
```
Run:
```sh
php artisan config:clear
```

---

## **5️⃣ Run WebSockets Server**
Start the WebSockets server:
```sh
php artisan websockets:serve
```

---

## **6️⃣ Create a WebSocket Event**  
Run:
```sh
php artisan make:event NewMessage
```
Modify **`app/Events/NewMessage.php`**:
```php
namespace App\Events;

use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Broadcasting\PresenceChannel;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Broadcasting\ShouldBroadcast;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class NewMessage implements ShouldBroadcast
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    public $message;

    public function __construct($message)
    {
        $this->message = $message;
    }

    public function broadcastOn()
    {
        return new Channel('chat');
    }

    public function broadcastAs()
    {
        return 'new.message';
    }
}
```
---

## **7️⃣ Define Broadcast Channels**  
Modify `routes/channels.php`:
```php
use Illuminate\Support\Facades\Broadcast;

Broadcast::channel('chat', function () {
    return true;
});
```
---

## **8️⃣ Trigger Event in Controller**  
Modify **`app/Http/Controllers/ChatController.php`**:
```php
namespace App\Http\Controllers;

use App\Events\NewMessage;
use Illuminate\Http\Request;

class ChatController extends Controller
{
    public function sendMessage(Request $request)
    {
        $message = $request->input('message');

        broadcast(new NewMessage($message))->toOthers();

        return response()->json(['message' => $message]);
    }
}
```
---

## **9️⃣ Set Up Frontend to Listen for Events**  
Install dependencies:
```sh
npm install --save laravel-echo pusher-js
```

Modify **`resources/js/bootstrap.js`**:
```js
import Echo from "laravel-echo";
import Pusher from "pusher-js";

window.Pusher = Pusher;

window.Echo = new Echo({
    broadcaster: "pusher",
    key: process.env.MIX_PUSHER_APP_KEY,
    wsHost: process.env.MIX_PUSHER_HOST,
    wsPort: process.env.MIX_PUSHER_PORT,
    forceTLS: false,
    disableStats: true,
});

window.Echo.channel('chat')
    .listen('.new.message', (event) => {
        console.log("New Message:", event.message);
    });
```
Run:
```sh
npm run dev
```

---

## **🔎 Test WebSockets**
1. **Start WebSockets Server**:
   ```sh
   php artisan websockets:serve
   ```
2. **Trigger a Message** via API:
   ```
   POST http://127.0.0.1:8000/api/send-message
   Body: { "message": "Hello WebSockets!" }
   ```
3. **Check Browser Console** for real-time updates.

---

## **🎉 Summary**
✔ Installed WebSockets package  
✔ Configured `.env` & broadcasting settings  
✔ Created a **WebSocket event**  
✔ Defined a **broadcast channel**  
✔ Triggered event from a **controller**  
✔ Listened to events in **frontend**  

Your Laravel app now supports **real-time WebSockets**! 🚀🔥