---
title: "Database Seeding & Factories: สร้างข้อมูลทดสอบมหาศาลในพริบตา"
weight: 90
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Laravel 11", "Database", "Seeding", "Factories", "Testing"]
---

{{< figure src="laravel-database-seeding-factories-cover.jpg" alt="A futuristic developer desk with holographic seeds transforming into database structures" >}}

## 1. 🎯 ชื่อตอน (Title): ตอนที่ 22: Database Seeding & Factories เสกข้อมูลทดสอบระดับแสนเรคคอร์ดในพริบตาเดียว

## 2. 📖 เปิดฉาก (The Hook):
มาครับน้องๆ ลากเก้าอี้มานั่งจิบกาแฟกัน... ลองจินตนาการดูนะครับว่า น้องๆ เพิ่งเขียนระบบ E-Commerce เสร็จ โค้ดหน้าบ้าน (Frontend) สวยงามมาก มีระบบแบ่งหน้า (Pagination) มีระบบค้นหาสินค้า (Search) แต่พอรันเว็บขึ้นมา... ว่างเปล่า! ไม่มีสินค้าสักชิ้นให้ทดสอบเลย 

ถ้าเป็นสมัยก่อน เราคงต้องเปิด phpMyAdmin แล้วนั่งพิมพ์ข้อมูลสินค้าทีละบรรทัด หรือเขียนสคริปต์ `for` loop กากๆ มาวนลูป Insert ข้อมูล ซึ่งทั้งเสียเวลาและได้ข้อมูลที่ไม่สมจริง (เช่น สินค้าชื่อ "Test 1", "Test 2")

แต่ในโลกของ Laravel เราจะไม่ทำแบบนั้นครับ! สถาปนิกของ Laravel ได้เตรียมระบบ "โรงงานผลิตข้อมูล" ที่เรียกว่า **Model Factories** และ "ผู้หว่านเมล็ดพันธุ์" ที่เรียกว่า **Database Seeders** เอาไว้ให้เราแล้ว เมื่อรวมพลังกับไลบรารีสร้างข้อมูลปลอมอย่าง **Faker** เราจะสามารถเสกข้อมูลลูกค้า สินค้า หรือบทความที่ดูสมจริงจำนวนหลักหมื่นหลักแสนเรคคอร์ด ลงฐานข้อมูลได้ด้วยการเคาะคีย์บอร์ดเพียงครั้งเดียว! วันนี้พี่จะพาไปดูเวทมนตร์นี้กันครับ

## 3. 🧠 แก่นวิชา (Core Concepts):
กลไกการสร้างข้อมูลจำลอง (Mock Data) ใน Laravel ประกอบด้วย 3 ทหารเสือที่ทำงานร่วมกันครับ:

*   **🎭 Faker (นักแต่งเรื่องโกหก):** 
    เป็นไลบรารี PHP ที่ถูกฝังมาใน Laravel ทำหน้าที่สร้างข้อมูลสุ่ม (Random data) ที่ดูสมจริงมากๆ เช่น ชื่อคน, อีเมล, เบอร์โทร, หรือแม้แต่ข้อความยาวๆ (Paragraph) ใน Laravel 11 เราสามารถเรียกใช้ผ่านฟังก์ชัน `fake()` ได้เลย,
*   **🏭 Model Factories (แบบแปลนโรงงาน):** 
    คือคลาสที่กำหนด "โครงสร้าง" ว่าข้อมูล 1 แถวของ Model นั้นๆ หน้าตาควรเป็นอย่างไร (เช่น Product ต้องมีชื่อจาก Faker และมีราคาเป็นตัวเลขสุ่ม),
*   **🌱 Database Seeders (คนหว่านเมล็ด):** 
    คือสคริปต์ที่ทำหน้าที่สั่งรัน Factory หรือสั่ง Insert ข้อมูลลงฐานข้อมูลจริงๆ โดยจะมีคลาสศูนย์กลางชื่อ `DatabaseSeeder` เป็นตัวควบคุมลำดับการรัน,

{{< figure src="laravel-database-seeding-factories-diagram.jpg" alt="System flow diagram showing Faker feeding Factory, which is executed by Seeder into the Database" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code):
เรามาลองเสก "สินค้า (Products)" จำนวน 100 ชิ้นกันครับ! 

**ขั้นตอนที่ 1: สร้าง Factory และ Seeder แบบคอมโบ**
เราสามารถใช้ Artisan สร้าง Model พร้อม Factory และ Seeder ได้ในคำสั่งเดียวด้วย Flag `-mfs`
```bash
php artisan make:model Product -mfs
```

**ขั้นตอนที่ 2: วางแบบแปลนใน Factory (`database/factories/ProductFactory.php`)**
เปิดไฟล์ Factory ขึ้นมา แล้วใช้ฟังก์ชัน `fake()` เพื่อกำหนดหน้าตาข้อมูล,
```php
<?php

namespace Database\Factories;

use Illuminate\Database\Eloquent\Factories\Factory;

class ProductFactory extends Factory
{
    public function definition(): array
    {
        return [
            // สุ่มคำศัพท์มาเป็นชื่อสินค้า
            'name' => fake()->word(), 
            // สุ่มข้อความจำลองมาเป็นรายละเอียด
            'description' => fake()->paragraph(), 
            // สุ่มราคาสินค้าตั้งแต่ 10 ถึง 1000 ทศนิยม 2 ตำแหน่ง
            'price' => fake()->randomFloat(2, 10, 1000), 
            // สุ่มจำนวนสต็อก 0-100
            'stock' => fake()->numberBetween(0, 100), 
        ];
    }
}
```

**ขั้นตอนที่ 3: สั่งให้ Seeder เริ่มผลิต (`database/seeders/ProductSeeder.php`)**
ไปที่ไฟล์ Seeder แล้วเรียกใช้ Factory ที่เราเพิ่งสร้าง โดยระบุจำนวน (Count) ที่ต้องการ,
```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use App\Models\Product;

class ProductSeeder extends Seeder
{
    public function run(): void
    {
        // สั่งโรงงานผลิต Product ออกมา 100 ชิ้น แล้วบันทึกลง Database ทันที!
        Product::factory()->count(100)->create();
    }
}
```

**ขั้นตอนที่ 4: ลงทะเบียน Seeder หลัก (`database/seeders/DatabaseSeeder.php`)**
เพื่อให้คำสั่งรันทำงานได้ครบ เราต้องเอา `ProductSeeder` ไปใส่ไว้ในไฟล์หลัก,
```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    public function run(): void
    {
        $this->call([
            ProductSeeder::class,
        ]);
    }
}
```

**ขั้นตอนที่ 5: กดปุ่มเดินเครื่อง! (Terminal)**
```bash
# รัน Seeder ทั้งหมด
php artisan db:seed

# หรือถ้าอยากล้างฐานข้อมูลเก่าทิ้งทั้งหมด แล้วสร้างตารางใหม่พร้อมใส่ข้อมูลเลย (ไม้ตายตอน Dev)
php artisan migrate:fresh --seed
```

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips):
ในฐานะสถาปนิกซอฟต์แวร์ พี่มีเคล็ดวิชาขั้นสูงเกี่ยวกับการทำ Seeding มาแชร์ครับ:

*   **✨ การเสกข้อมูลแบบมีความสัมพันธ์ (Magic Relationships):** 
    ถ้าน้องๆ มี Model `User` และ `Post` (ผู้ใช้ 1 คน มีหลายบทความ) น้องไม่ต้องมานั่งสุ่ม ID เองครับ! Laravel ให้เราเขียนเชน (Chain) เมธอดได้อย่างหรูหรา เช่น `User::factory()->hasPosts(3)->create();` คำสั่งเดียวนี้จะสร้าง User 1 คน และสร้าง Post ให้คนๆ นั้น 3 บทความทันที! โคตรหล่อ!
*   **🎭 สถานะพิเศษด้วย Factory States:**
    บางครั้งเราอยากได้ "แอดมิน" ปะปนมาในข้อมูลด้วย เราสามารถสร้าง State พิเศษใน Factory ได้ (เช่น `public function admin() { return $this->state(...); }`) แล้วเวลาเรียกใช้ก็แค่พิมพ์ `User::factory()->count(2)->admin()->create();` เราก็จะได้แอดมิน 2 คนมาใช้งานแล้ว
*   **⚠️ ระเบิดเวลาบน Production:** 
    จำไว้เสมอว่าคำสั่ง `php artisan migrate:fresh --seed` เป็นการ **ดรอป (Drop) ตารางทั้งหมดทิ้ง**! ห้าม (ย้ำว่าห้าม!) เอาไปรันเล่นบนเซิร์ฟเวอร์ Production เด็ดขาด (ถึงแม้ Laravel จะมีระบบเตือนให้พิมพ์ `yes` เพื่อยืนยันก็เถอะ) ฟีเจอร์นี้ออกแบบมาให้ใช้ตอน Development และ Testing เท่านั้นครับ

## 6. 🏁 บทสรุป (To be continued...):
เห็นความทรงพลังของ **Database Seeding และ Factories** ไหมครับ? มันช่วยเปลี่ยนงานที่น่าเบื่อและใช้เวลามาก ให้กลายเป็นเรื่องสนุกและสร้างสรรค์ (Developer Experience) ทีนี้เวลาน้องๆ จะเทสต์ระบบแบ่งหน้า หรือส่งโปรเจกต์ให้เพื่อนร่วมทีม ก็มั่นใจได้เลยว่าทุกคนจะมีฐานข้อมูลที่มีหน้าตาเหมือนกันและมีข้อมูลพร้อมทดสอบทันที

เมื่อเรามีข้อมูลมหาศาลอยู่ในฐานข้อมูลแล้ว... ในตอนหน้า พี่จะพาไปเจาะลึกวิชาการดึงข้อมูลขั้นสูงด้วย **"Eloquent Query Builder"** มาดูกันว่าเราจะรีดประสิทธิภาพในการค้นหาและกรองข้อมูลพวกนี้ออกมาได้อย่างไร... เตรียมตัวให้พร้อม แล้วพบกันครับเหล่า Artisan!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Automation ให้กับโรงงานของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและติดตั้งระบบแบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p