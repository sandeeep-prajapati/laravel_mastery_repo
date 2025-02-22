To build a **real-time dashboard** in Laravel using **AJAX** and **WebSockets**, follow these steps:

---

## **🚀 Features**
✅ Real-time data updates using **WebSockets**  
✅ Data fetching using **AJAX**  
✅ Display updates in a **dynamic dashboard**  

---

## **🛠 Steps to Implement**  

### **1️⃣ Install Laravel WebSockets Package**
```sh
composer require beyondcode/laravel-websockets
```

Publish the config file:
```sh
php artisan vendor:publish --provider="BeyondCode\LaravelWebSockets\WebSocketsServiceProvider" --tag="config"
```

Migrate WebSockets table:
```sh
php artisan migrate
```

---

### **2️⃣ Configure WebSockets**
Modify **`.env`** file:  
```env
BROADCAST_DRIVER=pusher
PUSHER_APP_ID=your-app-id
PUSHER_APP_KEY=your-app-key
PUSHER_APP_SECRET=your-app-secret
PUSHER_HOST=127.0.0.1
PUSHER_PORT=6001
PUSHER_SCHEME=http
```

Modify **`config/broadcasting.php`**:
```php
'default' => env('BROADCAST_DRIVER', 'pusher'),

'connections' => [
    'pusher' => [
        'driver' => 'pusher',
        'key' => env('PUSHER_APP_KEY'),
        'secret' => env('PUSHER_APP_SECRET'),
        'app_id' => env('PUSHER_APP_ID'),
        'options' => [
            'cluster' => env('PUSHER_APP_CLUSTER'),
            'useTLS' => false,
            'host' => env('PUSHER_HOST'),
            'port' => env('PUSHER_PORT'),
            'scheme' => env('PUSHER_SCHEME'),
        ],
    ],
],
```

Modify **`config/websockets.php`**:
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
```sh
php artisan websockets:serve
```

---

### **3️⃣ Create Event for Broadcasting**
Generate an event:
```sh
php artisan make:event DataUpdated
```

Modify **`app/Events/DataUpdated.php`**:
```php
namespace App\Events;

use App\Models\DataRecord;
use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Broadcasting\PresenceChannel;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Broadcasting\ShouldBroadcast;
use Illuminate\Contracts\Broadcasting\ShouldBroadcastNow;
use Illuminate\Queue\SerializesModels;

class DataUpdated implements ShouldBroadcastNow
{
    use InteractsWithSockets, SerializesModels;

    public $data;

    public function __construct($data)
    {
        $this->data = $data;
    }

    public function broadcastOn()
    {
        return new Channel('dashboard-data');
    }

    public function broadcastAs()
    {
        return 'data.updated';
    }
}
```

---

### **4️⃣ Create Model & Migration**
```sh
php artisan make:model DataRecord -m
```

Modify **migration file**:
```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up()
    {
        Schema::create('data_records', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->integer('value');
            $table->timestamps();
        });
    }

    public function down()
    {
        Schema::dropIfExists('data_records');
    }
};
```
Run migration:
```sh
php artisan migrate
```

---

### **5️⃣ Create Controller**
```sh
php artisan make:controller DashboardController
```

Modify **`app/Http/Controllers/DashboardController.php`**:
```php
namespace App\Http\Controllers;

use App\Events\DataUpdated;
use App\Models\DataRecord;
use Illuminate\Http\Request;

class DashboardController extends Controller
{
    public function index()
    {
        return view('dashboard');
    }

    public function fetchData()
    {
        return response()->json(DataRecord::latest()->get());
    }

    public function updateData(Request $request)
    {
        $record = DataRecord::create([
            'name' => $request->name,
            'value' => $request->value,
        ]);

        broadcast(new DataUpdated(DataRecord::latest()->get()));

        return response()->json(['message' => 'Data updated successfully!', 'record' => $record]);
    }
}
```

---

### **6️⃣ Define Routes**
Modify **`routes/web.php`**:
```php
use App\Http\Controllers\DashboardController;
use Illuminate\Support\Facades\Route;

Route::get('/dashboard', [DashboardController::class, 'index']);
Route::get('/fetch-data', [DashboardController::class, 'fetchData']);
Route::post('/update-data', [DashboardController::class, 'updateData']);
```

---

### **7️⃣ Create WebSocket & AJAX Dashboard UI**
Modify **`resources/views/dashboard.blade.php`**:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Real-Time Dashboard</title>
    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/pusher/7.0.3/pusher.min.js"></script>
    <meta name="csrf-token" content="{{ csrf_token() }}">
</head>
<body>

    <h2>Real-Time Dashboard</h2>

    <form id="dataForm">
        <input type="text" id="name" name="name" placeholder="Enter Name" required>
        <input type="number" id="value" name="value" placeholder="Enter Value" required>
        <button type="submit">Submit</button>
    </form>

    <h3>Live Data Table</h3>
    <table border="1">
        <thead>
            <tr>
                <th>ID</th>
                <th>Name</th>
                <th>Value</th>
                <th>Created At</th>
            </tr>
        </thead>
        <tbody id="dataTable">
        </tbody>
    </table>

    <script>
        // Fetch initial data
        function fetchData() {
            $.ajax({
                url: "/fetch-data",
                type: "GET",
                success: function(data) {
                    updateTable(data);
                }
            });
        }

        // Update table dynamically
        function updateTable(data) {
            let rows = "";
            data.forEach(record => {
                rows += `<tr>
                            <td>${record.id}</td>
                            <td>${record.name}</td>
                            <td>${record.value}</td>
                            <td>${record.created_at}</td>
                        </tr>`;
            });
            $("#dataTable").html(rows);
        }

        // Submit new data via AJAX
        $("#dataForm").submit(function(e) {
            e.preventDefault();
            $.ajax({
                url: "/update-data",
                type: "POST",
                data: {
                    name: $("#name").val(),
                    value: $("#value").val(),
                    _token: $('meta[name="csrf-token"]').attr('content')
                },
                success: function(response) {
                    alert(response.message);
                }
            });
        });

        // Listen for WebSocket updates
        const pusher = new Pusher("{{ env('PUSHER_APP_KEY') }}", {
            cluster: "mt1",
            wsHost: "{{ env('PUSHER_HOST') }}",
            wsPort: "{{ env('PUSHER_PORT') }}",
            forceTLS: false,
            disableStats: true,
        });

        const channel = pusher.subscribe("dashboard-data");
        channel.bind("data.updated", function(data) {
            updateTable(data.data);
        });

        $(document).ready(fetchData);
    </script>

</body>
</html>
```

---

## **🔥 Final Steps**
1️⃣ Start Laravel server:
```sh
php artisan serve
```
2️⃣ Start WebSockets:
```sh
php artisan websockets:serve
```
3️⃣ Open `http://127.0.0.1:8000/dashboard` 🚀  

✅ **Done! Now your dashboard updates in real time with AJAX & WebSockets!** 💡