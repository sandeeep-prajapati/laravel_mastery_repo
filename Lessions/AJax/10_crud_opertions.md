# **🛠️ AJAX-Based CRUD Operations in Laravel**  

This guide will help you implement **Create, Read, Update, and Delete (CRUD)** operations in **Laravel** using **AJAX**.  

---

## **1️⃣ Set Up Routes**  
Modify `routes/web.php`:  

```php
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\PostController;

Route::get('/posts', [PostController::class, 'index']);
Route::post('/posts', [PostController::class, 'store']);
Route::get('/posts/{id}', [PostController::class, 'show']);
Route::put('/posts/{id}', [PostController::class, 'update']);
Route::delete('/posts/{id}', [PostController::class, 'destroy']);
```

---

## **2️⃣ Create Controller**  
Run:

```bash
php artisan make:controller PostController
```

Modify `app/Http/Controllers/PostController.php`:

```php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Models\Post;

class PostController extends Controller
{
    public function index()
    {
        return response()->json(Post::latest()->get());
    }

    public function store(Request $request)
    {
        $post = Post::create($request->validate(['title' => 'required', 'content' => 'required']));
        return response()->json($post);
    }

    public function show($id)
    {
        return response()->json(Post::findOrFail($id));
    }

    public function update(Request $request, $id)
    {
        $post = Post::findOrFail($id);
        $post->update($request->validate(['title' => 'required', 'content' => 'required']));
        return response()->json($post);
    }

    public function destroy($id)
    {
        Post::destroy($id);
        return response()->json(['message' => 'Deleted successfully']);
    }
}
```

---

## **3️⃣ Create Model & Migration**  
Run:

```bash
php artisan make:model Post -m
```

Modify `database/migrations/xxxx_xx_xx_xxxxxx_create_posts_table.php`:

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up()
    {
        Schema::create('posts', function (Blueprint $table) {
            $table->id();
            $table->string('title');
            $table->text('content');
            $table->timestamps();
        });
    }

    public function down()
    {
        Schema::dropIfExists('posts');
    }
};
```

Run:

```bash
php artisan migrate
```

---

## **4️⃣ Create Blade View**  

Modify `resources/views/posts.blade.php`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AJAX CRUD in Laravel</title>
    <meta name="csrf-token" content="{{ csrf_token() }}">
    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
    <style>
        body { font-family: Arial, sans-serif; text-align: center; margin-top: 50px; }
        .container { width: 50%; margin: auto; }
        .input-group { margin: 10px 0; }
        .btn { background: #28a745; color: #fff; padding: 5px 10px; border: none; cursor: pointer; }
        .delete-btn { background: #d9534f; }
    </style>
</head>
<body>

    <div class="container">
        <h2>📌 AJAX CRUD in Laravel</h2>

        <!-- Create Form -->
        <div class="input-group">
            <input type="text" id="title" placeholder="Title">
            <textarea id="content" placeholder="Content"></textarea>
            <button class="btn" onclick="storePost()">Add Post</button>
        </div>

        <!-- Posts Table -->
        <table border="1" width="100%">
            <thead>
                <tr>
                    <th>Title</th>
                    <th>Content</th>
                    <th>Actions</th>
                </tr>
            </thead>
            <tbody id="postsTable"></tbody>
        </table>
    </div>

    <script>
        function fetchPosts() {
            $.ajax({
                url: "/posts",
                method: "GET",
                success: function (response) {
                    let postsHtml = "";
                    response.forEach(post => {
                        postsHtml += `<tr id="post-${post.id}">
                            <td><input type="text" value="${post.title}" id="title-${post.id}"></td>
                            <td><textarea id="content-${post.id}">${post.content}</textarea></td>
                            <td>
                                <button class="btn" onclick="updatePost(${post.id})">Update</button>
                                <button class="btn delete-btn" onclick="deletePost(${post.id})">Delete</button>
                            </td>
                        </tr>`;
                    });
                    $("#postsTable").html(postsHtml);
                }
            });
        }

        function storePost() {
            let title = $("#title").val();
            let content = $("#content").val();

            $.ajax({
                url: "/posts",
                method: "POST",
                data: { title: title, content: content, _token: $('meta[name="csrf-token"]').attr("content") },
                success: function () {
                    $("#title").val('');
                    $("#content").val('');
                    fetchPosts();
                }
            });
        }

        function updatePost(id) {
            let title = $(`#title-${id}`).val();
            let content = $(`#content-${id}`).val();

            $.ajax({
                url: `/posts/${id}`,
                method: "PUT",
                data: { title: title, content: content, _token: $('meta[name="csrf-token"]').attr("content") },
                success: function () {
                    alert("Post updated successfully!");
                }
            });
        }

        function deletePost(id) {
            $.ajax({
                url: `/posts/${id}`,
                method: "DELETE",
                data: { _token: $('meta[name="csrf-token"]').attr("content") },
                success: function () {
                    $(`#post-${id}`).remove();
                }
            });
        }

        $(document).ready(fetchPosts);
    </script>

</body>
</html>
```

---

## **5️⃣ Start Laravel Server**
Run:

```bash
php artisan serve
```

Visit **[http://127.0.0.1:8000/posts](http://127.0.0.1:8000/posts)**.

---

## **6️⃣ Features & Testing**
✅ **Fetches all posts using AJAX**  
✅ **Creates a new post without page refresh**  
✅ **Updates an existing post dynamically**  
✅ **Deletes a post instantly**  

---

### **🔥 Bonus Enhancements**
- Add **validation messages** for empty fields  
- Show **success notifications**  
- Implement **search & pagination**  
