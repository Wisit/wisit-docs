---
title: "Middleware: ด่านตรวจคนเข้าเมืองของ Request"
weight: 30
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Laravel 11", "Middleware", "Security", "Architecture"]
---

{{< figure src="laravel-11-middleware-cover.jpg" alt="A futuristic cyberpunk immigration checkpoint representing Middleware" >}}

## 1. 🎯 ชื่อตอน (Title): Middleware ด่านตรวจคนเข้าเมืองสุดโหด สกรีน Request ก่อนทะลวงถึง Core

## 2. 📖 เปิดฉาก (The Hook):
มาครับน้องๆ วันนี้พี่จะพาไปรู้จักกับ "ผู้คุมกฎ" ประจำแอปพลิเคชันของเรา... ลองจินตนาการดูว่า เว็บไซต์ของน้องๆ คือ "ประเทศ" ประเทศหนึ่ง และ Controller ที่เก็บข้อมูลความลับของระบบ (เช่น หน้า Admin Dashboard) คือ "เขต VIP หวงห้าม" 

คำถามคือ... ถ้าน้องปล่อยให้ใครก็ได้ (Request) พิมพ์ URL แล้วเดินดุ่มๆ เข้าไปถึง Controller เลย มันจะเกิดอะไรขึ้น? ระบบพัง ความลับรั่วไหลแน่นอนครับ! ในโลกความเป็นจริง เราต้องมี "ด่านตรวจคนเข้าเมือง (Immigration Checkpoint)" คอยขอดูพาสปอร์ตก่อนว่า "คุณเป็นใคร? (Authentication)" และ "คุณมีวีซ่าเข้าเขตนี้ไหม? (Authorization)" 

ในสถาปัตยกรรมของ Laravel ด่านตรวจที่ว่านี้มีชื่อว่า **"Middleware"** ครับ มันคือชั้นเลเยอร์ที่คอยห่อหุ้มแอปพลิเคชันของเราไว้ ทุกๆ Request ที่วิ่งเข้ามาจะต้องเดินผ่านด่านนี้ และวันนี้พี่จะพาไปดูว่าใน Laravel 11 การสร้างและตั้งค่าด่านตรวจนี้มันเปลี่ยนไปจนง่ายดายขนาดไหน!

## 3. 🧠 แก่นวิชา (Core Concepts):
สถาปัตยกรรม Middleware ใน Laravel มีแนวคิดที่ทรงพลังและสละสลวยมากครับ ลองมาดูแก่นของมันกัน:

*   **🧅 ทฤษฎีหัวหอม (The Onion Layer):** 
    มองแอปพลิเคชันเป็นหัวหอมครับ Core ของแอป (Controller/Business Logic) อยู่ตรงกลาง ส่วน Middleware คือเปลือกหัวหอมแต่ละชั้น Request ต้องทะลวงผ่านเปลือก (เช่น ชั้นเช็ค Session, ชั้นเช็ค CSRF, ชั้นเช็ค Role) เข้าไปหา Core และเมื่อ Core ประมวลผลเสร็จ Response ก็จะต้องวิ่งย้อนกลับผ่านเปลือกเหล่านี้ออกมาเช่นกัน
*   **🚦 หน้าที่หลัก:** 
    1. **ตรวจสอบและปฏิเสธ:** ถ้า Request ไม่ผ่านเงื่อนไข (เช่น ยังไม่ได้ล็อกอิน) Middleware จะเตะผู้ใช้กลับไปหน้า Login ทันที
    2. **ปรับแต่งข้อมูล:** สามารถแทรกหรือแปลงข้อมูลใน Request ขาเข้า หรือแอบเติม HTTP Headers ใน Response ขากลับได้
*   **✨ ความเปลี่ยนแปลงใน Laravel 11 (No more Kernel!):**
    ในเวอร์ชันก่อนๆ เราต้องไปนั่งจดทะเบียน Middleware ลงในไฟล์ `app/Http/Kernel.php` ซึ่งยาวเป็นหางว่าว แต่ใน Laravel 11 ไฟล์นั้นถูกอุ้มหายไปแล้วครับ! การลงทะเบียนทั้งหมดถูกยุบรวมมาไว้ที่ไฟล์ **`bootstrap/app.php`** ที่เดียว ทำให้โค้ดของเราคลีนขึ้นแบบสุดๆ (Thinner Skeleton)

{{< figure src="laravel-11-middleware-diagram.jpg" alt="System flow diagram showing an incoming Request passing through Middleware shields before reaching the Controller" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code):
เรามาลองสร้างด่านตรวจคนเข้าเมืองสำหรับ "แอดมิน (Admin)" กันครับ สมมติว่าใครจะเข้าหน้า Dashboard ได้ ต้องมี Role เป็น 'admin' เท่านั้น!

**ขั้นตอนที่ 1: สร้าง Middleware ผ่าน Artisan Console**
```bash
php artisan make:middleware CheckAdminRole
```
*คำสั่งนี้จะเสกไฟล์ขึ้นมาที่ `app/Http/Middleware/CheckAdminRole.php`*

**ขั้นตอนที่ 2: เขียนกฎหมายในด่านตรวจ (The Logic)**
```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class CheckAdminRole
{
    // เมธอด handle คือหัวใจของด่านตรวจ รับ Request เข้ามา และมี Closure $next ไว้ส่งไม้ต่อ
    public function handle(Request $request, Closure $next): Response
    {
        // 1. ตรวจสอบว่า "ล็อกอินหรือยัง?" และ "มี Role เป็น admin ไหม?"
        if ($request->user() && $request->user()->role === 'admin') {
            // ถ้าใช่! เชิญผ่านด่านไปได้เลย (ส่ง Request ต่อไปให้ Controller)
            return $next($request);
        }

        // 2. ถ้าไม่ใช่! เตะกลับไปหน้า Home พร้อมคำเตือน
        abort(403, 'Unauthorized action. เฉพาะแอดมินเท่านั้นนะน้อง!');
    }
}
```

**ขั้นตอนที่ 3: ขึ้นทะเบียนด่านตรวจใน `bootstrap/app.php` (สไตล์ Laravel 11)**
เพื่อให้ Router รู้จักด่านตรวจนี้ เราต้องตั้ง "ชื่อเล่น (Alias)" ให้มันก่อนครับ
```php
<?php

use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Middleware;
use App\Http\Middleware\CheckAdminRole; // ดึงคลาสเข้ามา

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        // ...
    )
    ->withMiddleware(function (Middleware $middleware) {
        
        // ลงทะเบียน Alias ให้เรียกใช้ง่ายๆ ว่า 'admin'
        $middleware->alias([
            'admin' => CheckAdminRole::class,
        ]);
        
    })->create();
```

**ขั้นตอนที่ 4: นำด่านตรวจไปแปะกั้นถนนใน `routes/web.php`**
```php
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\AdminController;

// กลุ่ม URL ที่ถูกคุ้มครองด้วย Middleware 'auth' (ต้องล็อกอิน) และ 'admin' (ต้องเป็นแอดมิน)
Route::middleware(['auth', 'admin'])->group(function () {
    
    // ใครหลุดเข้ามาถึงตรงนี้ได้ แปลว่าเป็นแอดมินตัวจริง!
    Route::get('/admin/dashboard', [AdminController::class, 'dashboard']);
    
});
```

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips):
ในฐานะสถาปนิกซอฟต์แวร์ พี่มีเคล็ดวิชาขั้นสูงเกี่ยวกับการจัดการ Middleware มาฝากครับ:

*   **⏳ Before & After Middleware:** อย่างที่บอกไปตอนต้น Middleware ทำงานได้ 2 จังหวะ ถ้าเราเขียนโค้ด *ก่อน* เรียก `$next($request)` มันคือการกรองขาเข้า แต่ถ้าเราเก็บค่าตัวแปร `$response = $next($request);` ไว้ก่อน แล้วมาเขียนโค้ดแทรกด้านล่าง เราจะสามารถดัดแปลงข้อมูล "ขาออก (Response)" ก่อนถึงมือ User ได้ เช่น การเติม CORS Headers หรือ Security Headers
*   **🛠️ Terminable Middleware (ไม้ตายทำงานเบื้องหลัง):** บางครั้งเราอยากเก็บ Log ว่า User เข้าเว็บหน้าไหนบ้าง แต่การเก็บ Log อาจทำให้เว็บช้าลงนิดนึง Laravel มีเมธอดที่ชื่อว่า `terminate($request, $response)` ให้ใช้ในคลาส Middleware ได้ มันจะถูกรัน *หลังจาก* ที่ส่งหน้าเว็บให้ User ดูไปเรียบร้อยแล้ว (เฉพาะฝั่ง Server ที่ใช้ FastCGI) ทำให้เว็บเร็วจี๊ดจ๊าด ไม่ต้องรอเซฟ Log!
*   **Controller Middleware:** นอกจากจะไปแปะใน Route แล้ว เราสามารถไปสั่งรัน Middleware ใน `__construct()` ของ Controller ได้ด้วยนะ (ใน Laravel 11 จะใช้อินเทอร์เฟซ `HasMiddleware`) ช่วยให้โค้ดดูเป็นสัดส่วนมากขึ้นสำหรับโปรเจกต์ใหญ่ๆ

## 6. 🏁 บทสรุป (To be continued...):
เห็นไหมครับว่า **Middleware** ไม่ใช่เรื่องน่ากลัว แต่มันคือ "ศิลปะของการแบ่งแยกหน้าที่ (Separation of Concerns)" สถาปัตยกรรมที่ดี จะไม่เอาโค้ดตรวจสอบสิทธิ์ หรือโค้ดเช็ค Session ไปยัดไว้ใน Controller เด็ดขาด เราจะยกหน้าที่นี้ให้ด่านตรวจคนเข้าเมืองจัดการ เพื่อให้ Controller ของเรามีหน้าที่แค่การจัดเตรียมข้อมูลจริงๆ (Business Logic) เท่านั้น 

เมื่อด่านตรวจของเราแข็งแกร่งแล้ว ในตอนหน้า พี่จะพาข้ามกำแพงเข้าไปดูหัวใจหลักที่ทำหน้าที่คุยกับฐานข้อมูล นั่นคือระบบ **"Eloquent ORM"** ว่ามันจะเนรมิตตารางฐานข้อมูลที่ซับซ้อนให้กลายเป็น Object-Oriented ที่สวยงามได้อย่างไร... เตรียมกาแฟแก้วใหม่ให้พร้อม แล้วเจอกันครับ!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Automation ให้กับโรงงานของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและติดตั้งระบบแบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p