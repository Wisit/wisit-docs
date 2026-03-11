---
title: "Validation: ตรวจสอบข้อมูลให้ถูกต้องก่อนบันทึก"
weight: 100
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Laravel 11", "Validation", "Form Request", "Security", "Clean Code"]
---

{{< figure src="laravel-validation-form-request-cover.jpg" alt="A futuristic security checkpoint filtering data packets" >}}

## 1. 🎯 ชื่อตอน (Title): ตอนที่ 23: Validation ปราการด่านสุดท้าย คัดกรองข้อมูลขยะก่อนทะลวงถึงฐานข้อมูล

## 2. 📖 เปิดฉาก (The Hook):
มาครับน้องๆ ลากเก้าอี้มานั่งจิบกาแฟกัน... ในวงการโปรแกรมเมอร์ เรามีกฎเหล็กทองคำอยู่ข้อหนึ่งที่ท่องจำกันมาตั้งแต่ยุคบุกเบิก นั่นคือ **"Never trust user input (จงอย่าเชื่อใจข้อมูลที่ผู้ใช้กรอกเข้ามาเด็ดขาด)"** 

ลองจินตนาการดูว่า ถ้าน้องๆ ทำระบบลงทะเบียน แต่ปล่อยให้ผู้ใช้กรอกตัวเลขอายุเป็นตัวอักษร กรอกรหัสผ่านแค่ 2 ตัว หรือแม้กระทั่งแอบฝังโค้ดอันตราย (Malicious data) ส่งเข้ามา แล้วเราเอาไปบันทึกลงฐานข้อมูลตรงๆ... ระบบพังพินาศแน่นอนครับ! 

ในโลกของ Laravel เรามีระบบรักษาความปลอดภัยที่เรียกว่า **"Validation"** ซึ่งเปรียบเสมือน "การ์ดฝีมือดีหน้าคลับ (Bouncer)" ที่คอยตรวจบัตรประชาชนและค้นตัวก่อนปล่อยคนเข้างาน วันนี้พี่จะพาไปดูวิธีการตั้งด่านตรวจข้อมูลตั้งแต่แบบไวๆ ใน Controller ไปจนถึงวิถีของสถาปนิกซอฟต์แวร์ด้วยการสร้าง **Form Request** เพื่อแยกโค้ดให้สะอาดกริ๊บกันครับ!

## 3. 🧠 แก่นวิชา (Core Concepts):
การตรวจสอบข้อมูลใน Laravel 11 สามารถทำได้หลายวิธี แต่พี่ขอหยิบยก 2 วิธีหลักที่ใช้กันในชีวิตจริงมาฝากครับ:

*   **🛡️ วิธีที่ 1: The Quick Filter (ตรวจจับไวใน Controller)**
    เราสามารถเรียกใช้เมธอด `$request->validate()` ได้โดยตรงใน Controller วิธีนี้เหมาะสำหรับฟอร์มเล็กๆ ที่มีข้อมูลแค่ 2-3 ช่อง หากข้อมูลไม่ผ่านกฎที่ตั้งไว้ Laravel จะฉลาดพอที่จะ "เด้ง (Redirect)" ผู้ใช้กลับไปหน้าเดิมทันที พร้อมกับพกข้อความ Error กลับไปแสดงผลให้ด้วย (โดยที่เราไม่ต้องเขียน if-else เลย!)
*   **🏰 วิธีที่ 2: The Pro Shield (แยกส่วนด้วย Form Request)**
    เมื่อโปรเจกต์ใหญ่ขึ้น ถ้าเราเอา Validation Rules ยาวเป็นหางว่าวไปยัดไว้ใน Controller มันจะผิดหลัก Clean Code และทำให้ Controller อ้วนท้วน (Fat Controller) Laravel จึงมี **Form Request** ซึ่งเป็นคลาสพิเศษที่สร้างขึ้นมาเพื่อ "ห่อหุ้ม (Encapsulate)" โลจิกการตรวจสอบ (Validation) และการอนุญาต (Authorization) แยกออกไปเป็นสัดส่วนต่างหาก

{{< figure src="laravel-validation-form-request-diagram.jpg" alt="System flow diagram comparing Controller Validation vs Form Request Validation" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code):

เรามาดูวิวัฒนาการของการเขียนโค้ดกันครับ เริ่มจากวิธีพื้นฐาน ไปสู่วิธีระดับโปร

**สเต็ปที่ 1: การใช้ Validate ใน Controller (แบบพื้นฐาน)**
```php
<?php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Models\User;

class UserController extends Controller
{
    public function store(Request $request)
    {
        // กำหนดกฎการตรวจสอบ (Rules)
        $validated = $request->validate([
            'name' => 'required|string|max:255',
            'email' => 'required|email|unique:users', // ห้ามซ้ำกับตาราง users
            'password' => 'required|min:8|confirmed', // ต้องมี password_confirmation ส่งมาคู่กัน
        ]);

        // ถ้าโค้ดหลุดมาถึงบรรทัดนี้ได้ แปลว่าข้อมูลถูกต้อง 100% แล้ว!
        User::create($validated);

        return redirect('/users')->with('success', 'บันทึกสำเร็จ!');
    }
}
```
*คอมเมนต์: โค้ดนี้ใช้งานได้ดีครับ แต่น้องเห็นไหมว่าความรับผิดชอบของ Controller เริ่มเยอะเกินไปแล้ว (ทำทั้งตรวจสอบ ทำทั้งบันทึก)*

---

**สเต็ปที่ 2: ยกระดับด้วย Form Request (วิถี Senior)**
เราจะสั่งให้ Artisan สร้างด่านตรวจขึ้นมาใหม่:
```bash
php artisan make:request StoreUserRequest
```
เปิดไฟล์ที่ถูกสร้างขึ้นใน `app/Http/Requests/StoreUserRequest.php` แล้วย้ายกฎไปไว้ที่นั่น:

```php
<?php
namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StoreUserRequest extends FormRequest
{
    // 1. ตรวจสอบสิทธิ์ (Authorization) เช่น เป็น Admin หรือเปล่าถึงจะรันโค้ดนี้ได้
    public function authorize(): bool
    {
        return true; // อนุญาตให้ทุกคนใช้งาน
    }

    // 2. กำหนดกฎหมายของด่านตรวจ (Validation Rules)
    public function rules(): array
    {
        return [
            'name' => 'required|string|max:255',
            'email' => 'required|email|unique:users',
            'password' => 'required|min:8|confirmed',
        ];
    }
}
```

คราวนี้กลับมาดูที่ Controller ของเรากันครับ ดูความหล่อเหลานี้สิ!
```php
<?php
// ใน UserController.php
use App\Http\Requests\StoreUserRequest;

class UserController extends Controller
{
    // แค่เปลี่ยน Type-hint จาก Request ธรรมดา เป็น StoreUserRequest ที่เราเพิ่งสร้าง
    public function store(StoreUserRequest $request)
    {
        // โค้ดจะสะอาดกริ๊บ! ดึงเฉพาะข้อมูลที่ผ่านการตรวจสอบแล้วออกมา
        $validated = $request->validated(); 

        User::create($validated);
        return redirect('/users');
    }
}
```

**สเต็ปที่ 3: การแสดง Error ที่หน้า Blade (View)**
ที่หน้าฟอร์ม HTML เราสามารถใช้ตัวช่วย `@error` เพื่อแสดงข้อความเตือนได้อย่างสวยงาม:
```html
<label for="email">อีเมล</label>
<input type="email" name="email" value="{{ old('email') }}">

<!-- หาก email ไม่ผ่านกฎ ให้แสดงข้อความเตือน -->
@error('email')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips):
*   **☠️ หายนะของ `$request->all()`:** พี่ขอเตือนเลยว่า การใช้ `User::create($request->all())` เป็นสิ่งที่อันตรายมาก (Mass Assignment vulnerability) เพราะแฮกเกอร์อาจแอบส่งฟิลด์แปลกๆ เช่น `is_admin = 1` ยัดเข้ามาในฟอร์มได้ จงใช้ `$request->validated()` หรือ `$request->safe()->only(['name', 'email'])` เพื่อดึงเฉพาะข้อมูลที่เราตั้งกฎไว้เท่านั้นครับ!
*   **🗣️ เปลี่ยนภาษา Error Messages ให้เป็นกันเอง:** ถ้าน้องๆ รู้สึกว่าข้อความเตือนภาษาอังกฤษมันดูแข็งไป เราสามารถเพิ่มเมธอด `messages()` ลงไปในคลาส Form Request เพื่อแต่งข้อความเองได้เลย เช่น:
    ```php
    public function messages(): array
    {
        return [
            'email.required' => 'ลืมกรอกอีเมลหรือเปล่าจ๊ะ?',
            'email.unique' => 'อีเมลนี้มีคนแย่งใช้ไปแล้วน้องเอ๊ย!',
        ];
    }
    ```
*   **🛑 เบรกให้ล้อลากด้วย `bail`:** สมมติช่องอีเมลมีกฎคือ `required|unique:users|email` ถ้าผู้ใช้ "ไม่กรอกอะไรเลย (fail ที่ required)" ระบบก็จะดันทุรังไปเช็ค `unique` ในฐานข้อมูลต่อ (ทำให้เสียทรัพยากรฟรีๆ) ให้เราเติมกฎคำว่า `bail` ลงไปข้างหน้าสุด (เช่น `'email' => 'bail|required|email'`) มันแปลว่า "ถ้าตกม้าตายตั้งแต่ด่านแรก ก็หยุดตรวจด่านต่อไปทันที" ช่วยรีด Performance ได้ดีเยี่ยมครับ!

## 6. 🏁 บทสรุป (To be continued...):
จะเห็นได้ว่าการใช้ **Form Request** ไม่ได้เกิดมาเพื่อทำให้ระบบซับซ้อนขึ้น แต่มันถูกออกแบบมาตามหลักการ "Separation of Concerns" (แยกส่วนความรับผิดชอบ) มันช่วยให้ Controller ของน้องๆ เป็นอิสระ อ่านง่าย และสามารถนำโค้ดชุดนี้ไปเขียน Test ได้ง่ายขึ้นมหาศาลครับ

เมื่อเราสามารถคัดกรองข้อมูลที่เป็นตัวหนังสือ (Text) ได้อย่างปลอดภัยแล้ว... ในตอนต่อไป พี่จะพาไปงัดข้อกับของที่ชิ้นใหญ่ขึ้นและอันตรายยิ่งกว่า นั่นก็คือ **"File Uploads และการจัดการ Storage"** เราจะรับไฟล์รูปภาพจากผู้ใช้อย่างไรให้ปลอดภัย? เตรียมกาแฟให้พร้อม แล้วเจอกันครับ!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Automation ให้กับโรงงานของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและติดตั้งระบบแบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p