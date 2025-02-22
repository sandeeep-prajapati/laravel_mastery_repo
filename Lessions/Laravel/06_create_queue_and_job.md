### **📌 Using Queues and Jobs in Laravel**  

Queues in Laravel allow you to defer **time-consuming tasks** (like sending emails, processing notifications, or database updates) so they run asynchronously in the background, improving application performance.  

---

## **✅ Steps to Implement Queues & Jobs**
1️⃣ **Set Up Queue Driver**  
2️⃣ **Create a Job**  
3️⃣ **Dispatch the Job**  
4️⃣ **Process the Queue**  

---

## **1️⃣ Set Up Queue Driver**  

Laravel supports multiple queue drivers like **database, Redis, Beanstalk, and Amazon SQS**.  

Modify `.env`:  
```ini
QUEUE_CONNECTION=database
```

Run:  
```sh
php artisan queue:table
php artisan migrate
```

This creates the `jobs` table to store queued jobs.

---

## **2️⃣ Create a Job**  

Generate a new job:  
```sh
php artisan make:job SendNotificationJob
```

Modify `app/Jobs/SendNotificationJob.php`:  
```php
namespace App\Jobs;

use App\Models\User;
use App\Notifications\NewUserNotification;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldBeUnique;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

class SendNotificationJob implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public $user;
    public $message;

    public function __construct(User $user, $message)
    {
        $this->user = $user;
        $this->message = $message;
    }

    public function handle()
    {
        $this->user->notify(new NewUserNotification($this->message));
    }
}
```

> 🛠 **`ShouldQueue`** makes the job run in the background.

---

## **3️⃣ Dispatch the Job**  

Modify a controller:  
```php
namespace App\Http\Controllers;

use App\Jobs\SendNotificationJob;
use App\Models\User;
use Illuminate\Http\Request;

class NotificationController extends Controller
{
    public function sendNotification()
    {
        $user = User::find(1);
        dispatch(new SendNotificationJob($user, "New user registered!"));

        return response()->json(['message' => 'Notification queued']);
    }
}
```
> 🔹 **`dispatch()`** adds the job to the queue.

---

## **4️⃣ Process the Queue**  

Run the worker to process queued jobs:  
```sh
php artisan queue:work
```

---

## **🎉 Summary**  
✔ **Configured queue driver**  
✔ **Created a job class**  
✔ **Dispatched the job from a controller**  
✔ **Processed the queue with `queue:work`**  

Now, Laravel **processes notifications asynchronously**, improving performance! 🚀