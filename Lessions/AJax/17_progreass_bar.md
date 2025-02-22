### 🚀 **Implement a Progress Bar Using AJAX in Laravel**  

A **progress bar** is useful when uploading files or performing long-running tasks. In Laravel, we can achieve this using **AJAX, jQuery, and Laravel controllers**.  

---

## **🛠️ Steps to Implement a Progress Bar**  

### **1️⃣ Install Laravel Project (Skip if already set up)**
```sh
composer create-project laravel/laravel ajax-progress
cd ajax-progress
php artisan serve
```

---

### **2️⃣ Create Routes**  
Modify `routes/web.php` to define upload routes:

📂 **Modify `routes/web.php`**  
```php
use App\Http\Controllers\FileUploadController;
use Illuminate\Support\Facades\Route;

Route::get('/upload', [FileUploadController::class, 'index']);
Route::post('/upload', [FileUploadController::class, 'upload'])->name('file.upload');
```

---

### **3️⃣ Create Controller**  
Generate a new controller for handling file uploads.

```sh
php artisan make:controller FileUploadController
```

Modify the generated controller:

📂 **Modify `app/Http/Controllers/FileUploadController.php`**  
```php
namespace App\Http\Controllers;

use Illuminate\Http\Request;

class FileUploadController extends Controller
{
    public function index()
    {
        return view('upload');
    }

    public function upload(Request $request)
    {
        $request->validate([
            'file' => 'required|mimes:jpg,png,pdf|max:2048'
        ]);

        if ($request->file('file')) {
            $file = $request->file('file');
            $filename = time().'_'.$file->getClientOriginalName();
            $file->move(public_path('uploads'), $filename);

            return response()->json(['message' => 'File uploaded successfully!', 'filename' => $filename]);
        }

        return response()->json(['error' => 'File upload failed!'], 400);
    }
}
```

---

### **4️⃣ Create Blade View with Progress Bar**
Modify the view to include a file upload form and a **progress bar**.

📂 **Create `resources/views/upload.blade.php`**  
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AJAX File Upload with Progress Bar</title>
    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
    <style>
        #progress-container {
            width: 100%;
            background-color: #ddd;
            margin-top: 10px;
        }
        #progress-bar {
            width: 0%;
            height: 20px;
            background-color: #4caf50;
            text-align: center;
            line-height: 20px;
            color: white;
        }
    </style>
</head>
<body>

    <h2>Upload File with AJAX Progress Bar</h2>
    <form id="uploadForm">
        @csrf
        <input type="file" id="file" name="file">
        <button type="submit">Upload</button>
    </form>

    <div id="progress-container">
        <div id="progress-bar">0%</div>
    </div>

    <p id="message"></p>

    <script>
        $(document).ready(function(){
            $('#uploadForm').on('submit', function(e){
                e.preventDefault();
                
                var formData = new FormData();
                formData.append('file', $('#file')[0].files[0]);
                formData.append('_token', '{{ csrf_token() }}');

                $.ajax({
                    url: "{{ route('file.upload') }}",
                    type: "POST",
                    data: formData,
                    processData: false,
                    contentType: false,
                    xhr: function() {
                        var xhr = new window.XMLHttpRequest();
                        xhr.upload.addEventListener("progress", function(evt) {
                            if (evt.lengthComputable) {
                                var percentComplete = (evt.loaded / evt.total) * 100;
                                $("#progress-bar").width(percentComplete + "%");
                                $("#progress-bar").text(Math.round(percentComplete) + "%");
                            }
                        }, false);
                        return xhr;
                    },
                    success: function(response) {
                        $("#message").text(response.message);
                        $("#progress-bar").css("background-color", "#4caf50");
                    },
                    error: function(response) {
                        $("#message").text("Upload failed.");
                        $("#progress-bar").css("background-color", "red");
                    }
                });
            });
        });
    </script>

</body>
</html>
```

---

## **🎯 Explanation**
1. **Frontend**
   - `#uploadForm` sends a file using AJAX.
   - `#progress-bar` updates based on upload progress.
   - `xhr.upload.addEventListener("progress", function(evt) {...}` calculates progress dynamically.

2. **Backend**
   - Validates file (`jpg, png, pdf`, max size: **2MB**).
   - Moves file to `public/uploads/` directory.
   - Returns a JSON response with success/error messages.

---

## **✅ Test the Upload**
Run your Laravel application:
```sh
php artisan serve
```
Visit: **`http://127.0.0.1:8000/upload`**  
Select a file and upload to see the **progress bar in action!** 🚀

Let me know if you need modifications! 😃