---
title: "Domain-Driven Design (DDD) ใน Laravel: สำหรับระบบขนาดใหญ่"
weight: 220
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Laravel 11", "DDD", "Architecture", "Enterprise", "Design Patterns"]
---

{{< figure src="laravel-ddd-architecture-cover.jpg" alt="Holographic blueprints organizing a digital city representing Domain-Driven Design" >}}

## 1. 🎯 ชื่อตอน (Title): ตอนที่ 43: Domain-Driven Design (DDD) สถาปัตยกรรมปราบมังกร สำหรับระบบระดับ Enterprise

## 2. 📖 เปิดฉาก (The Hook):
มาครับน้องๆ ลากเก้าอี้มานั่งจิบกาแฟกัน... ย้อนกลับไปตอนที่เราเริ่มเขียน Laravel ใหม่ๆ โครงสร้างแบบ MVC (Model-View-Controller) ที่เฟรมเวิร์กให้มา มันช่างสวยงามและรวดเร็วทันใจใช่ไหมครับ? มี `app/Http/Controllers` เอาไว้รับแขก มี `app/Models` เอาไว้คุยกับฐานข้อมูล ทุกอย่างดูเรียบง่าย

แต่เมื่อเวลาผ่านไป 2-3 ปี โปรเจกต์ของเราขยายตัวจากเว็บเล็กๆ กลายเป็นระบบ Enterprise ขนาดใหญ่ (เช่น ระบบ ERP หรือ E-Commerce ระดับประเทศ) น้องๆ เริ่มพบว่าในโฟลเดอร์ `Models` มีไฟล์กองรวมกันอยู่ 200 ไฟล์! ใน `Controllers` มีอีก 300 ไฟล์! 

วันดีคืนดี บอสเดินมาสั่งว่า *"น้อง ช่วยเพิ่มฟีเจอร์นับจำนวนพนักงานในหน้าแผนกให้หน่อย"* น้องอาจจะต้องเปิดแก้ไฟล์ที่กระจัดกระจายอยู่ถึง 5-6 โฟลเดอร์ ทั้ง Controller, Model, Request, Resource ฯลฯ แค่คิดก็ปวดหัวแล้ว แถมถ้ามีนักพัฒนาหน้าใหม่ (Junior Dev) เข้ามาร่วมทีม พวกเขาจะหลงทางในเขาวงกตโค้ดนี้อย่างแน่นอน 

เพื่อรับมือกับ "มังกร" (ความซับซ้อน) ตัวนี้ สถาปนิกระดับโลกจึงได้นำแนวคิด **Domain-Driven Design (DDD)** เข้ามาใช้ครับ แนวคิดนี้จะเปลี่ยนวิธีจัดโครงสร้างโปรเจกต์จากที่เคยแบ่งตาม "หน้าที่ทางเทคนิค (Technical)" ไปเป็นการแบ่งตาม "บริบททางธุรกิจ (Business Domain)" วันนี้พี่จะพาไปดูว่าบริษัทยักษ์ใหญ่เขาจัดโครงสร้าง Laravel กันอย่างไรครับ!

## 3. 🧠 แก่นวิชา (Core Concepts):
Domain-Driven Design ไม่ใช่แค่เรื่องของการจัดโฟลเดอร์ครับ แต่มันคือ "ปรัชญา" ที่พยายามทำให้ "ภาษาของโค้ด" ตรงกับ "ภาษาของธุรกิจ" มากที่สุด โดยในโลกของ Laravel เรามักจะแบ่งโลกออกเป็น 2 ใบครับ:

*   **🌍 The Application Layer (โลกของการสื่อสาร):**
    คือโฟลเดอร์ `app/` เดิมของเรานั่นแหละครับ หน้าที่ของมันคือการเป็น "พนักงานต้อนรับ" คอยรับ HTTP Request, รัน Console Commands, หรือจัดการ API โลกใบนี้จะ "ไม่มี Business Logic" อยู่เลย
*   **🏢 The Domain Layer (โลกของธุรกิจ):**
    เราจะสร้างโฟลเดอร์ใหม่ขึ้นมา เช่น `src/Domain/` โลกใบนี้จะถูกแบ่งเป็นแผนกๆ (Domains) เช่น `Invoice`, `Customer`, `Project` ภายในแผนกเหล่านี้จะมีโค้ดที่ทำหน้าที่คิดคำนวณและตัดสินใจทางธุรกิจทั้งหมด โดยมีพระเอกหลักๆ คือ:
    *   **📦 DTOs (Data Transfer Objects):** ออบเจกต์ที่เอาไว้ห่อหุ้มข้อมูลแทนการใช้ Array มั่วๆ ทำให้เรารู้ว่าข้อมูลที่ส่งไปมามีหน้าตาอย่างไร
    *   **⚙️ Actions:** คลาสที่รับผิดชอบการกระทำ "1 อย่างต่อ 1 คลาส (Single Use Case)" เช่น `CreateInvoiceAction`
    *   **💎 Value Objects:** คลาสที่ใช้แทนข้อมูลพื้นฐาน (Primitive types) แต่มีความหมายทางธุรกิจ เช่น คลาส `Money`, `Email`, `Percent`

{{< figure src="laravel-ddd-architecture-diagram.jpg" alt="System flow diagram comparing Standard MVC with tangled wires vs neatly organized DDD modules" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code):
เรามาดูวิวัฒนาการของการจัดโครงสร้างกันครับ สมมติว่าเรากำลังทำระบบ "ออกใบแจ้งหนี้ (Invoice)"

**สเต็ปที่ 1: การจัดโฟลเดอร์แบบ DDD**
แทนที่เราจะเอาทุกอย่างไปสุมไว้ใน `app/` เราจะสร้างโฟลเดอร์ `src/Domain/` ขึ้นมาแบบนี้ครับ:
```text
src/
 └── Domain/
     ├── Customer/
     │   ├── Actions/
     │   ├── DataTransferObjects/
     │   └── Models/
     └── Invoice/
         ├── Actions/
         ├── DataTransferObjects/
         └── Models/
```
*(เคล็ดลับ: เราต้องไปแก้ไฟล์ `composer.json` ตรงส่วน `autoload` เพื่อให้ Laravel รู้จักโฟลเดอร์ `src/Domain` ด้วยนะครับ)*

**สเต็ปที่ 2: The Application Layer (Controller สุดคลีน)**
Controller ของเราจะทำหน้าที่แค่รับ Request แปลงเป็น DTO แล้วโยนงานให้ Action ทำครับ
```php
<?php
namespace App\Http\Controllers;

use Domain\Invoice\Actions\CreateInvoiceAction;
use Domain\Invoice\DataTransferObjects\InvoiceData;
use Illuminate\Http\Request;

class InvoiceController extends Controller
{
    // Controller ทำหน้าที่แค่รับแขก!
    public function store(Request $request, CreateInvoiceAction $action)
    {
        // 1. แปลง Request Array ให้กลายเป็น DTO ที่มีโครงสร้างชัดเจน
        $dto = InvoiceData::fromRequest($request);

        // 2. สั่งให้ Action (คนงานใน Domain) ไปจัดการ Business Logic
        $invoice = $action->execute($dto);

        // 3. ตอบกลับลูกค้า
        return response()->json($invoice);
    }
}
```

**สเต็ปที่ 3: The Domain Layer (Action Class)**
นี่คือที่ซ่อน Business Logic ครับ คลาสนี้จะอิสระมาก สามารถเอาไปเรียกใช้ผ่าน Web, API, หรือรันผ่าน Console Command (Cron Job) ก็ได้ เพราะมันไม่ได้ยึดติดกับ HTTP Request เลย!

```php
<?php
namespace Domain\Invoice\Actions;

use Domain\Invoice\DataTransferObjects\InvoiceData;
use Domain\Invoice\Models\Invoice;

class CreateInvoiceAction
{
    // Action รับผิดชอบเรื่องเดียวเท่านั้น (Single Responsibility)
    public function execute(InvoiceData $data): Invoice
    {
        // สร้าง Invoice ลงฐานข้อมูล
        $invoice = Invoice::create([
            'customer_id' => $data->customerId,
            'amount' => $data->amount,
            'due_date' => $data->dueDate,
        ]);

        // อาจจะมีการยิง Event หรือ ส่ง Email ต่อที่นี่...
        
        return $invoice;
    }
}
```

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips):
การทำ DDD เป็นดาบสองคมครับ ถ้านำไปใช้ผิดที่ จะกลายเป็นฝันร้ายทันที นี่คือสิ่งที่ Senior ต้องระวัง:

*   **🚧 เส้นแบ่งเขตแดน (Boundaries):** กฎเหล็กของ DDD ที่ลึกซึ้งขึ้นคือ "ห้ามข้ามเขตแดนกันมั่วซั่ว" เช่น โค้ดใน Domain `Invoice` ไม่ควรไปดึง `CustomerModel` จาก Domain `Customer` มาแก้ไขตรงๆ (เพราะถ้าฝั่งนู้นแก้ตาราง โค้ดเราจะพัง) แต่ควรเรียกผ่าน `ClientService` เพื่อขอข้อมูลในรูปแบบ DTO แทน (แต่ข้อเสียคืออาจทำให้เกิด N+1 Query ได้ง่าย ต้องระวังให้ดี)
*   **🚨 การบังคับใช้กฎด้วย `deptrac`:** ถ้าน้องมีทีมใหญ่มาก จะรู้ได้อย่างไรว่าไม่มีใครเผลอเขียนโค้ดแหกกฎ? พี่แนะนำให้รู้จักกับเครื่องมือที่ชื่อว่า **Deptrac** ครับ มันคือ Static Analysis Tool ที่เราสามารถเขียนไฟล์ config กำหนดได้เลยว่า "Model ห้ามเรียกใช้ Job เด็ดขาด" ถ้าระบบ CI/CD ตรวจเจอ โค้ดชุดนั้นจะ Deploy ไม่ผ่านทันที! โคตรเฉียบ!
*   **⚖️ ข้อดี vs ข้อเสีย (Pros & Cons):**
    *   **✅ ข้อดี:** โค้ดสะท้อนภาพธุรกิจชัดเจน, เทสต์ (Unit Test) โคตรง่ายเพราะแยกส่วนชัดเจน, ควบคุมการสเกลทีมใหญ่ๆ ได้ดี แต่ละทีมดูแลคนละ Domain
    *   **❌ ข้อเสีย:** คลาสเยอะมหาศาล! (Class Explosion) แค่จะทำระบบบันทึกค่าง่ายๆ น้องอาจจะต้องสร้างถึง 6 ไฟล์ (Controller, Request, DTO, Action, Model, ViewModel) และมันขัดกับ "ธรรมชาติความง่าย (Magic)" ของ Laravel ไปพอสมควร

## 6. 🏁 บทสรุป (To be continued...):
**Domain-Driven Design (DDD)** ไม่ใช่ "กระสุนเงิน (Silver Bullet)" ที่จะแก้ปัญหาได้ทุกอย่างครับ ถ้าน้องทำระบบจองโต๊ะร้านอาหารเล็กๆ การใช้ DDD คือการขี่ช้างจับตั๊กแตนที่สิ้นเปลืองงบประมาณสุดๆ

แต่ถ้าน้องกำลังสร้างระบบ Core Banking, ระบบ Logistics ที่มีเงื่อนไขซับซ้อน หรือแอปพลิเคชันระดับ Shopify ที่มีโค้ดหลักล้านบรรทัด การแยกส่วนแบบ DDD คือ "เครื่องช่วยชีวิต" ที่จะป้องกันไม่ให้โปรเจกต์ของน้องพังทลายลงมาด้วยน้ำหนักของตัวมันเอง 

ความสวยงามคือ น้องๆ ไม่จำเป็นต้องทำ DDD 100% ก็ได้ครับ ลองหยิบแค่ **DTO** หรือ **Action Classes** มาใช้เพื่อกำจัด Controller อ้วนๆ ดูก่อน แค่นี้โค้ดก็หล่อขึ้นเป็นกองแล้ว! สำหรับซีรีส์เส้นทางสู่การเป็น Laravel Architect ของเราก็เดินทางมาถึงระดับสุดยอดแล้ว หวังว่าวิชาเหล่านี้จะเป็นประโยชน์กับน้องๆ ในสมรภูมิโค้ดของจริงนะครับ ขอให้สนุกกับการเขียนโปรแกรมครับ!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Automation ให้กับโรงงานของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและติดตั้งระบบแบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p