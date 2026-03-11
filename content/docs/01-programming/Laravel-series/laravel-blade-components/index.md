---
title: "Blade Components: แยกส่วน UI เพื่อการนำกลับมาใช้ใหม่"
weight: 60
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Laravel 11", "Blade", "Components", "Frontend", "UI"]
---

{{< figure src="laravel-blade-components-cover.jpg" alt="A futuristic developer desk with holographic Lego-like blocks forming a web UI" >}}

## 1. 🎯 ชื่อตอน (Title): Blade Components ศิลปะแห่งการประกอบร่าง UI สไตล์ตัวต่อ Lego

## 2. 📖 เปิดฉาก (The Hook):
มาครับน้องๆ วันนี้พี่ขอถามคำถามจี้ใจดำหน่อย... น้องๆ เคยต้องเขียนโค้ดปุ่ม (Button) หรือการ์ด (Card) ที่มีหน้าตาเหมือนๆ กัน แปะกระจายอยู่ 50 หน้าในโปรเจกต์ไหมครับ? แล้ววันดีคืนดี ลูกค้าหรือดีไซเนอร์เดินมาบอกว่า "พี่คะ ขอเปลี่ยนปุ่มจากสีน้ำเงินเป็นสีคราม ปรับขอบให้มนขึ้นอีกนิดนึงนะคะ" 

สิ่งที่เกิดขึ้นคือเราต้องมานั่งเปิดไฟล์ 50 ไฟล์แล้วกด Find & Replace ด้วยความหวาดระแวงว่าเว็บจะพัง! นี่แหละครับคือหายนะของการเขียนโค้ดซ้ำซ้อน (Spaghetti UI)

ในโลกของ Frontend Framework สมัยใหม่อย่าง React หรือ Vue เขาแก้ปัญหานี้ด้วยสิ่งที่เรียกว่า **"Component"** (การสร้าง UI เป็นชิ้นๆ แล้วนำมาประกอบกัน) ข่าวดีคือ Laravel ก็มีไม้ตายนี้เหมือนกันครับ! มันเรียกว่า **"Blade Components"** ซึ่งถูกปรับปรุงมาอย่างสมบูรณ์แบบใน Laravel 11 วันนี้พี่จะพาไปดูว่า เราจะแปลงโค้ด HTML รกๆ ให้กลายเป็น Custom Tag สุดหล่ออย่าง `<x-button>` ได้อย่างไร รับรองว่าชีวิตการทำ Frontend ของน้องๆ จะเปลี่ยนไปตลอดกาล!

## 3. 🧠 แก่นวิชา (Core Concepts):
Blade Components ถูกออกแบบมาให้ทำหน้าที่เหมือน Custom HTML Tags ครับ โดยแบ่งออกเป็น 2 ประเภทหลัก และมีกลไกรับส่งข้อมูล 2 แบบที่เราต้องเข้าใจ:

*   **🎭 1. ประเภทของ Component (The Two Flavors)**
    *   **Anonymous Components (สายมินิมอล):** ไม่มีคลาส PHP ควบคุม มีแค่ไฟล์ `.blade.php` เพียวๆ เหมาะสำหรับ UI โง่ๆ (Dumb UI) ที่ไม่ต้องคิดเลขหรือดึงฐานข้อมูล เช่น ปุ่ม, กล่อง Alert, หรือ Input Form
    *   **Class-based Components (สายจัดเต็ม):** มีทั้งคลาส PHP และไฟล์ Blade ทำงานคู่กัน เหมาะสำหรับ UI ที่ต้องมี Logic ซับซ้อน (Business Logic) ก่อนนำไปแสดงผล

*   **📦 2. การส่งข้อมูลด้วย Props (Data Properties)**
    ถ้าเราอยากส่งตัวแปร (เช่น สี, ข้อความ) เข้าไปใน Component เราจะส่งผ่านสิ่งที่เรียกว่า "Attributes" ถ้าเป็นข้อความธรรมดาพิมพ์ใส่ได้เลย แต่ถ้าเป็นตัวแปร PHP ให้ใส่เครื่องหมาย โคลอน `:` นำหน้า เช่น `<x-alert type="danger" :message="$error" />`

*   **🕳️ 3. การแทรกโครงสร้างด้วย Slots**
    Props มีไว้ส่ง "ข้อมูล (Data)" แต่ถ้าเราอยากส่ง "ก้อน HTML (Markup)" เข้าไปแทรกกลาง Component เราจะใช้ **Slot** ครับ โดยมีทั้งตัวแปร `$slot` (สำหรับเนื้อหาหลัก) และ **Named Slots** (สำหรับการเจาะรูเฉพาะจุด เช่น ส่วน Header หรือ Footer)

*   **🛍️ 4. กระเป๋าวิเศษ Attribute Bag (`$attributes`)**
    เวลาเราเรียกใช้ Component แล้วอยากแถมคลาส CSS พิเศษเข้าไป Laravel จะเก็บของแถมเหล่านั้นไว้ในตัวแปร `$attributes` ซึ่งเราสามารถจับมันไป Merge รวมกับคลาสพื้นฐานของ Component ได้อย่างเนียนกริ๊บ!

{{< figure src="laravel-blade-components-diagram.jpg" alt="System flow diagram illustrating Props and Slots feeding into a Blade Component" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code):

เรามาเริ่มสร้าง **Anonymous Component** แบบเทพๆ กันครับ สมมติว่าเราจะสร้าง "การ์ดโปรไฟล์"
เพียงแค่สร้างไฟล์ไว้ที่ `resources/views/components/card.blade.php` (ไม่ต้องรันคำสั่ง Artisan เลย)

```html
<!-- ไฟล์ resources/views/components/card.blade.php -->

<!-- 1. ประกาศรับตัวแปร (Props) พร้อมกำหนดค่า Default -->
@props(['theme' => 'light'])

<!-- 2. ใช้ $attributes->merge() เพื่อรวมคลาส CSS ที่แถมมา เข้ากับคลาสหลักของเรา -->
<div {{ $attributes->merge(['class' => 'card card-'.$theme]) }}>
    
    <!-- 3. ตรวจสอบว่ามีการส่ง Named Slot ที่ชื่อ header มาไหม? ถ้ามีก็แสดงผล -->
    @if(isset($header))
        <div class="card-header">
            {{ $header }}
        </div>
    @endif
    
    <!-- 4. เนื้อหาหลัก (Default Slot) จะมาโผล่ตรงนี้ -->
    <div class="card-body">
        {{ $slot }}
    </div>

</div>
```

**วิธีนำ Component ตัวนี้ไปใช้งานในหน้าเว็บหลัก (`welcome.blade.php`):**

```html
<!-- เรียกใช้การ์ด และแถมคลาส CSS พิเศษ (shadow-lg) ไปให้ด้วย -->
<x-card theme="dark" class="shadow-lg mt-4">
    
    <!-- ส่งโครงสร้าง HTML เข้าไปเสียบที่รู $header -->
    <x-slot:header>
        <h3 class="text-xl font-bold">ข้อมูลผู้ใช้งาน</h3>
    </x-slot:header>
    
    <!-- อะไรก็ตามที่ไม่ได้อยู่ใน x-slot จะวิ่งไปเข้าตัวแปร $slot อัตโนมัติ -->
    <p>ชื่อ: พี่วิสิทธิ์</p>
    <p>ตำแหน่ง: Senior Web Architect</p>
    
    <!-- ลองเรียกใช้ Component ที่อยู่ในโฟลเดอร์ย่อย เช่น components/forms/button.blade.php -->
    <x-forms.button type="submit">แก้ไขข้อมูล</x-forms.button>

</x-card>
```

*คอมเมนต์: เห็นความสวยงามนี้ไหมครับ? โค้ดหน้าเว็บเราจะอ่านง่ายเหมือนอ่านหนังสือเลยล่ะ ยิ่งถ้าทำโฟลเดอร์ย่อยอย่าง `forms` เราก็ใช้จุด (Dot notation) เป็น `<x-forms.button>` ได้เลย!*

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips):
จากคู่มือระดับลึกและประสบการณ์ของพี่ นี่คือจุดที่หลายคนมักจะพลาดครับ:

*   **💥 ความลับของ `$attributes->merge()`:** อันนี้สำคัญมาก! ถ้าน้องไม่ใช้เมธอดนี้ แล้วพิมพ์ `<div class="card" {{ $attributes }}>` เวลาเรียกใช้งาน `<x-card class="mt-4">` คลาส `mt-4` จะไปเขียนทับคลาส `card` หายวับไปเลยครับ! การใช้ `merge(['class' => 'card'])` จะช่วยเอาทั้งสองอันมาต่อกันอย่างปลอดภัย
*   **⚡ Short Attribute Syntax:** ใน Laravel 11 ถ้าน้องมีตัวแปรที่ชื่อตรงกับชื่อ Prop พอดี เช่น มีตัวแปร `$userId` แล้วอยากส่งเข้าไปใน Component แทนที่จะเขียน `<x-profile :user-id="$userId" />` น้องสามารถเขียนย่อๆ เท่ๆ แบบนี้ได้เลยครับ `<x-profile :$userId />` (ประหยัดเวลาพิมพ์ไปได้เยอะ!)
*   **🏗️ เมื่อไหร่ควรใช้ Class-based?** พี่แนะนำว่า 90% ของ UI ในระบบ (เช่น ปุ่ม, ฟอร์ม, การ์ด) ให้ใช้ **Anonymous Components** ก็พอครับเพราะมันเบาและเร็วกว่า แต่ถ้า Component นั้นต้องดึงข้อมูลจาก Database เอง (เช่น Component สรุปยอดขายรายเดือน) แบบนี้ควรใช้คำสั่ง `php artisan make:component MonthlySummary` เพื่อให้ได้คลาส PHP มาจัดการ Logic ให้สะอาดครับ

## 6. 🏁 บทสรุป (To be continued...):
**Blade Components** คือจิ๊กซอว์ชิ้นสำคัญที่เปลี่ยนการเขียน Frontend ใน Laravel ให้กลายเป็นเรื่องสนุกครับ มันช่วยลดความซ้ำซ้อน (DRY Principle) ทำให้โค้ดอ่านง่าย และที่สำคัญคือบำรุงรักษา (Maintain) ได้ง่ายมากๆ ถ้าวันไหนต้องแก้สไตล์ของปุ่มทั้งเว็บ เราก็แค่เดินไปแก้ที่ไฟล์ Component จุดเดียวจบ!

เมื่อเราสามารถจัดระเบียบหน้าบ้าน (UI) ให้สวยงามและมีโครงสร้างที่แข็งแรงได้แล้ว... ในตอนหน้า พี่จะพาข้ามกลับไปยังฝั่งหลังบ้าน (Backend) เพื่อพูดคุยถึงพระเอกตัวจริงของ Laravel นั่นคือระบบจัดการฐานข้อมูลสุดอัจฉริยะ **"Eloquent ORM"** เตรียมพร้อมบอกลาการเขียน SQL ดิบๆ ยาวๆ ไปได้เลย... ชงกาแฟแก้วใหม่ แล้วพบกันครับ!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Automation ให้กับโรงงานของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและติดตั้งระบบแบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p
