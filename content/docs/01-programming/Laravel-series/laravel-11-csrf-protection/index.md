---
title: "CSRF Protection: เข้าใจและป้องกันช่องโหว่ความปลอดภัยพื้นฐาน"
weight: 110
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Laravel 11", "Security", "CSRF", "Form", "Middleware"]
---

{{< figure src="laravel-11-csrf-protection-cover.jpg" alt="A futuristic high-tech vault with a glowing shield blocking malicious data packets" >}}

## 1. 🎯 ชื่อตอน (Title): ตอนที่ 24: CSRF Protection โล่พิทักษ์ภัยเงียบ ปิดประตูตายช่องโหว่คลาสสิกของชาวเว็บ

## 2. 📖 เปิดฉาก (The Hook):
มาครับน้องๆ ลากเก้าอี้มานั่งจิบกาแฟกัน... วันนี้พี่มีเรื่องสยองขวัญในวงการเว็บมาเล่าให้ฟัง ลองจินตนาการดูนะครับว่า น้องเพิ่งล็อกอินเข้าเว็บธนาคารเพื่อเช็คยอดเงิน ระหว่างนั้นน้องดันไปเปิดแท็บใหม่แล้วเผลอคลิกดู "เว็บดูรูปแมวเหมียวสุดน่ารัก" ที่เพื่อนส่งลิงก์มาให้... 

แต่หารู้ไม่ว่า ในเว็บรูปแมวนั้น แฮกเกอร์ได้แอบฝังโค้ด JavaScript ที่สั่งยิง Request ลับๆ ให้ "โอนเงิน" จากบัญชีของน้องไปเข้ากระเป๋าแฮกเกอร์! และความน่ากลัวคือ เบราว์เซอร์ของน้องจะยอมส่ง "คุกกี้ยืนยันตัวตน (Session Cookies)" ของเว็บธนาคารที่น้องเพิ่งล็อกอินค้างไว้แนบตามไปด้วย ทำให้เซิร์ฟเวอร์ธนาคารคิดว่า "อ้อ น้องเป็นคนสั่งโอนเงินเองนี่นา" และปล่อยผ่านรายการนั้นทันที... บึ้ม! เงินหายวับไปในพริบตา 💸

ช่องโหว่ระดับตำนานนี้มีชื่อว่า **"CSRF (Cross-Site Request Forgery)"** ครับ เป็นหนึ่งในภัยคุกคามพื้นฐานที่น่ากลัวมาก แต่ข่าวดีคือ ถ้าเราใช้ Laravel... กรอบงานนี้ได้เตรียมชุดเกราะสะท้อนการโจมตีนี้ไว้ให้เราเรียบร้อยแล้วตั้งแต่เกิด! วันนี้พี่จะพาไปดูว่ามันทำงานอย่างไร และใน Laravel 11 เราจะปรับแต่งมันได้อย่างไรครับ

## 3. 🧠 แก่นวิชา (Core Concepts):
กลไกการทำงานของ CSRF Protection ใน Laravel มีแนวคิดที่เรียบง่ายแต่ทรงพลังมากครับ:

*   **🛡️ กุญแจลับแห่ง Session (The CSRF Token):** 
    ทุกครั้งที่ผู้ใช้เข้ามาในแอปพลิเคชัน Laravel จะแอบสร้าง "รหัสลับ (Token)" ขึ้นมาผูกติดกับ Session ของผู้ใช้คนนั้นเสมอ รหัสนี้จะเปลี่ยนไปเรื่อยๆ และแฮกเกอร์ไม่มีทางเดาได้
*   **👮 ยามเฝ้าประตู (VerifyCsrfToken Middleware):**
    เมื่อใดก็ตามที่มีการส่งข้อมูลแบบเปลี่ยนสถานะ (เช่น `POST`, `PUT`, `PATCH`, `DELETE`) กลับมาที่เซิร์ฟเวอร์ Middleware ของ Laravel จะยืนดักรออยู่หน้าประตู และขอดู "Token" นี้เสมอ ถ้า Token ไม่ตรงกับที่ระบบจำได้ หรือไม่มีส่งมา... Laravel จะถีบ Request นั้นทิ้งทันที และพ่น Error ยอดฮิต **"419 Page Expired"** ใส่หน้าผู้ใช้!
*   **✅ ฝั่งที่ปลอดภัยเสมอ (Read-only Methods):**
    จำไว้ว่า Request ประเภท `GET`, `HEAD`, `OPTIONS` จะไม่ถูกตรวจ CSRF ครับ เพราะถือว่าเป็นแค่การ "ขอดูข้อมูล" ไม่ได้ทำให้ข้อมูลในฐานข้อมูลเปลี่ยนแปลง

{{< figure src="laravel-11-csrf-protection-diagram.jpg" alt="System flow diagram comparing an unprotected app vs a Laravel protected app requiring a CSRF token" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code):
เรามาดูวิธีการเรียกใช้โล่ป้องกันนี้ในโค้ดกันครับ บอกเลยว่าเขียนง่ายจนน่าตกใจ!

**1. การแปะยันต์กันผีในฟอร์ม HTML (Blade Template)**
เวลาน้องๆ จะสร้างฟอร์มเพื่อบันทึกข้อมูล สิ่งเดียวที่ต้องทำคือเพิ่ม Directive `@csrf` เข้าไปข้างในฟอร์มครับ
```html
<!-- ไฟล์ resources/views/profile/edit.blade.php -->
<form method="POST" action="/profile/update">
    <!-- ร่ายมนต์เรียก Token -->
    @csrf
    
    <label>ชื่อผู้ใช้:</label>
    <input type="text" name="name">
    
    <button type="submit">อัปเดตข้อมูล</button>
</form>
```
*เบื้องหลังการทำงาน:* ตอนที่ Laravel Render หน้าเว็บนี้ มันจะแปลงคำสั่ง `@csrf` ให้กลายเป็น Input ซ่อน (Hidden field) หน้าตาแบบนี้ครับ:
`<input type="hidden" name="_token" value="tRyTq2hJ...รหัสยาวเหยียด...vK9">`

**2. ข้อยกเว้นกฎเหล็ก (Excluding URIs) สไตล์ Laravel 11**
ในโลกความจริง บางครั้งเราต้องรับ Request จาก "ระบบภายนอก" (เช่น ระบบตัดบัตรเครดิต Stripe หรือระบบ Webhook ของ Line) ซึ่งระบบพวกนี้เขาไม่มีทางรู้ Token ของเราหรอกครับ 

ใน Laravel 10 เราต้องไปแก้ในไฟล์ `VerifyCsrfToken.php` แต่สำหรับ **Laravel 11 (Thinner Skeleton)** ไฟล์นั้นหายไปแล้วครับ! เราจะไปกำหนดข้อยกเว้นกันที่ศูนย์บัญชาการหลัก `bootstrap/app.php` แทน:

```php
<?php
// ไฟล์ bootstrap/app.php

use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Middleware;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        // ...
    )
    ->withMiddleware(function (Middleware $middleware) {
        
        // สั่งให้ยามเฝ้าประตู ละเว้นการตรวจ CSRF สำหรับ URL เหล่านี้
        $middleware->validateCsrfTokens(except: [
            'stripe/*',         // ยกเว้น Webhook จาก Stripe ทั้งหมด
            'api/payment-callback', // ยกเว้น URL สำหรับรับผลการจ่ายเงิน
        ]);
        
    })->create();
```

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips):
ในฐานะสถาปนิกซอฟต์แวร์ พี่มีเคล็ดวิชาขั้นสูงเกี่ยวกับการจัดการ CSRF มาฝากครับ:

*   **📡 CSRF สำหรับ AJAX / Vue / React:**
    ถ้าน้องเขียน Frontend แยกรีเควสต์ผ่าน AJAX (เช่น ใช้ Axios) น้องไม่ต้องมานั่งแนบค่า `_token` ใส่ Payload ทุกครั้งให้เมื่อยมือครับ! Laravel แอบส่งค่า Token ผ่านคุกกี้ที่ชื่อว่า `XSRF-TOKEN` ไปให้เบราว์เซอร์อัตโนมัติ และไลบรารีอย่าง Axios ก็ฉลาดพอที่จะดูดค่าจากคุกกี้นี้ ไปใส่เป็น Header ชื่อ `X-XSRF-TOKEN` ให้เราทันที ทำให้เรายิง API ภายในเว็บเดียวกันได้อย่างราบรื่น
*   **🛠️ หากไม่ได้ใช้ Axios ล่ะ?:**
    ถ้าน้องใช้ jQuery หรือ `fetch()` แบบเพียวๆ ให้สร้าง `<meta>` tag เก็บ Token ไว้ที่ `<head>` ของไฟล์ Layout ซะก่อน แล้วค่อยสั่งให้ jQuery อ่านค่าไปตั้งเป็น Header แบบ Global ครับ:
    ```html
    <meta name="csrf-token" content="{{ csrf_token() }}">
    ```
    ```javascript
    $.ajaxSetup({
        headers: { 'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content') }
    });
    ```
*   **⚠️ ระวังกับดัก `api.php`:**
    Route ที่น้องๆ เขียนในไฟล์ `routes/api.php` จะ **ไม่มี** การตรวจ CSRF Token นะครับ! เพราะกลุ่มนั้นมันถูกออกแบบมาให้เป็น Stateless (ไม่มี Session) ซึ่งมักจะป้องกันผ่าน API Token (เช่น Laravel Sanctum หรือ Passport) แทน ดังนั้นอย่าสับสนเอาฟอร์มหน้าเว็บปกติไปยิงเข้า Route กลุ่ม API เชียวล่ะ!

## 6. 🏁 บทสรุป (To be continued...):
**CSRF Protection** ไม่ใช่แค่ฟีเจอร์เก๋ๆ แต่มันคือ "มาตรฐานความปลอดภัยบังคับ" ที่แอปพลิเคชันยุคใหม่ต้องมีครับ การที่ Laravel เตรียมของพวกนี้มาให้แบบเปิดใช้งานอัตโนมัติ ช่วยเซฟชีวิตนักพัฒนาและรักษาเงินในกระเป๋าของผู้ใช้มานักต่อนักแล้ว

จำไว้เสมอนะครับน้องๆ **"อย่าลืมใส่ `@csrf` ในทุกๆ ฟอร์มที่เป็น POST, PUT, DELETE"** แค่นี้ชีวิตก็ปลอดภัยขึ้นอีกเยอะ!

เมื่อเราปกป้องฟอร์มจากการโจมตีภายนอกได้แล้ว... ในตอนต่อไป พี่จะพาไปดูอีกหนึ่งเรื่องที่โปรแกรมเมอร์มักจะปวดหัว นั่นก็คือการจัดการ **"การอัปโหลดไฟล์ (File Storage)"** เราจะรับรูปภาพจากฟอร์มมาเก็บอย่างไรให้เป็นระเบียบ และปลอดภัย? เตรียมกาแฟให้พร้อม แล้วพบกันครับ!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Automation ให้กับโรงงานของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและติดตั้งระบบแบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p