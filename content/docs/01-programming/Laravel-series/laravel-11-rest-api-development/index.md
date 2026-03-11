---
title: "API Development: สร้าง RESTful API ด้วย Laravel 11"
weight: 150
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Laravel 11", "API", "RESTful", "API Resources", "Postman"]
---

{{< figure src="laravel-11-rest-api-development-cover.jpg" alt="A holographic bridge connecting a server to devices representing an API" >}}

## 1. 🎯 ชื่อตอน (Title): ตอนที่ 28: เปิดประตูสู่โลกกว้าง! ปั้น RESTful API ให้เนี้ยบด้วย API Resources

## 2. 📖 เปิดฉาก (The Hook):
มาครับน้องๆ ลากเก้าอี้มานั่งจิบกาแฟกัน... ลองจินตนาการดูว่า น้องๆ เพิ่งสร้างระบบ E-Commerce ด้วย Laravel เสร็จ เว็บไซต์ทำงานได้ดีมาก แต่จู่ๆ บอสก็เดินมาบอกว่า *"น้อง! เดือนหน้าเราจะเปิดตัวแอปพลิเคชันมือถือ (Mobile App) ทั้ง iOS และ Android นะ ช่วยส่งข้อมูลสินค้าไปให้ทีม Mobile หน่อยสิ"* 

ถ้าแอปพลิเคชันของเรามีแต่หน้า HTML (Blade Template) ทีมมือถือเขาเอาไปใช้งานไม่ได้หรอกครับ! (Pain Point คลาสสิก) สิ่งที่เราต้องทำคือการเปิด "ช่องทางสื่อสารพิเศษ" เพื่อให้ระบบอื่นเข้ามาคุยกับระบบเราได้ด้วยภาษาที่เป็นสากลอย่าง **JSON (JavaScript Object Notation)** ช่องทางนั้นเราเรียกว่า **API (Application Programming Interface)**

สมัยก่อน เวลาเราจะส่งข้อมูลออกไป เราอาจจะเผลอเขียนโค้ดแบบ `return User::all();` ซึ่งมันจะพ่นข้อมูลทุกอย่างในฐานข้อมูลออกมา รวมไปถึงรหัสผ่าน (Password) และข้อมูลส่วนตัว! โคตรอันตรายแถมยังดูไม่เป็นมืออาชีพเลยครับ 

วันนี้พี่จะพาไปดูศิลปะของการทำ **RESTful API** ใน Laravel 11 และทำความรู้จักกับพระเอกของเราอย่าง **"API Resources"** ที่จะทำหน้าที่เป็น "นักตกแต่งข้อมูล (Transformer)" แปลงข้อมูลดิบๆ ให้กลายเป็น JSON ที่สวยงามและปลอดภัย พร้อมแล้วเตรียม Postman ไว้ให้ดี แล้วลุยกันเลย!

## 3. 🧠 แก่นวิชา (Core Concepts):
ก่อนจะเริ่มเขียนโค้ด เรามาทำความเข้าใจสถาปัตยกรรมกันก่อนครับ:

*   **🌐 RESTful API คืออะไร?** 
    มันคือสถาปัตยกรรมการออกแบบ API ที่เน้นการมองทุกอย่างเป็น "ทรัพยากร (Resource)" เช่น สินค้า (Products) หรือ ผู้ใช้ (Users) และใช้ **HTTP Methods (Verbs)** ในการจัดการข้อมูล เช่น `GET` (อ่าน), `POST` (สร้าง), `PUT/PATCH` (แก้ไข) และ `DELETE` (ลบ)
*   **🛣️ ไฟล์ `routes/api.php`**
    ใน Laravel 11 เส้นทางสำหรับทำ API จะถูกแยกออกจาก `web.php` แต่เวอร์ชันนี้ไฟล์ `api.php` จะไม่ได้แถมมาตั้งแต่ต้น! เราต้องรันคำสั่งเพื่อเปิดใช้งานมันก่อน (ข้อดีคือเบาและคลีนขึ้น)
*   **🎁 API Resources (The Transformer)**
    นี่คือฟีเจอร์ระดับเทพของ Laravel ครับ มันทำหน้าที่คั่นกลางระหว่าง Eloquent Model กับ JSON Response ทำหน้าที่ "กรอง" และ "แปลง" ข้อมูล เช่น เปลี่ยนชื่อฟิลด์จากภาษาฐานข้อมูลให้เป็นภาษา API หรือซ่อนข้อมูลความลับ (เช่น Password) ไม่ให้หลุดออกไปถึงมือ Client

{{< figure src="laravel-11-rest-api-development-diagram.jpg" alt="System flow diagram illustrating the API transformation process using API Resources" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code):
เรามาลองสร้าง API สำหรับดึงข้อมูล "สินค้า (Products)" กันครับ

**สเต็ปที่ 1: ปลุกพลัง API ใน Laravel 11**
เปิด Terminal ขึ้นมา แล้วพิมพ์คำสั่งนี้ เพื่อให้ Laravel สร้างไฟล์ `routes/api.php` ขึ้นมาให้เรา
```bash
php artisan install:api
```

**สเต็ปที่ 2: สร้าง Model, Controller และ Resource**
เราจะสร้าง Controller แบบ API (จะไม่มีเมธอด create กับ edit มาให้รกตา)
```bash
# สร้าง Model พร้อม API Controller และ Migration
php artisan make:model Product -mc --api

# สร้าง API Resource (ตัวแปลงโฉมข้อมูล)
php artisan make:resource ProductResource
```

**สเต็ปที่ 3: กำหนดเส้นทาง (Routes)**
เปิดไฟล์ `routes/api.php` แล้วใส่โค้ดนี้ครับ มันจะสร้าง Route สไตล์ RESTful ให้เราครบทุก Action
```php
use App\Http\Controllers\ProductController;
use Illuminate\Support\Facades\Route;

// สร้าง Route: /api/products ครบทั้ง GET, POST, PUT, DELETE
Route::apiResource('products', ProductController::class);
```

**สเต็ปที่ 4: แต่งตัวให้ข้อมูลด้วย API Resource**
เปิดไฟล์ `app/Http/Resources/ProductResource.php` หน้าที่ของไฟล์นี้คือการแปลง Model ไปเป็น Array ที่เราต้องการส่งออก
```php
<?php
namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class ProductResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        // เลือกเฉพาะข้อมูลที่อยากให้ Client เห็น!
        return [
            'id' => $this->id,
            'name' => $this->name,
            'price_formatted' => number_format($this->price, 2) . ' ฿', // แปลงฟอร์แมตเงินซะเลย
            'stock' => $this->stock,
            // สังเกตว่าเราไม่ส่ง created_at, updated_at ออกไปให้รก
        ];
    }
}
```

**สเต็ปที่ 5: ใช้งานใน Controller**
เปิดไฟล์ `app/Http/Controllers/ProductController.php` แล้วเรียกใช้ Resource สุดหล่อของเรา
```php
<?php
namespace App\Http\Controllers;

use App\Models\Product;
use App\Http\Resources\ProductResource;
use Illuminate\Http\Request;

class ProductController extends Controller
{
    // ดึงข้อมูลทั้งหมด (GET /api/products)
    public function index()
    {
        $products = Product::all();
        // ถ้าส่งข้อมูลเป็นกลุ่ม (Collection) ให้ใช้เมธอด collection()
        return ProductResource::collection($products);
    }

    // ดึงข้อมูลชิ้นเดียว (GET /api/products/{id})
    public function show(Product $product)
    {
        // ถ้าส่งข้อมูลชิ้นเดียว (Single Model) ให้ new ออกมาได้เลย
        return new ProductResource($product);
    }
}
```

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips):
*   **🛠️ การทดสอบด้วย Postman หรือ Insomnia:** 
    เวลาน้องๆ ยิง Request เข้าหา API ของ Laravel อย่าลืมไปที่แท็บ Headers แล้วเพิ่ม `Accept: application/json` เสมอนะครับ! ถ้าไม่ใส่ แล้วเรายิงข้อมูลผิด Laravel อาจจะเด้ง (Redirect) เรากลับไปหน้า Login แบบ HTML แทนที่จะพ่น JSON Error ออกมาครับ
*   **📦 เวทมนตร์แห่ง Data Wrapping:** 
    สังเกตไหมครับว่า พอเราใช้ API Resource ข้อมูล JSON ที่ได้ออกมาจะถูกห่อหุ้มด้วยคีย์ที่ชื่อว่า `"data": { ... }` เสมอ นี่คือความตั้งใจของ Laravel เพื่อเผื่อที่ว่างไว้ให้เราใส่พวก Meta Data อื่นๆ เช่น ข้อมูลการแบ่งหน้า (Pagination) หรือลิงก์ (Links) ในระดับเดียวกันได้ครับ
*   **⚡ เลี่ยงปัญหา N+1 ด้วย `with()`:** 
    ถ้า API Resource ของน้องมีการดึงข้อมูลตารางอื่น (Relationship) ระวังปัญหา N+1 Query นะครับ! เช่น ถ้าใน Resource เรามี `'author' => $this->author->name` ตอนเขียน Controller น้องต้องดึงข้อมูลมารอไว้ล่วงหน้าด้วย `Product::with('author')->get()` เสมอ เพื่อให้เว็บเรายังคงความเร็วแสง!

## 6. 🏁 บทสรุป (To be continued...):
เป็นอย่างไรกันบ้างครับ กับการทำ **RESTful API** ด้วย Laravel การมี **API Resources** เข้ามาช่วย ทำให้โค้ด Controller ของเราสั้นและคลีนมากๆ แถมเรายังสามารถคุมหน้าตาของ JSON ขาออกได้อย่างเบ็ดเสร็จ แยกส่วนรับผิดชอบ (Separation of Concerns) ได้อย่างสวยงามตามวิถีของสถาปนิกซอฟต์แวร์

แต่ช้าก่อน! ตอนนี้ API ของเราเปิดกว้างอ้าซ่า ใครหน้าไหนก็สามารถยิงมารูปดูข้อมูล หรือแม้แต่ลบข้อมูลของเราได้! ในตอนหน้า พี่จะพาน้องๆ ไปสวมชุดเกราะติดอาวุธให้ API ด้วยระบบรักษาความปลอดภัย **"API Authentication ด้วย Laravel Sanctum"** เตรียมตัวให้พร้อม แล้วพบกันครับ!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Automation ให้กับโรงงานของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและติดตั้งระบบแบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p