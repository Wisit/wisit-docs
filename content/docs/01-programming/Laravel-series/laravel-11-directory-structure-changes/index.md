---
title: "เจาะลึก Directory Structure ใหม่ใน Laravel 11: หายไปไหนหมด!?"
weight: 10
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Laravel 11", "Directory Structure", "Architecture", "PHP 8"]
---

{{< figure src="laravel-11-directory-structure-changes-cover.jpg" alt="Holographic folder structure morphing into a single smart tablet displaying bootstrap/app.php" >}}

## 1. 🎯 ชื่อตอน (Title): เจาะลึกโครงสร้างแบบ Thinner Skeleton... เมื่อคัมภีร์ถูกย่อให้เหลือเพียงหน้าเดียว!

## 2. 📖 เปิดฉาก (The Hook):
มาครับน้องๆ ลากเก้าอี้มานั่งจิบกาแฟกัน... พี่เชื่อว่าหลายคนที่เพิ่งอัปเกรดมาใช้ Laravel 11 แล้วรันคำสั่ง `laravel new` เป็นครั้งแรก พอเปิดโปรเจกต์ขึ้นมาใน VS Code ถึงกับต้องขยี้ตาแล้วอุทานว่า **"เห้ย! ไฟล์โปรเจกต์ฉันหายไปไหนหมด!?"** 

ถ้าเปรียบ Laravel ยุคก่อนเป็น "คฤหาสน์หลังใหญ่" ที่มีห้องว่างมากมายเตรียมไว้ให้คุณใช้งาน (เช่น โฟลเดอร์ `Middleware` ที่มีไฟล์เต็มไปหมด, ไฟล์ `Kernel.php` ทั้งฝั่ง HTTP และ Console, รวมไปถึง `Service Providers` ที่เรียงหน้ากระดานมาให้ถึง 5 ตัว) ในยุคของ Laravel 11 ทีมพัฒนาอย่าง Taylor Otwell และ Nuno Maduro ได้ทำการรีโนเวทคฤหาสน์หลังนี้ใหม่ ให้กลายเป็น "สมาร์ทคอนโด (Streamlined Application Structure)" ที่ซ่อนความซับซ้อน (Boilerplate) เอาไว้เบื้องหลัง เพื่อให้หน้าบ้านดูคลีนและทันสมัยที่สุดครับ

วันนี้พี่จะพาไปสืบสวนกันว่า ไฟล์ที่เคยรกหูรกตาเหล่านั้นมันถูกย้ายไปซ่อนไว้ที่ไหน แล้วถ้าเราอยากจะปรับแต่งการตั้งค่าแบบเจาะลึก เราจะต้องไปตามหามันที่ใด!

## 3. 🧠 แก่นวิชา (Core Concepts):
การเปลี่ยนแปลงระดับโครงสร้าง (Directory Structure) ใน Laravel 11 มีจุดประสงค์เพื่อลดจำนวนไฟล์เริ่มต้น (Default files) ให้น้อยที่สุด โดยมีไฮไลต์สำคัญดังนี้ครับ:

*   **🎛️ ศูนย์บัญชาการใหม่: `bootstrap/app.php`**
    ในอดีต เวลาเราจะเพิ่ม Middleware เราต้องไปแก้ที่ `app/Http/Kernel.php` ถ้าจะจัดการ Error ต้องไปที่ `Exceptions/Handler.php` แต่ใน Laravel 11 ไฟล์เหล่านั้นถูกลบทิ้งไปแล้วครับ! ทุกอย่างถูกรวบศูนย์มาไว้ที่ไฟล์ `bootstrap/app.php` ซึ่งกลายเป็นไฟล์ตั้งค่าแอปพลิเคชันแบบ "Code-first" ที่ใช้จัดการทั้ง Routing, Middleware และ Exception Handling ในที่เดียว
*   **👻 การหายตัวไปของ Middleware**
    โฟลเดอร์ `app/Http/Middleware` ที่เคยมีไฟล์ถึง 9 ไฟล์ (เช่น การเช็ค CSRF, การตัดช่องว่าง Whitespace) ตอนนี้พวกมันถูกย้ายเข้าไปซ่อนอยู่ในแก่นของ Framework (Core) เป็นที่เรียบร้อยแล้ว เพื่อไม่ให้โครงสร้างแอปของเราบวมโดยไม่จำเป็น
*   **🔌 บริการที่ลดรูป (Fewer Service Providers)**
    จากเดิมที่เรามีถึง 5 ตัว (เช่น `AuthServiceProvider`, `RouteServiceProvider`) ตอนนี้โครงสร้างใหม่จะเหลือเพียงแค่ `AppServiceProvider` ตัวเดียวโดดๆ เลยครับ! ฟังก์ชันต่างๆ เช่น การลงทะเบียน Event (Event Discovery) จะถูกเปิดใช้งานอัตโนมัติ ทำให้เราไม่ต้องมานั่งจับคู่ Event กับ Listener เองอีกต่อไป
*   **✂️ Opt-in Routing (อยากใช้ค่อยเรียกหา)**
    ปกติเราจะเห็นไฟล์ `routes/api.php` และ `routes/channels.php` นอนรอเราอยู่เสมอ แต่หลายโปรเจกต์ก็ไม่ได้ทำ API ใช่ไหมครับ? Laravel 11 จึงตัด 2 ไฟล์นี้ออกไปเป็นค่าเริ่มต้น หากน้องๆ ต้องการสร้าง API ก็แค่ร่ายมนต์คำสั่ง Artisan ระบบถึงจะสร้างไฟล์ขึ้นมาให้

{{< figure src="laravel-11-directory-structure-changes-diagram.jpg" alt="System flow diagram comparing Laravel 10 vs Laravel 11 directory structure centralized at bootstrap/app.php" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code):
เรามาดูหน้าตาของศูนย์บัญชาการใหม่ หรือไฟล์ `bootstrap/app.php` กันครับ สังเกตความสวยงามและ Fluent API ของมันสิครับ อ่านลื่นเหมือนอ่านเรียงความภาษาอังกฤษเลย!

```php
<?php

use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Foundation\Configuration\Middleware;
use App\Http\Middleware\EnsureUserIsSubscribed;

// สร้างแอปพลิเคชันและผูกส่วนประกอบทั้งหมดเข้าด้วยกัน
return Application::configure(basePath: dirname(__DIR__))
    
    // 1. ระบุเส้นทาง (Routing) แทนที่ RouteServiceProvider เดิม
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up', // มี Route /up เอาไว้ให้ระบบอย่าง Kubernetes เช็คว่าเว็บยังรอดอยู่ไหม
    )
    
    // 2. จัดการ Middleware ทะลุทะลวงเข้ามาที่ Core ได้เลย!
    ->withMiddleware(function (Middleware $middleware) {
        
        // ยกเว้นการตรวจ CSRF สำหรับ URL ของ Stripe
        $middleware->validateCsrfTokens(
            except: ['stripe/*']
        );
        
        // แปะ Middleware ของเราเองต่อท้ายเข้าไปในกลุ่ม 'web'
        $middleware->web(append: [
            EnsureUserIsSubscribed::class,
        ]);
    })
    
    // 3. จัดการ Error และ Exception สวยๆ จบในที่เดียว
    ->withExceptions(function (Exceptions $exceptions) {
        // เช่น ไม่ต้องส่ง Report เก็บลง Log ถ้าเป็น Error ชนิดนี้
        $exceptions->dontReport(MissedFlightException::class);
    })->create();
```

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips):
ในฐานะสถาปนิกซอฟต์แวร์ พี่มีเกร็ดความรู้เล็กๆ ที่ซ่อนอยู่ในโครงสร้างใหม่นี้มาเตือนกันครับ:
*   **หลุมพรางตอนอัปเกรด (Upgrading is Safe):** ถ้าน้องๆ มีโปรเจกต์ Laravel 10 อยู่ แล้วทำการอัปเกรดเป็น Laravel 11 **"ไม่จำเป็น"** ต้องรื้อโครงสร้างไฟล์เก่าทิ้งเพื่อมาทำแบบใหม่นะครับ! ระบบของ Laravel 11 ถูกจูนมาให้รองรับโครงสร้างแบบเก่า (Backward Compatible) ได้อย่างสมบูรณ์แบบ
*   **ไฟล์ Config หายไปไหน?:** โฟลเดอร์ `config` ใน Laravel 11 จะไม่มีไฟล์พวก `cors.php` หรือ `view.php` มาให้เห็นตั้งแต่แรก เพราะส่วนใหญ่เราแทบไม่เคยไปแตะมันเลย แต่ถ้าเกิดอยากจะปรับแต่งขึ้นมาจริงๆ ให้รันคำสั่ง `php artisan config:publish` (หรือตามด้วยชื่อไฟล์) ระบบก็จะคายไฟล์นั้นออกมาให้แก้ไขครับ
*   **ปลุกชีพ API:** เมื่อถึงเวลาที่โปรเจกต์ต้องคุยกับ Mobile App ให้รันคำสั่ง `php artisan install:api` ระบบจะทำการสร้างไฟล์ `routes/api.php` กลับมาให้ พร้อมกับเชื่อมต่อให้ใน `bootstrap/app.php` อย่างไร้รอยต่อ

## 6. 🏁 บทสรุป (To be continued...):
การเปลี่ยนแปลง Directory Structure ใน Laravel 11 คือก้าวสำคัญที่สะท้อนให้เห็นถึงปรัชญา **"Developer Happiness"** ครับ การลบความรกรุงรังของไฟล์ที่ไม่ได้ใช้งานออกไป (Minimalism) ช่วยลดอาการหน้ามืดของนักพัฒนามือใหม่ และช่วยให้มือเก๋าโฟกัสไปที่สิ่งสำคัญที่สุด นั่นคือ "การเขียนโค้ดเพื่อตอบโจทย์ทางธุรกิจ (Business Logic)" นั่นเอง

เมื่อเราเข้าใจแผนผังเมืองใหม่กันทะลุปรุโปร่งแล้ว ในบทความหน้า พี่จะพาน้องๆ ไปเจาะลึกระบบ **Controller** โฉมใหม่ ว่ามันถูกปรับปรุงให้ทำงานคู่กับ Route ได้ง่ายดายขึ้นแค่ไหน... เตรียมกาแฟให้พร้อม แล้วเจอกันครับเหล่า Artisan!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Automation ให้กับโรงงานของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและติดตั้งระบบแบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p