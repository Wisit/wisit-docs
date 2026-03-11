---
title: "Laravel Horizon: มอนิเตอร์ระบบ Queue แบบ Real-time"
weight: 170
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Laravel 11", "Horizon", "Queue", "Redis", "Performance"]
---

{{< figure src="laravel-11-horizon-queue-monitoring-cover.jpg" alt="A futuristic holographic dashboard representing Laravel Horizon monitoring Redis queues" >}}

## 1. 🎯 ชื่อตอน (Title): ตอนที่ 38: Laravel Horizon เบิกเนตรระบบ Queue ส่องแดชบอร์ดจัดการ Worker อัจฉริยะ

## 2. 📖 เปิดฉาก (The Hook):
มาครับน้องๆ ลากเก้าอี้มานั่งจิบกาแฟกัน... ในตอนที่แล้วพี่พาน้องๆ ไปรู้จักกับ "ระบบคิว (Queues & Jobs)" ที่เปรียบเสมือนการส่งออเดอร์ไปให้พ่อครัวหลังร้านทำ เพื่อให้หน้าร้าน (Web App) ของเราตอบสนองลูกค้าได้ไวที่สุดใช่ไหมครับ?

แต่พอระบบเราเริ่มสเกลใหญ่ขึ้น มีคิวส่งอีเมล คิวตัดรูปภาพ คิวออกรายงาน... ปัญหาใหม่ที่ Senior Dev ทุกคนต้องเจอคือ **"ความตาบอด (Blind Spot)"** ครับ! เราโยนงานไปหลังร้านก็จริง แต่เราไม่รู้เลยว่าตอนนี้พ่อครัวทำงานทันไหม? มีออเดอร์ค้างกี่คิว? งานไหนพังบ้าง? ถ้าจะดูก็ต้องไปนั่งงมอ่านไฟล์ Log หรือพิมพ์คำสั่งเช็คใน Terminal ซึ่งมันไม่ทันกินเอาเสียเลย

เพื่อแก้ปัญหานี้ สถาปนิกของ Laravel ได้สร้างเครื่องมือระดับเทพขึ้นมาตัวหนึ่งชื่อว่า **"Laravel Horizon"** ครับ มันเปรียบเสมือนกล้องวงจรปิดพร้อม AI ผู้จัดการร้าน ที่นอกจากจะให้ แดชบอร์ด (Dashboard) สวยๆ เพื่อดูสถานะทุกอย่างแบบ Real-time แล้ว มันยังสามารถ "ปรับเพิ่มลดจำนวนพ่อครัว (Auto-Balancing)" ให้สอดคล้องกับปริมาณงานที่เข้ามาได้แบบอัตโนมัติ! วันนี้พี่จะพาไปติดตั้งและคอนฟิกฮีโร่ตัวนี้กันครับ!

## 3. 🧠 แก่นวิชา (Core Concepts):
ก่อนจะเริ่มลุยโค้ด เราต้องเข้าใจกฎเหล็กและคอนเซปต์ของ Horizon กันก่อนครับ:

*   **🔴 สหายแท้แห่ง Redis:** กฎข้อแรกคือ Horizon ถูกออกแบบมาเพื่อทำงานร่วมกับ **"Redis"** เท่านั้นครับ! น้องไม่สามารถใช้ Horizon กับ Database Queue หรือ SQS ได้ เพราะมันต้องการความเร็วระดับ In-memory ของ Redis ในการคำนวณและดึงเมตริกซ์ต่างๆ
*   **⚙️ Code-driven Configuration:** การตั้งค่าจำนวน Worker และ Queue ทั้งหมดของ Horizon จะถูกเก็บไว้ในไฟล์โค้ด (`config/horizon.php`) เพียงไฟล์เดียว ทำให้เราสามารถเอาการตั้งค่าพวกนี้ขึ้น Git (Version Control) ได้เลย ทีมงานทุกคนจะได้เซ็ตติ้งที่ตรงกันเป๊ะ
*   **⚖️ Balancing Strategies (กลยุทธ์การจัดสมดุล):** นี่คือฟีเจอร์ไม้ตายครับ Horizon สามารถจัดการ Worker ได้ 3 แบบ:
    *   `simple`: แบ่งจำนวน Worker ให้แต่ละคิวเท่าๆ กัน (หารยาว)
    *   `auto`: (แนะนำ) ปรับจำนวน Worker ให้อัตโนมัติ คิวไหนงานล้นก็เกณฑ์คนไปช่วย คิวไหนว่างก็ดึงคนออก
    *   `false`: ปิดระบบจัดการ ให้รันตามลำดับปกติของ Laravel

{{< figure src="laravel-11-horizon-queue-monitoring-diagram.jpg" alt="System flow diagram illustrating Laravel Horizon's Auto-Balancing feature allocating workers based on queue load" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code):
เรามาเริ่มติดตั้งและปลุกชีพ Horizon กันเลยครับ!

**สเต็ปที่ 1: เปลี่ยนมาใช้ Redis**
ตรวจสอบให้แน่ใจว่าระบบมี Redis รันอยู่ และไปแก้ไฟล์ `.env` ให้ Queue วิ่งไปหา Redis ครับ:
```env
QUEUE_CONNECTION=redis
```

**สเต็ปที่ 2: ติดตั้งแพ็กเกจ Horizon**
เปิด Terminal แล้วรันคำสั่งดึงแพ็กเกจผ่าน Composer และสั่ง Publish ไฟล์ Assets ต่างๆ:
```bash
composer require laravel/horizon
php artisan horizon:install
```

**สเต็ปที่ 3: กำหนดกลยุทธ์ใน `config/horizon.php`**
เปิดไฟล์นี้ขึ้นมา เลื่อนลงไปล่างสุดที่หัวข้อ `environments` ครับ เราจะตั้งค่าให้ระบบ "Auto-Balance" บน Production:
```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            'connection' => 'redis',
            'queue' => ['emails', 'notifications', 'reports'], // คิวที่เราใช้งาน
            'balance' => 'auto', // เปิดระบบ AI ผู้จัดการร้าน!
            'autoScalingStrategy' => 'time', // คำนวณคิวจากเวลาที่ใช้
            'minProcesses' => 1, // จำนวน Worker ขั้นต่ำสุดต่อคิว
            'maxProcesses' => 10, // จำนวน Worker สูงสุดที่จะขยายได้
            'balanceMaxShift' => 1, // ปรับเพิ่ม/ลด ทีละ 1 คน
            'balanceCooldown' => 3, // รอ 3 วินาที ระหว่างปรับลด/เพิ่ม
            'tries' => 3,
        ],
    ],
    // ตอนพัฒนาบนเครื่องตัวเอง (Local) เอาแบบง่ายๆ พอ
    'local' => [
        'supervisor-1' => [
            'maxProcesses' => 3,
        ],
    ],
],
```

**สเต็ปที่ 4: เปิดสวิตช์เก็บสถิติ (Metrics)**
เพื่อให้กราฟในหน้า Dashboard ของเรามีข้อมูลย้อนหลัง เราต้องให้ระบบมันคอยสแนปช็อตข้อมูลทุกๆ 5 นาทีครับ โดยไปเพิ่มคำสั่งในไฟล์ `routes/console.php` สั้นๆ แบบนี้:
```php
use Illuminate\Support\Facades\Schedule;

// สั่งให้ Horizon เก็บสถิติทุกๆ 5 นาที
Schedule::command('horizon:snapshot')->everyFiveMinutes();
```

**สเต็ปที่ 5: เดินเครื่อง!**
ตอนนี้น้องๆ สามารถรัน Horizon ได้เลยด้วยคำสั่ง:
```bash
php artisan horizon
```
จากนั้นเปิดเบราว์เซอร์ไปที่ `http://localhost:8000/horizon` น้องๆ จะพบกับ Dashboard สุดล้ำที่โชว์ทุกอย่างแบบ Real-time ครับ! โคตรหล่อ!

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips):
*   **👮‍♂️ การป้องกัน Dashboard บน Production (Authorization):**
    ตอนอยู่เครื่อง Local เราเข้าหน้าเว็บ `/horizon` ได้เลย แต่พอขึ้น Production (เซิร์ฟเวอร์จริง) Laravel จะล็อคประตูนี้ไว้เพื่อความปลอดภัยครับ! ถ้าน้องอยากให้ตัวเองเข้าได้ ต้องไปแก้ "Gate" ที่ไฟล์ `app/Providers/HorizonServiceProvider.php` โดยเช็คอีเมลของแอดมินแบบนี้ครับ:
    ```php
    protected function gate(): void
    {
        Gate::define('viewHorizon', function ($user) {
            // อนุญาตเฉพาะอีเมลนี้เท่านั้น ถึงจะดู Dashboard ได้
            return in_array($user->email, [
                '[email protected]', 
            ]);
        });
    }
    ```
*   **📈 ศิลปะแห่ง Auto-Balancing (`balanceMaxShift` & `balanceCooldown`):**
    ค่า 2 ตัวนี้สำคัญมากครับ! ถ้าน้องตั้ง `balanceMaxShift` เป็น 5 หมายความว่า Horizon จะดึงพ่อครัวเข้ามาเสริมทีละ 5 คนพร้อมกันเลยเมื่อคิวพุ่งปรี๊ด ส่วน `balanceCooldown` คือ "ระยะเวลาพักเหนื่อย" สมมติเราตั้งไว้ 1 วินาที แปลว่ามันจะประเมินสถานการณ์และย้ายพ่อครัวทุกๆ 1 วินาทีครับ (ปรับให้เข้ากับความโหดของงานในระบบน้องๆ นะครับ)
*   **🚨 การ Deploy บน Server จริง ต้องใช้ Supervisor:**
    คำสั่ง `php artisan horizon` ถ้าน้องปิด Terminal มันก็จะดับไปเลยครับ! บนเซิร์ฟเวอร์จริง (เช่น Ubuntu) น้องต้องใช้โปรแกรมที่ชื่อว่า **Supervisor** เพื่อควบคุมให้คำสั่งนี้รันเป็น Background ตลอดเวลา และอย่าลืมใส่คำสั่ง `php artisan horizon:terminate` ไว้ใน Deployment Script ของน้องด้วย เพื่อให้มันรีสตาร์ทตัวเองรับโค้ดใหม่ทุกครั้งที่มีการอัปเดตระบบครับ!
*   **🗑️ การจัดการคิวขยะ:** 
    ถ้ามี Job ที่พัง (Failed Jobs) แล้วน้องไม่อยากตามแก้ น้องสามารถกดปุ่มลบจากหน้าเว็บได้เลย หรือถ้าอยากล้างบางใน Terminal ก็ใช้คำสั่ง `php artisan horizon:forget --all` หรือถ้าอยากลบงานที่กำลังค้างอยู่ในคิวทิ้งให้หมด ก็ใช้ `php artisan horizon:clear` ได้เลยครับ!

## 6. 🏁 บทสรุป (To be continued...):
**Laravel Horizon** ไม่ใช่แค่เครื่องมือวาดกราฟสวยๆ แต่มันคือ "ผู้จัดการระบบ Infrastructure" ชั้นยอด ที่ช่วยให้เราบริหารจัดการทรัพยากรเครื่องเซิร์ฟเวอร์ได้อย่างคุ้มค่าที่สุดผ่านระบบ Auto-balancing และช่วยให้ชีวิตการ Debugging ปัญหาเบื้องหลังเป็นเรื่องที่สนุกและจับต้องได้ครับ 

เมื่อเราจัดการลอจิกหลังบ้านได้อย่างหมดจดและมอนิเตอร์มันได้อย่างสวยงามแล้ว... ตอนนี้น้องๆ ก็เปรียบเสมือนสถาปนิกที่มีเครื่องมือครบมือเลยล่ะครับ! ในตอนต่อไป พี่จะพาไปเจาะลึกระบบที่ช่วย "ตรวจสุขภาพ" ของระบบทั้งหมดแบบรวบยอด ด้วยแพ็กเกจใหม่ล่าสุดอย่าง **"Laravel Pulse"** เตรียมตัวให้พร้อม แล้วพบกันครับ!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Automation ให้กับโรงงานของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและติดตั้งระบบแบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p