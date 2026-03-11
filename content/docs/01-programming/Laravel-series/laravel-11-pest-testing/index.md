---
title: "Basic Testing: เริ่มต้นเขียน Test ด้วย Pest"
weight: 190
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Laravel 11", "Pest", "Testing", "Unit Test", "Feature Test"]
---

{{< figure src="laravel-11-pest-testing-cover.jpg" alt="A futuristic developer desk with a glowing shield protecting code, representing Pest Testing" >}}

## 1. 🎯 ชื่อตอน (Title): ตอนที่ 32: ร่ายมนต์กันบั๊กด้วย Pest! เปลี่ยนการเขียน Test ให้สนุกและอ่านง่ายเหมือนอ่านนิยาย

## 2. 📖 เปิดฉาก (The Hook):
มาครับน้องๆ ลากเก้าอี้มานั่งจิบกาแฟกัน... พี่เชื่อว่าโปรแกรมเมอร์ทุกคนต้องเคยผ่าน "เย็นวันศุกร์สุดสยอง" มาก่อน วันที่เราแค่แก้โค้ดนิดเดียว (Minor update) แล้วกด Deploy ขึ้น Production ด้วยความมั่นใจ แต่พอถึงบ้านปุ๊บ ลูกค้าโทรตามว่า "ระบบล็อกอินพังครับพี่!" 

การเขียนโค้ดโดยไม่มี **Automated Tests** ก็เหมือนกับการกระโดดร่มโดยที่เราไม่ได้เป็นคนพับร่มเองนั่นแหละครับ เราได้แต่ "หวังและภาวนา" ว่ามันจะกางออก! 

ในอดีต การเขียน Test ใน PHP (ด้วย PHPUnit) อาจจะดูเป็นเรื่องน่าเบื่อ โค้ดยาวเยื้อย ต้องสร้าง Class ต้องตั้งชื่อฟังก์ชันยาวๆ แต่ในยุคของ Laravel 11 เรามีพระเอกตัวใหม่ที่ชื่อว่า **"Pest"** ซึ่งเป็น Testing Framework ที่มาแรงที่สุดในตอนนี้ มันเปลี่ยนไวยากรณ์การเทสต์ที่แข็งทื่อ ให้กลายเป็นภาษาพูดที่อ่านลื่นไหลเหมือนนิยาย วันนี้พี่จะพาไปเปิดโลกการทำ Automated Testing รับรองว่าน้องๆ จะหลงรักการเขียน Test แน่นอนครับ!

## 3. 🧠 แก่นวิชา (Core Concepts):
ก่อนจะไปลุยโค้ด เรามาทำความเข้าใจประเภทของการ Test ที่เราต้องใช้ในชีวิตประจำวันกันก่อนครับ:

*   **🔬 Unit Test (ทดสอบอะไหล่ทีละชิ้น):** 
    คือการทดสอบโค้ดชิ้นเล็กๆ ที่แยกตัวเป็นอิสระ (Isolated) เช่น ทดสอบฟังก์ชันคำนวณภาษี หรือฟังก์ชันตัดคำ การเทสต์แบบนี้จะไม่ยุ่งกับ Database ไม่ยุ่งกับ Network ทำให้มันทำงานได้ "เร็วปรู๊ดปร๊าด" ระดับมิลลิวินาที
*   **🚀 Feature Test (ทดสอบเครื่องยนต์ทั้งคัน):** 
    คือการจำลองว่ามี User ใช้งานระบบจริงๆ เช่น ทดสอบการส่ง HTTP Request ยิงเข้ามาที่ Route -> ผ่าน Controller -> บันทึกลง Database -> แล้วดูว่าตอบกลับมาเป็น JSON หรือหน้าเว็บที่ถูกต้องไหม การเทสต์แบบนี้จะช้ากว่านิดหน่อย แต่ให้ความมั่นใจสูงสุดว่าระบบทำงานได้จริง
*   **🐛 Pest คืออะไร?:** 
    Pest ถูกสร้างครอบทับ PHPUnit อีกทีครับ สิ่งที่มันทำคือตัดความรกรุงรังของคลาส (Boilerplate) ทิ้งไป แล้วให้เราเขียนเทสต์ด้วยฟังก์ชัน `test()` หรือ `it()` คู่กับวิธีการตรวจสอบ (Assertion) สุดหล่ออย่าง `expect()`

{{< figure src="laravel-11-pest-testing-diagram.jpg" alt="System flow diagram comparing Unit Test (single gear) and Feature Test (full engine)" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code):
ใน Laravel 11 หากตอนสร้างโปรเจกต์น้องๆ ใส่ธง `--pest` มันจะติดตั้งมาให้เรียบร้อยแล้ว (หรือสามารถลงเพิ่มได้) เรามาเริ่มเขียน Test กันเลยครับ!

**1. สัมผัสแรกกับ Pest (Unit Test)**
ใช้คำสั่งสร้างไฟล์ Test แบบ Unit:
```bash
php artisan make:test CalculatorTest --unit --pest
```
เปิดไฟล์ `tests/Unit/CalculatorTest.php` ขึ้นมา แล้วดูความสวยงามของ Pest สิครับ เราไม่ต้องมี Class ด้วยซ้ำ!

```php
<?php

// สร้างฟังก์ชันจำลองสมมติว่านี่คือ Business Logic ของเรา
function calculateVat(int $amount): float {
    return $amount * 0.07;
}

// เริ่มต้นเขียน Test ด้วยฟังก์ชัน test()
test('it can calculate 7% VAT correctly', function () {
    
    // 1. Arrange (เตรียมข้อมูล)
    $price = 100;
    
    // 2. Act (กระทำ)
    $vat = calculateVat($price);
    
    // 3. Assert (ตรวจสอบผลลัพธ์)
    // อ่านแบบภาษาคน: "คาดหวังว่า vat จะต้องมีค่าเท่ากับ 7"
    expect($vat)->toBe(7.0); 

});
```

**2. การจำลอง HTTP Request (Feature Test)**
ทีนี้มาลองของใหญ่ขึ้นครับ เราจะจำลองการเข้าหน้าเว็บ หรือยิง API
```bash
php artisan make:test UserApiTest --pest
```
เปิดไฟล์ `tests/Feature/UserApiTest.php`:

```php
<?php

use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;

// เรียกใช้ Trait เพื่อบอกให้ Laravel ล้าง Database ใหม่ทุกครั้งที่รัน Test ข้อนี้
uses(RefreshDatabase::class);

it('returns a successful response from the home page', function () {
    // จำลองการเข้า URL '/'
    $response = $this->get('/');

    // ตรวจสอบว่า HTTP Status คือ 200 (OK)
    $response->assertStatus(200);
});

test('a user can see their profile', function () {
    // 1. สร้าง User จำลองขึ้นมาใน Database
    $user = User::factory()->create([
        'name' => 'พี่วิสิทธิ์'
    ]);

    // 2. จำลองการล็อกอิน (actingAs) แล้วยิง Request ไปที่หน้าโปรไฟล์
    $response = $this->actingAs($user)->get('/profile');

    // 3. ตรวจสอบว่าหน้าเว็บต้องมีคำว่า 'พี่วิสิทธิ์' ปรากฏอยู่
    $response->assertSee('พี่วิสิทธิ์');
});
```
*คอมเมนต์: เห็นไหมครับว่า `test()` กับ `it()` ทำงานเหมือนกันเป๊ะ แค่ `it()` จะช่วยให้เราตั้งชื่อประโยคภาษาอังกฤษได้ไหลลื่นกว่า (เช่น it returns a successful response)*

**3. การสั่งรัน Test**
เปิด Terminal แล้วพิมพ์คำสั่งสุดเท่นี้ได้เลย:
```bash
php artisan test
```
ถ้าทุกอย่างถูกต้อง หน้าจอจะปรากฏกล่องข้อความสีเขียวพร้อมเครื่องหมายถูก (✓) รัวๆ เป็นความรู้สึกที่ฟินสุดๆ สำหรับโปรแกรมเมอร์ครับ!

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips):
*   **🧹 คาถาล้างบาง `RefreshDatabase`:** 
    จำไว้เป็นกฎเหล็กเลยครับ! ถ้า Feature Test ของน้องมีการแตะต้องฐานข้อมูล (เช่น ใช้ Factory หรือ Insert ข้อมูล) น้อง **ต้อง** เรียกใช้ `uses(RefreshDatabase::class);` เสมอ เพื่อให้มันจำลอง Transaction และล้างข้อมูลทิ้งหลังเทสต์เสร็จ ไม่งั้นฐานข้อมูลหลักของน้องจะพังเละเทะ!
*   **📊 โค้ดของเราปลอดภัยแค่ไหน? (Test Coverage):**
    อยากรู้ไหมว่าโค้ดที่เราเขียนไป มี Test รองรับกี่เปอร์เซ็นต์? เพียงแค่น้องรัน `php artisan test --coverage` ระบบจะสรุปเป็น % มาให้เลย (ต้องติดตั้ง Xdebug หรือ PCOV ใน PHP ก่อนนะ) กฎของ Senior คือ ไม่ต้องบ้าคลั่งเอา 100% หรอกครับ เอาแค่ Business Logic สำคัญๆ ให้ได้ 80% ก็หรูแล้ว!
*   **⏩ ทริครันเทสต์แบบติดจรวด (Parallel Testing):**
    เมื่อโปรเจกต์น้องมี Test เป็นพันๆ ข้อ การรันปกติอาจใช้เวลาหลายนาที ใน Laravel 11 น้องสามารถสั่ง `php artisan test --parallel` เพื่อกระจายการทำงานให้รันพร้อมกันตามจำนวน Core ของ CPU ในเครื่อง น้องจะพบว่ามันเร็วขึ้นแบบก้าวกระโดด!

## 6. 🏁 บทสรุป (To be continued...):
การทำ **Automated Testing** ไม่ใช่การเสียเวลาครับ แต่มันคือ "การลงทุนซื้อความสบายใจ" ยิ่งเราเขียน Test ควบคู่กับการพัฒนาไปด้วย (แนวคิดแบบ TDD - Test-Driven Development) โค้ดของเราจะมีโครงสร้างที่ดีขึ้น (Clean Architecture) และกล้าที่จะ Refactor โค้ดเก่าๆ โดยไม่ต้องกลัวว่ามันจะระเบิดคามือ

เมื่อเรามั่นใจในความแข็งแกร่งของระบบแล้ว... ในตอนหน้า พี่จะพาก้าวข้ามจากการทำแค่ API ไปสู่การสร้าง "หน้าบ้าน (Frontend)" แบบล้ำๆ ด้วยการใช้ **"Inertia.js ร่วมกับ Vue หรือ React"** โดยที่เราไม่ต้องไปเหนื่อยทำ API ด้วยซ้ำ! มันจะว้าวขนาดไหน เตรียมกาแฟให้พร้อม แล้วพบกันครับ!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Automation ให้กับโรงงานของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและติดตั้งระบบแบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p