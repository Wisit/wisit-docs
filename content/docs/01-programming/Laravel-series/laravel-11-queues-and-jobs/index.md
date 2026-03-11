---
title: "Queues & Jobs: ทำงานหนักไว้เบื้องหลังเพื่อความลื่นไหล"
weight: 160
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Laravel 11", "Queues", "Jobs", "Performance", "Background Processing"]
---

{{< figure src="laravel-11-queues-and-jobs-cover.jpg" alt="A futuristic conveyor belt representing Laravel Queues and Jobs processing tasks in the background" >}}

## 1. 🎯 ชื่อตอน (Title): ตอนที่ 37: Queues & Jobs ปล่อยงานหนักไว้เบื้องหลัง เสกเว็บให้ลื่นไหลไร้รอยต่อ

## 2. 📖 เปิดฉาก (The Hook):
มาครับน้องๆ ลากเก้าอี้มานั่งจิบกาแฟกัน... ลองจินตนาการดูนะครับว่า น้องๆ ทำเว็บ E-Commerce เสร็จแล้ว ลูกค้ากดปุ่ม "ชำระเงิน" ปุ๊บ ระบบของเราต้องทำอะไรบ้าง? 
1. บันทึกข้อมูลลง Database 
2. ตัดบัตรเครดิตผ่าน API ธนาคาร 
3. สร้างใบเสร็จ PDF 
4. ส่งอีเมลยืนยันการสั่งซื้อ 
5. ยิงแจ้งเตือนเข้าแอป Slack ของแอดมิน

ถ้าน้องๆ เขียนโค้ดให้มันทำงานทั้งหมดนี้แบบ "ตามลำดับ (Synchronous)" ใน Request เดียว... ลูกค้าอาจจะต้องนั่งจ้องไอคอนหมุนติ้วๆ (Loading spinner) นานถึง 10 วินาที! และความน่ากลัวคือ ถ้ารอนานเกินไป ลูกค้ามักจะคิดว่าเว็บค้าง แล้วเผลอกดปุ่ม "ชำระเงิน" ซ้ำสองรอบ... บึ้ม! หายนะบังเกิดครับ! ตัดเงินลูกค้าเบิ้ลไปซะงั้น

ทางออกของสถาปนิกซอฟต์แวร์ระดับโลกคือการใช้ **"ระบบคิว (Queue System)"** ครับ งานไหนที่ใช้เวลานานและลูกค้าไม่ต้องรอผลลัพธ์เดี๋ยวนั้น (เช่น การส่งอีเมล หรือสร้าง PDF) เราจะจับมันยัดใส่กล่องที่เรียกว่า **"Job"** แล้วโยนไปไว้หลังร้าน ให้ลูกจ้างที่เรียกว่า **"Worker"** ค่อยๆ ทยอยทำตามคิว ส่วนหน้าบ้านก็ตอบกลับลูกค้าทันทีว่า "สั่งซื้อสำเร็จแล้วจ้า!" วันนี้พี่จะพาไปดูเวทมนตร์นี้ใน Laravel 11 กันครับ!

## 3. 🧠 แก่นวิชา (Core Concepts):
เปรียบเทียบระบบ Queue ให้เห็นภาพง่ายๆ เหมือน **ร้านฟาสต์ฟู้ด** ครับ:
*   **🧑‍💻 Web App (พนักงานรับออเดอร์):** รับคำสั่งซื้อจากลูกค้า โยนตั๋วออเดอร์ไปเสียบไว้ที่รางเหล็ก แล้วหันมารับออเดอร์ลูกค้าคนต่อไปทันที (ตอบสนองไวมาก)
*   **🎫 Job (ตั๋วออเดอร์):** คือคลาสที่บรรจุคำสั่งว่าต้องทำอะไรบ้าง (เช่น "อบเบอร์เกอร์ 1 ชิ้น" หรือ "ส่งอีเมลหา User ID 5")
*   **🗄️ Queue (รางเหล็กเสียบตั๋ว):** สถานที่เก็บตั๋วออเดอร์ที่รอการประมวลผล ใน Laravel เรามักจะเก็บคิวไว้ใน Database หรือ Redis 
*   **👨‍🍳 Worker (พ่อครัวหลังร้าน):** โปรแกรมที่รันวนลูป (Infinite Loop) อยู่เบื้องหลัง คอยดึงตั๋วออเดอร์ออกจากรางมาทำทีละใบ ถ้าคิวว่าง พ่อครัวก็จะยืนงีบรอ (Sleep) 

{{< figure src="laravel-11-queues-and-jobs-diagram.jpg" alt="System flow diagram showing Web App sending Jobs to a Queue, which are later processed by a Queue Worker" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code):
เรามาลองสร้างระบบส่งอีเมลต้อนรับผู้ใช้ใหม่แบบพื้นหลังกันครับ

**สเต็ปที่ 1: ตั้งค่าโกดังเก็บคิว**
ในไฟล์ `.env` ให้เราเปลี่ยน `QUEUE_CONNECTION` เป็น `database` (สำหรับโปรเจกต์ทั่วไปแค่นี้ก็เอาอยู่แล้วครับ)
```env
QUEUE_CONNECTION=database
```
*ปล. ใน Laravel 11 ตารางสำหรับเก็บคิวมักจะถูกสร้างมาให้ตั้งแต่ติดตั้งโปรเจกต์แล้ว แต่ถ้ายังไม่มี ให้รัน `php artisan make:queue-table` ตามด้วย `php artisan migrate` ครับ*

**สเต็ปที่ 2: สร้าง Job (เขียนตั๋วออเดอร์)**
ใช้คำสั่ง Artisan สุดหล่อของเราสร้างคลาส Job ขึ้นมา:
```bash
php artisan make:job SendWelcomeEmail
```

เปิดไฟล์ `app/Jobs/SendWelcomeEmail.php` ขึ้นมา สังเกตว่ามันจะมี Interface `ShouldQueue` แปะมาให้แล้ว นี่แหละคือสัญลักษณ์ที่บอกว่า "โยนฉันลงคิวได้เลย!"
```php
<?php
namespace App\Jobs;

use App\Models\User;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;
use Illuminate\Support\Facades\Mail;

class SendWelcomeEmail implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    // รับข้อมูล User เข้ามาเก็บไว้ผ่าน Constructor
    public function __construct(
        public User $user
    ) {}

    // ฟังก์ชันนี้จะถูกเรียกทำงานโดยพ่อครัว (Worker) ในภายหลัง
    public function handle(): void
    {
        // จำลองการส่งอีเมล (ซึ่งอาจใช้เวลา 2-3 วินาที)
        Mail::to($this->user->email)->send(new \App\Mail\WelcomeMail($this->user));
    }
}
```

**สเต็ปที่ 3: สั่งจ่ายงาน (Dispatching)**
ไปที่ Controller ของเรา แทนที่เราจะสั่งส่งอีเมลตรงๆ เราจะโยน Job ลงคิวแทน:
```php
public function register(Request $request)
{
    $user = User::create($request->validated());

    // ส่งงานเข้าคิว! (โค้ดบรรทัดนี้ใช้เวลาทำงานแค่เสี้ยววินาที)
    SendWelcomeEmail::dispatch($user);

    // ตอบกลับลูกค้าทันที ลูกค้าแฮปปี้!
    return response()->json(['message' => 'สมัครสมาชิกสำเร็จ!']);
}
```

**สเต็ปที่ 4: ปลุกพ่อครัวให้เริ่มทำงาน (Start the Worker)**
ถ้าเราทำแค่สเต็ปที่ 3 งานมันจะไปกองอยู่ใน Database แต่ไม่มีใครทำครับ! เราต้องเปิด Terminal อีกหน้าต่างนึง แล้วพิมพ์คำสั่งเพื่อเรียกพ่อครัวมาทำงาน:
```bash
php artisan queue:work
```
*เมื่อรันแล้ว น้องๆ จะเห็นข้อความขึ้นว่ากำลัง Processing... และ Processed... ถือว่าพ่อครัวทำงานเสร็จสิ้นครับ!*

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips):
โลกของคิวไม่ได้มีแค่ "ส่งแล้วจบ" ครับ ในระดับ Enterprise เราต้องรับมือกับ "ความล้มเหลว" ให้เป็นด้วย!

*   **💥 การจัดการงานที่พัง (Failed Jobs) และการทำ Retry:**
    สมมติเซิร์ฟเวอร์ส่งเมลล่ม ถ้าเป็นโค้ดปกติคือพังแล้วพังเลย แต่ใน Job เราสามารถกำหนด **"จำนวนครั้งที่ให้ลองใหม่ (`$tries`)"** และ **"เวลาพักเหนื่อยก่อนลองใหม่ (`$backoff`)"** ได้เลยครับ!
    ```php
    class SendWelcomeEmail implements ShouldQueue
    {
        public $tries = 3; // ให้ลองส่งใหม่ได้สูงสุด 3 ครั้ง
        
        // ให้ลองครั้งที่สองห่างจากครั้งแรก 10 วิ, และครั้งที่สามห่าง 60 วิ (Exponential Backoff)
        public $backoff =; 

        // ถ้ายื้อจนถึงที่สุด (เกิน 3 ครั้ง) แล้วยังพังอีก ฟังก์ชันนี้จะทำงาน
        public function failed(\Throwable $exception)
        {
            // อาจจะเขียนโค้ดเพื่อแจ้งเตือนแอดมินผ่าน Slack ว่าเมลระบบมีปัญหา!
            \Log::error('ส่งเมลไม่ผ่านจ้า: ' . $exception->getMessage());
        }
    }
    ```
*   **🛠️ เครื่องมือสืบสวน (Artisan Commands for Queues):**
    ถืองานไหนพังจนหมดโควต้า มันจะถูกเด้งไปเก็บในตาราง `failed_jobs` ครับ เราสามารถสืบสวนและสั่งรันใหม่ได้ด้วยคำสั่ง:
    *   `php artisan queue:failed` (ดูรายการงานที่พังทั้งหมด)
    *   `php artisan queue:retry all` (สั่งเอาที่พังทั้งหมด กลับไปต่อคิวทำใหม่!)
    *   `php artisan queue:flush` (ล้างบางงานที่พังทิ้งให้หมด)
*   **⚠️ คำเตือนบน Production (Supervisor):**
    คำสั่ง `php artisan queue:work` มันเป็นโปรแกรมที่รันค้างไว้ (Long-lived process) ถ้าน้องปิด Terminal หรือเซิร์ฟเวอร์รีสตาร์ท พ่อครัวก็ตายครับ! บน Server จริง (Production) เราบังคับให้ต้องใช้โปรแกรมที่ชื่อว่า **Supervisor** (หรือ Systemd) มาช่วยรันและคอยปลุก Worker ให้ตื่นขึ้นมาทำงานตลอดเวลา ห้ามลืมเด็ดขาด! และทุกครั้งที่น้องๆ อัปเดตโค้ด (Deploy) ต้องรัน `php artisan queue:restart` เพื่อให้พ่อครัวโหลดโค้ดใหม่ด้วยนะครับ!

## 6. 🏁 บทสรุป (To be continued...):
เห็นพลังของ **Queues & Jobs** ไหมครับน้องๆ? มันคือสถาปัตยกรรมกุญแจสำคัญ (Key Architecture) ที่ช่วยให้แอปพลิเคชันของเรารองรับคนใช้งานหลักแสนหลักล้านคนได้ (Scalability) โดยที่เซิร์ฟเวอร์ไม่น็อก และลูกค้าก็ได้รับประสบการณ์ที่รวดเร็วทันใจ

เมื่อเราสามารถแยกส่วนการทำงานที่หนักหน่วงออกไปไว้เบื้องหลังได้แล้ว... ในบทความต่อไป พี่จะพาไปดูอีกหนึ่งฟีเจอร์ที่เกิดมาคู่กับระบบแบบนี้ นั่นก็คือ **"Events & Listeners"** สถาปัตยกรรมการส่งสัญญาณวิทยุภายในแอปพลิเคชัน ที่จะทำให้โค้ดของเราหลุดจากการพึ่งพากัน (Decoupled) ได้อย่างสมบูรณ์แบบ... เตรียมตัวให้พร้อม แล้วพบกันครับ!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Automation ให้กับโรงงานของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและติดตั้งระบบแบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p