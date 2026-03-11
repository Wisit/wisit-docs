---
title: "Concurrency ใน Laravel 11: รันงานหลายอย่างพร้อมกัน"
weight: 220
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Laravel 11", "Concurrency", "Performance", "Parallel", "Optimization"]
---

{{< figure src="laravel-11-concurrency-cover.jpg" alt="A futuristic developer desk showing a single data beam splitting into multiple parallel high-speed beams, representing Laravel Concurrency" >}}

## 1. 🎯 ชื่อตอน (Title): ตอนที่ 44: Concurrency ทลายขีดจำกัด PHP รันงานคู่ขนานเพื่อความเร็วทะลุนรก!

## 2. 📖 เปิดฉาก (The Hook):
มาครับน้องๆ ลากเก้าอี้มานั่งจิบกาแฟกัน... น้องๆ เคยทำหน้า Dashboard สำหรับผู้บริหารกันไหมครับ? หน้าเว็บที่พอเปิดมาปุ๊บ ต้องแสดงทั้ง "ยอดขายรวม", "จำนวนลูกค้าใหม่", และ "ดึงข้อมูลเรตติ้งจาก API ภายนอก" 

โดยธรรมชาติของภาษา PHP มันทำงานแบบ **"Single-threaded"** หรือทำงานได้ทีละคำสั่ง (Blocking) สมมติว่า:
- นับยอดขายใน Database ใช้เวลา 1 วินาที
- นับลูกค้าใหม่ ใช้เวลา 1 วินาที
- ยิง API ไปหาเว็บนอก ใช้เวลา 2 วินาที

ถ้าน้องเขียนโค้ดเรียงต่อกันไปเรื่อยๆ (Sequential) ผู้บริหารจะต้องนั่งรอหน้าจอหมุนติ้วๆ นานถึง **4 วินาที** กว่าเว็บจะโหลดเสร็จ! 

เพื่อแก้ปัญหานี้ สถาปนิกของ Laravel 11 ได้ปล่อยฟีเจอร์ระดับเวทมนตร์ตัวใหม่ (ปัจจุบันยังอยู่ในช่วง Beta) ที่ชื่อว่า **"Concurrency Facade"** ออกมาครับ! แนวคิดคือ "ในเมื่อเรามีงาน 3 อย่าง แทนที่จะให้พ่อครัวคนเดียวทำทีละจาน ทำไมเราไม่จ้างพ่อครัว 3 คนมาทำอาหารพร้อมกันซะเลยล่ะ?" วันนี้พี่จะพาไปดูวิธีเสกให้งานทั้ง 3 อย่างเสร็จภายใน 2 วินาทีกันครับ!

## 3. 🧠 แก่นวิชา (Core Concepts):
ระบบ Concurrency ของ Laravel 11 ทำงานโดยการแพ็กโค้ดของเรา (Serialize) แล้วส่งไปรันเป็น Background Process ที่ซ่อนอยู่เบื้องหลัง เมื่อประมวลผลเสร็จ มันจะส่งผลลัพธ์กลับมารวมกันให้เราครับ โดยมี "คนขับรถ (Drivers)" ให้เราเลือกใช้ 3 แบบ:

*   **⚙️ `process` (ค่าเริ่มต้น):** 
    ทำงานโดยการสั่งรัน Artisan Command แบบซ่อนรูปใน Background ใช้ได้กับทุกสภาพแวดล้อมโดยไม่ต้องลงอะไรเพิ่ม
*   **🚀 `fork` (เร็วที่สุด แต่มีข้อจำกัด):** 
    ใช้พลังของการทำ Process Forking ผ่านแพ็กเกจ `spatie/fork` (ต้องติดตั้งเพิ่ม) ตัวนี้ทำงานได้เร็วกว่ามาก แต่ **ใช้ได้เฉพาะเมื่อรันผ่าน CLI (Command Line)** เท่านั้นครับ (เช่น ใช้ใน Artisan Command หรือ Queue Job) PHP ไม่รองรับการ Fork ระหว่างที่รับ HTTP Request ปกติ
*   **🧪 `sync` (สำหรับทดสอบ):** 
    เอาไว้ใช้ตอนเขียน Automated Test ระบบจะรันโค้ดแบบเรียงคิวตามปกติ เพื่อให้เราตรวจสอบผลลัพธ์ได้ง่ายๆ

{{< figure src="laravel-11-concurrency-diagram.jpg" alt="System flow diagram comparing Sequential execution (stacked timeline) vs Concurrent execution (parallel timelines)" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code):
เรามาดูวิธีการใช้งานจริงกันครับ โค้ดนั้นสั้นและหล่อมาก!

### 🔥 พาร์ท A: การรันงานคู่ขนาน (Running Concurrent Tasks)
เราจะใช้คำสั่ง `Concurrency::run()` โดยส่ง Array ของฟังก์ชัน (Closures) เข้าไป และรับผลลัพธ์กลับมาด้วยเทคนิค Array Destructuring ของ PHP

```php
<?php
namespace App\Http\Controllers;

use Illuminate\Support\Facades\Concurrency;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Http;

class DashboardController extends Controller
{
    public function index()
    {
        // ร่ายมนต์! สั่งให้ 3 งานนี้วิ่งไปทำพร้อมกันเลย!
        [$revenue, $newUsers, $externalData] = Concurrency::run([
            
            // พ่อครัวคนที่ 1: ไปนับยอดขายมา
            fn () => DB::table('orders')->where('status', 'paid')->sum('amount'),
            
            // พ่อครัวคนที่ 2: ไปนับจำนวนลูกค้ามา
            fn () => DB::table('users')->where('created_at', '>=', now()->subMonth())->count(),
            
            // พ่อครัวคนที่ 3: ไปคุยกับ API ภายนอกมา
            fn () => Http::get('https://api.external.com/stats')->json(),
            
        ]);

        // นำผลลัพธ์ที่เสร็จพร้อมกันมาส่งให้ View
        return view('dashboard', [
            'revenue' => $revenue,
            'newUsers' => $newUsers,
            'externalData' => $externalData,
        ]);
    }
}
```

### 🏃‍♂️ พาร์ท B: การผลัดงานไปทำทีหลัง (Deferring Tasks)
บางครั้งเรามีงานที่ต้องทำพร้อมกันหลายอย่าง แต่ **"เราไม่ต้องรอผลลัพธ์"** (เช่น การส่งสถิติไปเก็บ หรือยิงแจ้งเตือน) เราอยากให้หน้าเว็บโหลดเสร็จส่งหาลูกค้าทันที แล้วค่อยให้ Server แอบไปทำต่อเบื้องหลัง เราสามารถใช้ `Concurrency::defer()` ได้ครับ

```php
use App\Services\Metrics;
use Illuminate\Support\Facades\Concurrency;

public function store(Request $request)
{
    // ... บันทึกข้อมูลหลักเสร็จแล้ว ...

    // บอกให้ Laravel ตอบกลับ Response ไปก่อนเลย 
    // แล้วค่อยรัน 2 งานนี้แบบขนานกันในภายหลัง (Background)
    Concurrency::defer([
        fn () => Metrics::report('users'),
        fn () => Metrics::report('orders'),
    ]);

    return response()->json(['message' => 'บันทึกสำเร็จ หน้าเว็บโหลดเสร็จแล้วจ้า!']);
}
```

## 5. 📊 ตารางเปรียบเทียบความเร็ว (Performance Comparison)
เพื่อให้เห็นภาพชัดเจน พี่ขอจำลองการเปรียบเทียบระหว่างการเขียนโค้ดแบบเก่า (Sequential) กับแบบใหม่ (Concurrent) โดยอิงจากงานที่ต้องรอดึงข้อมูล (I/O Bound) เช่น การดึง API 3 แหล่ง ที่ใช้เวลาแหล่งละ ~1 วินาที:

| รูปแบบการเขียน | การทำงาน (3 Tasks) | เวลาที่ใช้รวม (Total Time) | การใช้ CPU |
| :--- | :--- | :--- | :--- |
| **Traditional (Sequential)** | ทำ Task 1 (1s) -> รอเสร็จ -> ทำ Task 2 (1s) -> รอเสร็จ -> ทำ Task 3 (1s) | **~3.0 วินาที** (อืดเป็นเต่า) | ต่ำ (ทำงานทีละ 1 แกน) |
| **Laravel Concurrency** | ทำ Task 1, 2, 3 พร้อมกันในเวลาเดียวกัน | **~1.1 วินาที** (เร็วติดจรวด!) | สูงขึ้นชั่วขณะ (กระจายงานไปหลาย Process) |

*ข้อสังเกต: เวลารวมของการทำ Concurrency จะเท่ากับ "งานที่ใช้เวลาทำงานนานที่สุด" ในกลุ่มนั้นครับ!*

## 6. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips):
*   **⚠️ ข้อควรระวังเรื่อง Serialization:** 
    เพราะ Laravel อาศัยการแปลงโค้ด (Serialize) ส่งข้าม Process ถ้าน้องๆ พยายามส่งตัวแปรที่มีความซับซ้อนมากๆ (เช่น Object ที่ผูกกับ Database Connection หรือ File Resource) เข้าไปในฟังก์ชัน `fn ()` ระบบอาจจะพังได้ครับ ให้ส่งเฉพาะค่าพื้นฐาน (Primitive data) เช่น String หรือ Array เข้าไปจะปลอดภัยที่สุด
*   **🛠️ อยากแรงต้องใช้ `fork` (สำหรับ CLI):**
    ถ้าน้องเขียน Artisan Command ไว้ดึงข้อมูลเป็นแสนๆ เรคคอร์ด ลองเปลี่ยนมาใช้ Driver `fork` ดูครับ โดยลงแพ็กเกจ `composer require spatie/fork` แล้วใช้คำสั่ง `$results = Concurrency::driver('fork')->run([...]);` น้องจะพบว่ามันเร็วกว่าการใช้ `process` ธรรมดาอย่างเห็นได้ชัด!
*   **🚧 อย่าทำร้าย Server ตัวเอง:**
    การรัน Concurrency คือการสร้าง Process ย่อยขึ้นมากิน RAM ของเครื่องเซิร์ฟเวอร์ครับ ถ้าน้องมีคนเข้าเว็บพร้อมกัน 1,000 คน แล้วแต่ละคนสั่งรัน Concurrency แตกออกไปอีก 5 งาน เท่ากับ Server น้องต้องรับภาระถึง 5,000 Processes ในเสี้ยววินาที! ระบบอาจจะล่ม (OOM) ได้ทันที ให้ใช้ฟีเจอร์นี้กับงานที่คุ้มค่าแก่การแยกประมวลผลจริงๆ เท่านั้นครับ

## 7. 🏁 บทสรุป (To be continued...):
**Concurrency ใน Laravel 11** คืออีกหนึ่งก้าวสำคัญที่ทำให้ PHP ก้าวข้ามขีดจำกัดเดิมๆ ของตัวเองครับ จากที่เคยต้องรอนานๆ ตอนนี้เราสามารถแตกแขนงงานออกไปทำขนานกันได้อย่างสวยงามและอ่านโค้ดง่ายสุดๆ (Developer Experience ดีเยี่ยม) 

ฟีเจอร์นี้เหมาะมากสำหรับหน้า Dashboard สรุปผล, การดึงข้อมูลจาก Third-party APIs หลายๆ ตัวพร้อมกัน, หรือการทำ Web Scraping ลองนำไปปรับใช้กับโปรเจกต์ของน้องๆ ดูนะครับ รับรองว่าลูกค้าจะทึ่งกับความเร็วที่เพิ่มขึ้นแน่นอน!

มาถึงตรงนี้ น้องๆ ก็ได้เรียนรู้อาวุธหนักในการรีดประสิทธิภาพ (Performance Optimization) ไปอีกหนึ่งชิ้นแล้ว ในบทความหน้า พี่จะพาไปดูเครื่องมือที่ช่วยจัดการกับ "ข้อมูลมหาศาล" ที่โหลดเข้า Memory ไม่ไหว ด้วยสิ่งที่เรียกว่า **"Lazy Collections"** เตรียมตัวให้พร้อม แล้วพบกันครับ!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Automation ให้กับโรงงานของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและติดตั้งระบบแบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p