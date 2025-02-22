### **AJAX File Upload with Drag-and-Drop in Laravel**
Here’s how you can **upload files using a drag-and-drop interface with AJAX in Laravel**.

---

## **1. Set Up Laravel Project**
If you haven't already set up a Laravel project, run:

```bash
composer create-project --prefer-dist laravel/laravel drag-drop-upload
cd drag-drop-upload
```

---

## **2. Configure Routes**
Modify `routes/web.php`:

```php
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\FileUploadController;

Route::get('/', function () {
    return view('drag-drop-upload');
});

Route::post('/upload', [FileUploadController::class, 'upload'])->name('file.upload');
```

---

## **3. Create Controller**
Run:

```bash
php artisan make:controller FileUploadController
```

Modify `app/Http/Controllers/FileUploadController.php`:

```php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Validator;

class FileUploadController extends Controller
{
    public function upload(Request $request)
    {
        $validator = Validator::make($request->all(), [
            'file' => 'required|mimes:jpg,jpeg,png,pdf|max:2048'
        ]);

        if ($validator->fails()) {
            return response()->json(['error' => $validator->errors()->first()], 400);
        }

        if ($request->hasFile('file')) {
            $file = $request->file('file');
            $fileName = time() . '_' . $file->getClientOriginalName();
            $file->move(public_path('uploads'), $fileName);

            return response()->json(['success' => 'File uploaded successfully!', 'file' => $fileName]);
        }

        return response()->json(['error' => 'File upload failed!'], 500);
    }
}
```

---

## **4. Create Blade View**
Create `resources/views/drag-drop-upload.blade.php`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Drag & Drop File Upload</title>
    <meta name="csrf-token" content="{{ csrf_token() }}">
    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            margin-top: 50px;
        }
        #drop-area {
            width: 400px;
            height: 200px;
            border: 2px dashed #ccc;
            display: flex;
            align-items: center;
            justify-content: center;
            margin: 20px auto;
            background-color: #f9f9f9;
        }
        #drop-area.highlight {
            border-color: blue;
        }
        #file-input {
            display: none;
        }
        #message {
            margin-top: 15px;
        }
    </style>
</head>
<body>
    <h2>Drag & Drop File Upload in Laravel</h2>

    <div id="drop-area">
        <p>Drag & Drop files here or <label for="file-input" style="cursor: pointer; color: blue;">Browse</label></p>
        <input type="file" id="file-input">
    </div>

    <div id="message"></div>

    <script>
        $(document).ready(function () {
            let dropArea = $("#drop-area");

            dropArea.on("dragover", function (e) {
                e.preventDefault();
                dropArea.addClass("highlight");
            });

            dropArea.on("dragleave", function (e) {
                e.preventDefault();
                dropArea.removeClass("highlight");
            });

            dropArea.on("drop", function (e) {
                e.preventDefault();
                dropArea.removeClass("highlight");

                let files = e.originalEvent.dataTransfer.files;
                if (files.length > 0) {
                    uploadFile(files[0]);
                }
            });

            $("#file-input").change(function () {
                let file = this.files[0];
                if (file) {
                    uploadFile(file);
                }
            });

            function uploadFile(file) {
                let formData = new FormData();
                formData.append("file", file);

                $.ajax({
                    url: "{{ route('file.upload') }}",
                    type: "POST",
                    data: formData,
                    processData: false,
                    contentType: false,
                    headers: {
                        'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
                    },
                    success: function (response) {
                        $("#message").html("<p style='color: green;'>" + response.success + "</p>");
                    },
                    error: function (xhr) {
                        $("#message").html("<p style='color: red;'>" + xhr.responseJSON.error + "</p>");
                    }
                });
            }
        });
    </script>
</body>
</html>
```

---

## **5. Run the Laravel Server**
Start your application:

```bash
php artisan serve
```

Visit [http://127.0.0.1:8000/](http://127.0.0.1:8000/) in your browser.

---

## **6. Features & Testing**
✅ **Drag & Drop support**  
✅ **File validation (only images & PDFs, max 2MB)**  
✅ **AJAX-based upload (no page refresh)**  
✅ **Success & error message display**  
✅ **Click to select file option available**  

---
