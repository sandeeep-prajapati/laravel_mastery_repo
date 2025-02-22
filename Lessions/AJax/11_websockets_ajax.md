# **🔥 Real-Time Data Updates with WebSockets in Laravel 🚀**

This guide walks you through implementing **real-time data updates** in Laravel using **WebSockets** and **Laravel Echo**.

---

## **1️⃣ Install Laravel WebSockets Package**
First, install the **Laravel WebSockets** package:

```bash
composer require beyondcode/laravel-websockets
```

Then, publish the configuration file:

```bash
php artisan vendor:publish --provider="BeyondCode\LaravelWebSockets\WebSocketsServiceProvider" --tag="migrations"
php artisan migrate
```

Now, publish the WebSocket configuration:

```bash
php artisan vendor:publish --provider="BeyondCode\LaravelWebSockets\WebSocketsServiceProvider"
```

---

## **2️⃣ Configure WebSockets in Laravel**
Update your `.env` file:

```ini
BROADCAST_DRIVER=pusher
PUSHER_APP_ID=your_app_id
PUSHER_APP_KEY=your_app_key
PUSHER_APP_SECRET=your_secret
PUSHER_HOST=127.0.0.1
PUSHER_PORT=6001
PUSHER_SCHEME=http
```

Now, update `config/broadcasting.php`:

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
            'useTLS' => false,
            'host' => env('PUSHER_HOST', '127.0.0.1'),
            'port' => env('PUSHER_PORT', 6001),
            'scheme' => env('PUSHER_SCHEME', 'http'),
        ],
    ],
],
```

---

## **3️⃣ Start Laravel WebSockets Server**
Run the WebSocket server:

```bash
php artisan websockets:serve
```

This will start the WebSocket server at **http://127.0.0.1:6001**.

---

## **4️⃣ Create an Event for Real-Time Updates**
Generate a Laravel event:

```bash
php artisan make:event NewPostCreated
```

Modify `app/Events/NewPostCreated.php`:

```php
namespace App\Events;

use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Broadcasting\PresenceChannel;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Broadcasting\ShouldBroadcast;
use Illuminate\Contracts\Broadcasting\ShouldBroadcastNow;
use Illuminate\Queue\SerializesModels;
use App\Models\Post;

class NewPostCreated implements ShouldBroadcastNow
{
    use InteractsWithSockets, SerializesModels;

    public $post;

    public function __construct(Post $post)
    {
        $this->post = $post;
    }

    public function broadcastOn()
    {
        return new Channel('posts');
    }

    public function broadcastWith()
    {
        return [
            'id' => $this->post->id,
            'title' => $this->post->title,
            'content' => $this->post->content,
        ];
    }
}
```

---

## **5️⃣ Broadcast Event in Controller**
Modify `app/Http/Controllers/PostController.php`:

```php
use App\Events\NewPostCreated;
use App\Models\Post;
use Illuminate\Http\Request;

class PostController extends Controller
{
    public function store(Request $request)
    {
        $post = Post::create($request->validate([
            'title' => 'required',
            'content' => 'required',
        ]));

        broadcast(new NewPostCreated($post))->toOthers();

        return response()->json($post);
    }
}
```

---

## **6️⃣ Listen to Events in JavaScript**
Include Laravel Echo and Pusher in `resources/js/app.js`:

```bash
npm install --save laravel-echo pusher-js
```

Modify `resources/js/app.js`:

```javascript
import Echo from "laravel-echo";
import Pusher from "pusher-js";

window.Pusher = Pusher;

window.Echo = new Echo({
    broadcaster: "pusher",
    key: process.env.MIX_PUSHER_APP_KEY,
    wsHost: window.location.hostname,
    wsPort: 6001,
    forceTLS: false,
    disableStats: true,
});

window.Echo.channel("posts")
    .listen("NewPostCreated", (event) => {
        console.log("New Post:", event);
        let postHtml = `<tr>
            <td>${event.id}</td>
            <td>${event.title}</td>
            <td>${event.content}</td>
        </tr>`;
        document.getElementById("postsTable").innerHTML += postHtml;
    });
```

Run:

```bash
npm run dev
```

---

## **7️⃣ Display Data in Blade File**
Modify `resources/views/posts.blade.php`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Real-Time Posts</title>
    <meta name="csrf-token" content="{{ csrf_token() }}">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
</head>
<body>

    <h2>📡 Real-Time Posts with WebSockets</h2>

    <!-- Form to Create Post -->
    <input type="text" id="title" placeholder="Title">
    <textarea id="content" placeholder="Content"></textarea>
    <button onclick="storePost()">Add Post</button>

    <!-- Posts Table -->
    <table border="1">
        <thead>
            <tr>
                <th>ID</th>
                <th>Title</th>
                <th>Content</th>
            </tr>
        </thead>
        <tbody id="postsTable"></tbody>
    </table>

    <script>
        function storePost() {
            let title = $("#title").val();
            let content = $("#content").val();

            $.ajax({
                url: "/posts",
                method: "POST",
                data: {
                    title: title,
                    content: content,
                    _token: $('meta[name="csrf-token"]').attr("content")
                },
                success: function () {
                    $("#title").val('');
                    $("#content").val('');
                }
            });
        }
    </script>

    <script src="{{ mix('js/app.js') }}"></script>

</body>
</html>
```

---

## **8️⃣ Start Everything**
Run these commands in separate terminals:

```bash
php artisan websockets:serve
```

```bash
php artisan serve
```

```bash
npm run dev
```

---

## **🎯 Features & Benefits**
✅ **Real-time post updates using WebSockets**  
✅ **No need to refresh the page**  
✅ **Efficient broadcasting with Laravel Echo**  
