---
title: "Database Migrations: ระบบ Version Control สำหรับฐานข้อมูล"
weight: 70
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Laravel 11", "Database", "Migrations", "Schema", "Version Control"]
---

{{< figure src="laravel-database-migrations-cover.jpg" alt="A futuristic architect's desk with glowing holographic database blueprints" >}}

## 1. 🎯 ชื่อตอน (Title): Database Migrations สถาปนิกผู้คุมกฎ กางแบบแปลนฐานข้อมูลด้วยปลายคีย์บอร์ด

## 2. 📖 เปิดฉาก (The Hook):
มาครับน้องๆ ลากเก้าอี้มานั่งจิบกาแฟกัน... ลองจินตนาการถึงสถานการณ์สุดคลาสสิกในออฟฟิศดูนะครับ น้อง A เพิ่งเขียนฟีเจอร์ใหม่เสร็จ โดยมีการเข้าไปคลิกเพิ่มคอลัมน์ `status` ในฐานข้อมูลผ่าน phpMyAdmin จากนั้นน้อง A ก็ Push โค้ดขึ้น Git ให้เพื่อนๆ 

พอเช้าวันต่อมา น้อง B ดึงโค้ด (Pull) ลงมาลองรันที่เครื่องตัวเอง... บึ้ม! 💥 หน้าแดงเถือกพร้อม Error ว่า *"Column 'status' not found"* น้อง B เลยต้องตะโกนข้ามโต๊ะไปถามว่า *"พี่ครับ! พี่แก้อะไรในดาต้าเบสป่าวเนี่ย ส่งไฟล์ .sql มาให้ผม Import ทับหน่อยสิ!"*... การต้องมานั่ง Export/Import ไฟล์ฐานข้อมูลไปมาเวลาทำงานเป็นทีม มันคือฝันร้าย (Pain Point) และความยุ่งเหยิงระดับชาติครับ!

เพื่อจบปัญหานี้ วิศวกรซอฟต์แวร์จึงคิดค้นระบบ **"Database Migrations"** ขึ้นมาครับ มันเปรียบเสมือน **"ระบบ Version Control (เช่น Git) สำหรับฐานข้อมูล"** ที่ทำให้เราสามารถเขียน "แบบแปลน" การสร้างตารางลงในโค้ด PHP ได้เลย ไม่ต้องไปนั่งคลิกเองอีกต่อไป วันนี้พี่จะพาไปดูความหล่อเท่ของระบบนี้กันครับ!

## 3. 🧠 แก่นวิชา (Core Concepts):
การใช้ Migration มีข้อดีที่เหนือกว่าการไปสร้าง Table เองใน SQL ตรงๆ หลายมิติมากครับ:

*   **🏗️ Version Control (ควบคุมเวอร์ชันได้):** ทุกการเปลี่ยนแปลงของฐานข้อมูล (เพิ่ม/ลด/แก้ตาราง) จะถูกเก็บเป็นไฟล์โค้ด ซึ่งเราสามารถนำไปใส่ใน Git ให้เพื่อนร่วมทีมทุกคน หรือเซิร์ฟเวอร์ Production ดึงไปรันและได้โครงสร้าง Database ที่ "เหมือนกันเป๊ะ 100%"
*   **⏪ ย้อนเวลาได้ (Rollback):** ถ้าสร้างตารางผิด เราสามารถสั่ง "ย้อนกลับ (Undo)" สิ่งที่เพิ่งทำไปได้ง่ายๆ เพียงแค่พิมพ์คำสั่งเดียว
*   **🧩 โครงสร้างของไฟล์ Migration:** ใน 1 ไฟล์จะมี 2 เมธอดหลักเสมอคือ:
    *   `up()`: มีไว้สำหรับ **"สร้าง หรือ ขยาย"** (ทำไปข้างหน้า) เช่น สร้างตารางใหม่ หรือเพิ่มคอลัมน์
    *   `down()`: มีไว้สำหรับ **"ทำลาย หรือ ย้อนกลับ"** (ทำสิ่งที่ตรงข้ามกับ `up`) เช่น ลบตารางทิ้ง หรือลบคอลัมน์ออก
*   **🌟 Anonymous Classes:** ใน Laravel 11 เป็นต้นมา ไฟล์ Migration จะถูกสร้างมาเป็น Anonymous class (`return new class extends Migration`) เพื่อป้องกันปัญหาชื่อคลาสซ้ำกันเวลาเรามีไฟล์เยอะๆ ครับ

{{< figure src="laravel-database-migrations-diagram.jpg" alt="System flow diagram showing up() creating tables and down() dropping them" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code):
เรามาดูเวทมนตร์ในการวาดแบบแปลนตารางกันครับ สมมติว่าเราอยากสร้างตาราง `products`

**1. ร่ายมนต์สร้างไฟล์ Migration**
```bash
# ใช้คำสั่ง Artisan สร้างไฟล์ (ระบบจะเติม Timestamp ไว้หน้าไฟล์ให้อัตโนมัติ เพื่อจัดลำดับ)
php artisan make:migration create_products_table
```

**2. เขียนแบบแปลน (The Schema Builder)**
เข้าไปที่โฟลเดอร์ `database/migrations/` แล้วเปิดไฟล์ที่เพิ่งสร้างขึ้นมา:
```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    
    // เมื่อสั่งให้ "เดินหน้า" (php artisan migrate)
    public function up(): void {
        // ใช้ Schema::create เพื่อสร้างตารางใหม่
        Schema::create('products', function (Blueprint $table) {
            $table->id(); // สร้าง PK แบบ Auto-increment
            $table->string('name', 100); // VARCHAR(100)
            $table->text('description')->nullable(); // TEXT ที่ยอมให้เป็นค่าว่าง (NULL) ได้
            $table->decimal('price', 8, 2); // DECIMAL (ความยาว 8, ทศนิยม 2 ตำแหน่ง)
            $table->boolean('is_active')->default(true); // BOOLEAN ค่าเริ่มต้นคือ true
            $table->timestamps(); // สร้างคอลัมน์ created_at และ updated_at ให้ฟรีๆ!
        });
    }

    // เมื่อสั่ง "ถอยหลัง" (php artisan migrate:rollback)
    public function down(): void {
        // ลบตารางทิ้งถ้ามีอยู่
        Schema::dropIfExists('products');
    }
};
```

**3. การแก้ไขคอลัมน์ (Updating Tables)**
ถ้าวันนึงอยากเพิ่มคอลัมน์ `stock` เข้าไปในตาราง `products` เราจะไม่ไปแก้ไฟล์เก่านะครับ (เดี๋ยวประวัติพัง) ให้สร้างไฟล์ใหม่:
```bash
php artisan make:migration add_stock_to_products_table
```
```php
// ในไฟล์ใหม่
public function up(): void {
    // ใช้ Schema::table (ไม่ใช่ create นะ) เพื่อแก้ไขตารางเดิม
    Schema::table('products', function (Blueprint $table) {
        $table->integer('stock')->after('price'); // เติมคอลัมน์ stock ต่อท้าย price
    });
}

public function down(): void {
    Schema::table('products', function (Blueprint $table) {
        $table->dropColumn('stock'); // ย้อนกลับด้วยการลบคอลัมน์ทิ้งซะ
    });
}
```

**4. คำสั่งควบคุมผู้รับเหมา (Artisan Commands)**
```bash
# 🚀 สร้าง/อัปเดตตารางทั้งหมดที่ยังไม่ได้รัน
php artisan migrate

# ⏪ ย้อนกลับ Migration "รอบล่าสุด (Last Batch)"
php artisan migrate:rollback

# ⏪ ย้อนกลับไป 2 ก้าว
php artisan migrate:rollback --step=2

# ♻️ ล้างไพ่ทั้งหมด! (Drop ทิ้งทุกตาราง แล้วรัน up() ใหม่ตั้งแต่ต้น) เหมาะกับตอนกำลัง Dev
php artisan migrate:fresh
```

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips):
ในฐานะสถาปนิก พี่มีข้อควรระวังขั้นสูงมาฝากครับ:

*   **☠️ หายนะของการลืม `down()`:** บางครั้งนักพัฒนาขี้เกียจเขียนเมธอด `down()` ปล่อยให้มันว่างไว้... พี่บอกเลยว่านี่คือระเบิดเวลา! ถ้าวันนึงโค้ดขึ้น Production ไปแล้วบั๊กกระจาย เราต้องการกด Rollback กลับไปเวอร์ชันก่อนหน้า แต่ระบบย้อนกลับให้ไม่ได้เพราะไม่มีโค้ดใน `down()`... คราวนี้แอปพลิเคชันกับฐานข้อมูลจะพังและอยู่ในสถานะที่ไม่สอดคล้องกัน (Inconsistent state) ทันทีครับ! เขียนเผื่อไว้เสมอเถอะนะ 
*   **⚠️ กฎเหล็กของ Laravel 11 เวลาใช้ `->change()`:** ถ้าเราต้องการเปลี่ยนชนิดหรือขนาดคอลัมน์ด้วยเมธอด `change()` ใน Laravel 11 คุณ **"ต้อง"** ระบุ Attribute เก่าที่ต้องการเก็บไว้ให้ครบด้วยนะครับ! (เช่น ของเดิมเป็น `unsigned()->default(1)`) ถ้าตอน change คุณเขียนแค่ `->nullable()->change()` ตัว `unsigned` กับ `default` อันเก่าจะหลุดหายไปทันที! (นี่คือ High Impact Change ในเวอร์ชัน 11 เลยครับ)
*   **📦 บีบอัดประวัติศาสตร์ (Squashing Migrations):** ถ้าแอปเราพัฒนามา 3 ปี มีไฟล์ Migration เป็นร้อยๆ ไฟล์ เวลาเปิดโฟลเดอร์ทีคือตาลาย... Laravel มีคำสั่ง `php artisan schema:dump --prune` มันจะยุบไฟล์เก่าๆ ทั้งหมดรวมกันเป็นไฟล์ SQL ก้อนเดียว เพื่อความคลีนของโปรเจกต์ครับ 

## 6. 🏁 บทสรุป (To be continued...):
เห็นไหมครับว่า **Database Migrations** คือฮีโร่ที่ช่วยให้ทีมพัฒนาคุยภาษาเดียวกัน ทุกคนมีสคริปต์ก่อสร้างตารางที่ชัดเจน สามารถเดินหน้าและถอยหลังได้อย่างปลอดภัย ช่วยเปลี่ยนงาน Database ที่เคยน่ากลัวและยุ่งยาก ให้กลายเป็นโค้ดที่สวยงาม

ตอนนี้เรามี "โครงสร้างตาราง (Schema)" ที่แข็งแกร่งแล้ว แต่ตารางมันยังว่างเปล่าอยู่เลย! ในตอนหน้า พี่จะพาไปดูวิธีการเสกข้อมูลจำลอง (Dummy Data) เข้าตารางแบบอัตโนมัติด้วย **"Seeders และ Factories"** เตรียมตัวให้พร้อม แล้วพบกันครับ!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Automation ให้กับโรงงานของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและติดตั้งระบบแบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p