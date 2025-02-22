### **🔥 Build a Real-Time Chat Application using WebSockets in Laravel 🚀**

This guide will walk you through **building a real-time chat app** using **Laravel, Laravel WebSockets, Laravel Echo, and Vue.js**.

---

## **📌 1. Install Laravel & WebSockets Package**
First, create a Laravel project and install the WebSockets package:

```bash
composer create-project --prefer-dist laravel/laravel ChatApp
cd ChatApp
composer require beyondcode/laravel-websockets
```

Publish the WebSockets migration and config files:

```bash
php artisan vendor:publish --provider="BeyondCode\LaravelWebSockets\WebSocketsServiceProvider" --tag="migrations"
php artisan migrate
php artisan vendor:publish --provider="BeyondCode\LaravelWebSockets\WebSocketsServiceProvider"
```

---

## **📌 2. Configure WebSockets & Broadcasting**
Update `.env`:

```ini
BROADCAST_DRIVER=pusher
PUSHER_APP_ID=your_app_id
PUSHER_APP_KEY=your_app_key
PUSHER_APP_SECRET=your_secret
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
            'cluster' => env('PUSHER_APP_CLUSTER'),
            'useTLS' => false,
            'host' => env('PUSHER_HOST', '127.0.0.1'),
            'port' => env('PUSHER_PORT', 6001),
            'scheme' => env('PUSHER_SCHEME', 'http'),
        ],
    ],
],
```

---

## **📌 3. Create Chat Models & Migration**
Run the following:

```bash
php artisan make:model Message -m
```

Modify `database/migrations/*_create_messages_table.php`:

```php
public function up()
{
    Schema::create('messages', function (Blueprint $table) {
        $table->id();
        $table->foreignId('user_id')->constrained()->onDelete('cascade');
        $table->text('message');
        $table->timestamps();
    });
}
```

Run:

```bash
php artisan migrate
```

Modify `app/Models/Message.php`:

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Message extends Model
{
    use HasFactory;
    protected $fillable = ['user_id', 'message'];

    public function user()
    {
        return $this->belongsTo(User::class);
    }
}
```

---

## **📌 4. Create Chat Event**
Run:

```bash
php artisan make:event MessageSent
```

Modify `app/Events/MessageSent.php`:

```php
namespace App\Events;

use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Broadcasting\ShouldBroadcast;
use Illuminate\Queue\SerializesModels;
use App\Models\Message;

class MessageSent implements ShouldBroadcast
{
    use InteractsWithSockets, SerializesModels;

    public $message;

    public function __construct(Message $message)
    {
        $this->message = $message;
    }

    public function broadcastOn()
    {
        return new Channel('chat');
    }

    public function broadcastWith()
    {
        return [
            'user' => $this->message->user->name,
            'message' => $this->message->message,
        ];
    }
}
```

---

## **📌 5. Create Chat Controller**
Run:

```bash
php artisan make:controller ChatController
```

Modify `app/Http/Controllers/ChatController.php`:

```php
namespace App\Http\Controllers;

use App\Events\MessageSent;
use App\Models\Message;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

class ChatController extends Controller
{
    public function index()
    {
        return Message::with('user')->get();
    }

    public function store(Request $request)
    {
        $message = Message::create([
            'user_id' => Auth::id(),
            'message' => $request->message
        ]);

        broadcast(new MessageSent($message))->toOthers();

        return response()->json($message);
    }
}
```

---

## **📌 6. Set Up Routes**
Modify `routes/api.php`:

```php
use App\Http\Controllers\ChatController;
use Illuminate\Support\Facades\Route;

Route::middleware('auth:sanctum')->group(function () {
    Route::get('/messages', [ChatController::class, 'index']);
    Route::post('/messages', [ChatController::class, 'store']);
});
```

---

## **📌 7. Set Up Vue.js for Real-Time Chat**
Install dependencies:

```bash
npm install --save vue@3 axios laravel-echo pusher-js
```

Modify `resources/js/app.js`:

```javascript
import { createApp } from "vue";
import ChatComponent from "./components/ChatComponent.vue";
createApp(ChatComponent).mount("#chat");
```

Create `resources/js/components/ChatComponent.vue`:

```vue
<template>
    <div>
        <div v-for="msg in messages" :key="msg.id">
            <strong>{{ msg.user }}:</strong> {{ msg.message }}
        </div>
        
        <input v-model="newMessage" @keyup.enter="sendMessage" placeholder="Type a message..." />
    </div>
</template>

<script>
import Echo from "laravel-echo";
import Pusher from "pusher-js";
import axios from "axios";

export default {
    data() {
        return {
            messages: [],
            newMessage: ""
        };
    },
    mounted() {
        this.fetchMessages();
        this.listenForNewMessages();
    },
    methods: {
        fetchMessages() {
            axios.get("/api/messages").then(response => {
                this.messages = response.data;
            });
        },
        listenForNewMessages() {
            window.Echo = new Echo({
                broadcaster: "pusher",
                key: process.env.MIX_PUSHER_APP_KEY,
                wsHost: window.location.hostname,
                wsPort: 6001,
                forceTLS: false,
                disableStats: true,
            });

            window.Echo.channel("chat").listen("MessageSent", event => {
                this.messages.push(event);
            });
        },
        sendMessage() {
            axios.post("/api/messages", { message: this.newMessage }).then(response => {
                this.messages.push(response.data);
                this.newMessage = "";
            });
        }
    }
};
</script>
```

---

## **📌 8. Compile Frontend & Run WebSockets**
Run:

```bash
npm run dev
php artisan serve
php artisan websockets:serve
```

---

## **🚀 Features & Benefits**
✅ **Real-time messages without refreshing**  
✅ **Scalable WebSockets using Laravel Echo**  
✅ **Secure authentication with Laravel Sanctum**  
