---
title: "Workshop: สร้างระบบ Blog อย่างง่ายด้วย Laravel 11 (ตอนที่ 1)"
weight: 120
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Laravel 11", "Workshop", "CRUD", "Blog", "MVC"]
---

{{< figure src="laravel-11-blog-workshop-part-1-cover.jpg" alt="A developer workspace with holographic blueprints representing a Laravel CRUD Workshop" >}}

## 1. 🎯 ชื่อตอน (Title): ตอนที่ 25: Workshop ลุยสนามจริง! ประกอบร่างวิชาสร้างระบบ Blog ด้วยคอนเซปต์ CRUD

## 2. 📖 เปิดฉาก (The Hook):
มาครับน้องๆ! ตลอดหลายตอนที่ผ่านมา เราได้เรียนรู้อาวุธแต่ละชิ้นของ Laravel 11 กันไปหมดแล้ว ไม่ว่าจะเป็นการกำหนดเส้นทาง (Routing), การรับออเดอร์ด้วย Controller, การปั้นหน้าตาด้วย Blade Views, และการคุยกับฐานข้อมูลด้วย Eloquent ORM 

ถ้าเปรียบเทียบก็เหมือนตอนนี้เรามีทั้งค้อน เลื่อย ตะปู และไม้กระดานชั้นดีวางอยู่ตรงหน้าแล้ว... วันนี้ล่ะครับ! พี่จะพาน้องๆ สวมวิญญาณช่างไม้ นำเครื่องมือทั้งหมดนี้มาประกอบร่างสร้าง "บ้าน" หลังแรกของเรากัน นั่นก็คือ **"ระบบ Blog อย่างง่าย"** 

เราจะมาลุยทำระบบ **CRUD (Create, Read, Update, Delete)** ซึ่งเป็นหัวใจสำคัญระดับพื้นฐานที่เว็บแอปพลิเคชัน 99% บนโลกนี้ต้องมี (ทำอันนี้เป็น ก็ทำระบบจัดการสินค้า หรือระบบพนักงานได้หมดครับ) เตรียมเปิด Visual Studio Code ให้พร้อม ร่ายมนต์เรียก Terminal ขึ้นมา แล้วไปลุยกันเลย!

## 3. 🧠 แก่นวิชา (Core Concepts):
ก่อนจะไปเขียนโค้ด เรามาทบทวนแผนผังของระบบ CRUD กันก่อนครับ ระบบนี้จะประกอบด้วย 4 การกระทำหลัก ซึ่ง Laravel ได้ออกแบบสิ่งที่เรียกว่า **"Resource Controller"** มาให้เราจัดการเรื่องพวกนี้ได้ง่ายสุดๆ โดยจับคู่กับ HTTP Verbs ดังนี้ครับ:

*   **C - Create (สร้าง):** ต้องใช้ 2 Routes คือ `GET` (เพื่อแสดงหน้าฟอร์มสร้างบทความ) และ `POST` (เพื่อรับข้อมูลจากฟอร์มไปบันทึกลง Database)
*   **R - Read (อ่าน):** ใช้ 2 Routes คือ `GET` สำหรับเมธอด `index` (ดูบทความทั้งหมด) และ `GET` สำหรับเมธอด `show` (ดูรายละเอียดบทความเดียว)
*   **U - Update (แก้ไข):** คล้ายกับ Create ครับ คือต้องมี `GET` (แสดงหน้าฟอร์มแก้ไข) และ `PUT/PATCH` (รับข้อมูลไปอัปเดตทับของเดิม)
*   **D - Delete (ลบ):** ใช้ `DELETE` (เพื่อสั่งลบบทความออกจากระบบ)

{{< figure src="laravel-11-blog-workshop-part-1-diagram.jpg" alt="System flow diagram illustrating the CRUD architecture in Laravel" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code):
เราจะทำตามสเต็ปของสถาปนิกที่ดี คือเริ่มจาก "ฐานข้อมูล -> ควบคุมการจราจร -> สร้างหน้าจอแสดงผล" ครับ

**สเต็ปที่ 1: สร้าง Model, Migration และ Controller ในคำสั่งเดียว!**
ใช้ไม้ตาย Artisan ของเราครับ พิมพ์คำสั่งนี้ลงใน Terminal:
```bash
# -m คือสร้าง Migration, -c คือสร้าง Controller, -r คือให้ Controller เป็นแบบ Resource (มี CRUD ให้ครบ)
php artisan make:model Post -mcr
```

**สเต็ปที่ 2: วาดแบบแปลนฐานข้อมูล (`database/migrations/xxxx_create_posts_table.php`)**
```php
public function up(): void
{
    Schema::create('posts', function (Blueprint $table) {
        $table->id();
        $table->string('title'); // คอลัมน์เก็บชื่อบทความ
        $table->text('content'); // คอลัมน์เก็บเนื้อหาบทความ
        $table->timestamps(); // สร้าง created_at และ updated_at ให้อัตโนมัติ
    });
}
```
อย่าลืมรัน `php artisan migrate` เพื่อสร้างตารางในฐานข้อมูลนะครับ!

**สเต็ปที่ 3: ปลดล็อค Model (`app/Models/Post.php`)**
อนุญาตให้ Laravel บันทึกข้อมูลลง 2 คอลัมน์นี้ได้ (ป้องกัน Mass Assignment)
```php
class Post extends Model
{
    // อนุญาตให้บันทึกข้อมูลแบบกลุ่มได้เฉพาะ title และ content
    protected $fillable = ['title', 'content']; 
}
```

**สเต็ปที่ 4: กำหนดเส้นทาง (`routes/web.php`)**
ความหล่อของ Laravel คือเราไม่ต้องเขียน Route ทั้ง 7 เส้นเอง ใช้คำสั่งเดียวจบเลยครับ!
```php
use App\Http\Controllers\PostController;

// คำสั่งเดียวนี้ สร้าง Routes สำหรับ CRUD ให้ครบถ้วน!
Route::resource('posts', PostController::class);
```

**สเต็ปที่ 5: จัดการ Logic ใน Controller (`app/Http/Controllers/PostController.php`)**
```php
namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Http\Request;

class PostController extends Controller
{
    // 1. อ่านทั้งหมด (Read - All)
    public function index()
    {
        $posts = Post::latest()->get(); // ดึงบทความทั้งหมด เรียงจากใหม่ไปเก่า
        return view('posts.index', compact('posts'));
    }

    // 2. แสดงฟอร์มสร้าง (Create - Form)
    public function create()
    {
        return view('posts.create');
    }

    // 3. รับข้อมูลมาบันทึก (Create - Store)
    public function store(Request $request)
    {
        // ตรวจสอบข้อมูลก่อน (Validation)
        $validated = $request->validate([
            'title' => 'required|max:255',
            'content' => 'required'
        ]);

        Post::create($validated); // บันทึกลง Database
        return redirect()->route('posts.index')->with('success', 'สร้างบทความสำเร็จ!');
    }

    // 4. อ่านบทความเดียว (Read - Single)
    public function show(Post $post)
    {
        return view('posts.show', compact('post'));
    }

    // 5. แสดงฟอร์มแก้ไข (Update - Form)
    public function edit(Post $post)
    {
        return view('posts.edit', compact('post'));
    }

    // 6. รับข้อมูลมาอัปเดต (Update - Save)
    public function update(Request $request, Post $post)
    {
        $validated = $request->validate([
            'title' => 'required|max:255',
            'content' => 'required'
        ]);

        $post->update($validated); // อัปเดตข้อมูล
        return redirect()->route('posts.index')->with('success', 'แก้ไขบทความสำเร็จ!');
    }

    // 7. ลบข้อมูล (Delete)
    public function destroy(Post $post)
    {
        $post->delete(); // ลบทิ้งเลย
        return redirect()->route('posts.index')->with('success', 'ลบบทความเรียบร้อย!');
    }
}
```

**สเต็ปที่ 6: สร้างหน้าจอ Blade Views (`resources/views/posts/`)**
พี่ขอยกตัวอย่าง 2 หน้าหลักๆ นะครับ คือหน้า List (index) และหน้าสร้าง (create)

*หน้า `index.blade.php` (หน้ารวมบทความ)*
```html
<h1>ระบบ Blog ของฉัน</h1>
<a href="{{ route('posts.create') }}">สร้างบทความใหม่</a>

<!-- แสดงข้อความแจ้งเตือน -->
@if(session('success'))
    <div style="color: green;">{{ session('success') }}</div>
@endif

<ul>
    @foreach($posts as $post)
        <li>
            <!-- คลิกลิงก์ไปดูรายละเอียด -->
            <a href="{{ route('posts.show', $post->id) }}">{{ $post->title }}</a>
            
            <a href="{{ route('posts.edit', $post->id) }}">แก้ไข</a>
            
            <!-- ฟอร์มสำหรับปุ่มลบ (ต้องใช้ฟอร์มเพราะเป็น DELETE Method) -->
            <form action="{{ route('posts.destroy', $post->id) }}" method="POST" style="display:inline;">
                @csrf
                @method('DELETE')
                <button type="submit" onclick="return confirm('แน่ใจนะว่าจะลบ?')">ลบ</button>
            </form>
        </li>
    @endforeach
</ul>
```

*หน้า `create.blade.php` (หน้าสร้างบทความ)*
```html
<h1>เขียนบทความใหม่</h1>

<form action="{{ route('posts.store') }}" method="POST">
    <!-- อย่าลืม @csrf เด็ดขาด ป้องกันการโดนแฮก! -->
    @csrf 
    
    <div>
        <label>ชื่อหัวข้อ:</label>
        <input type="text" name="title" value="{{ old('title') }}">
        @error('title') <span style="color:red;">{{ $message }}</span> @enderror
    </div>
    
    <div>
        <label>เนื้อหา:</label>
        <textarea name="content">{{ old('content') }}</textarea>
        @error('content') <span style="color:red;">{{ $message }}</span> @enderror
    </div>
    
    <button type="submit">บันทึกบทความ</button>
</form>
```

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips):
*   **เวทมนตร์ Route Model Binding:** สังเกตใน Controller ไหมครับว่า เมธอด `show(Post $post)` พี่ไม่ได้เขียนโค้ด `Post::find($id)` เลยสักบรรทัดเดียว! นี่คือความอัจฉริยะของ Laravel มันเห็นว่าเรา Type-hint คลาส `Post` มันเลยไปดึงข้อมูลจาก Database ที่ ID ตรงกับ URL มาให้เราอัตโนมัติ โค้ดคลีนขึ้นแบบสุดๆ
*   **Method Spoofing หลอกเบราว์เซอร์ให้เนียน:** ด้วยความที่ HTML Form ปกติมันรู้จักแค่ `GET` กับ `POST` เวลาเราจะทำปุ่มลบ (Delete) หรือฟอร์มแก้ไข (Put/Patch) เราต้องใช้ท่าไม้ตาย `@method('DELETE')` แทรกไว้ในฟอร์มครับ Laravel จะแอบอ่านค่านี้แล้วแปลงร่าง Request ให้กลายเป็น Method ที่ถูกต้องตอนถึงเซิร์ฟเวอร์
*   **รักษาค่าเดิมด้วย `old()`:** ในหน้า Form Create ถ้าผู้ใช้กรอกข้อมูลผิดแล้วโดนเด้งกลับมา หน้าจอจะว่างเปล่า (หงุดหงิดมาก!) เราสามารถใช้ฟังก์ชัน `{{ old('title') }}` แปะไว้ที่ attribute `value` ของ Input ได้เลย Laravel จะจำค่าที่เคยพิมพ์ไว้ก่อนหน้ามาใส่คืนให้แบบหล่อๆ ครับ

## 6. 🏁 บทสรุป (To be continued...):
เยี่ยมมากครับทุกคน! ตอนนี้เราได้นำสถาปัตยกรรม MVC มาใช้งานจริง จนสามารถสร้างระบบ Blog ที่เพิ่ม อ่าน แก้ไข และลบ ข้อมูล (CRUD) ลงฐานข้อมูลได้สำเร็จแล้ว 

แต่ระบบ Blog ของเราในตอนนี้ยังมีช่องโหว่อยู่ นั่นคือ "ใครๆ ก็เข้ามากดลบบทความเราได้!" ในตอนต่อไป (ตอนที่ 2) พี่จะพาไปติดเกราะป้องกันด้วยระบบ **"Authentication"** (ล็อกอินก่อนถึงจะเขียน/ลบได้) และเชื่อมความสัมพันธ์ **"Relationships"** ว่าบทความนี้ใครเป็นคนเขียน... เตรียมกาแฟให้พร้อม แล้วพบกันใน Workshop ภาคจบครับ!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Automation ให้กับโรงงานของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและติดตั้งระบบแบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p