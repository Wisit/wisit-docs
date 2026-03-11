---
title: "Security Mastery: การเข้ารหัส (Encryption) และการทำ Hashing"
weight: 210
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Laravel 11", "Security", "Encryption", "Hashing", "Bcrypt"]
---

{{< figure src="laravel-11-security-encryption-hashing-cover.jpg" alt="A holographic shield showing the difference between Encryption and Hashing" >}}

## 1. 🎯 ชื่อตอน (Title): ตอนที่ 42: Security Mastery ถอดรหัสลับ Encryption และ Hashing ศิลปะการปกป้องข้อมูลขั้นสูงสุด

## 2. 📖 เปิดฉาก (The Hook):
มาครับน้องๆ ลากเก้าอี้มานั่งจิบกาแฟกัน... ลองจินตนาการถึงเหตุการณ์ฝันร้ายที่สุดของคนทำเว็บดูนะครับ วันดีคืนดี ฐานข้อมูล (Database) ของบริษัทเราโดนแฮกเกอร์เจาะเข้ามาได้ สิ่งแรกที่พวกมันจะมองหาคือ "ข้อมูลส่วนตัวลูกค้า" และ "รหัสผ่าน" 

ถ้าน้องเก็บรหัสผ่านเป็นตัวอักษรธรรมดาๆ (Plain text) เช่น `password123` หรือเก็บเลขบัตรเครดิตแบบโต้งๆ... เตรียมตัวเขียนจดหมายขอโทษลูกค้าและรอรับหมายศาลได้เลยครับ! 

โปรแกรมเมอร์หลายคนยังสับสนและมักจะพูดว่า *"เดี๋ยวเราเข้ารหัส (Encrypt) รหัสผ่านกันเถอะ"* ซึ่งเป็นคำพูดที่ **ผิดมหันต์** ครับ! ในโลกของ Security เรามีอาวุธ 2 ชิ้นที่ทำหน้าที่ต่างกันอย่างสิ้นเชิง นั่นคือ **Encryption (การเข้ารหัส)** และ **Hashing (การสับข้อมูล)** วันนี้พี่จะพาไปดูความแตกต่าง และวิถีของสถาปนิกซอฟต์แวร์ในการนำเครื่องมือเหล่านี้มาใช้ใน Laravel อย่างถูกต้องกันครับ!

## 3. 🧠 แก่นวิชา (Core Concepts):
ก่อนจะไปเขียนโค้ด เราต้องแยกให้ออกก่อนว่า 2 ตัวนี้ต่างกันอย่างไร:

*   **🔒 Encryption (การเข้ารหัสแบบไป-กลับได้):** 
    เปรียบเสมือน "ตู้เซฟ" ที่เราเอาเอกสารใส่เข้าไปแล้วล็อกกุญแจ ข้อมูลจะอ่านไม่ออกจนกว่าเราจะเอา "กุญแจ (Key)" ดอกเดิมมาไขเพื่ออ่านข้อมูลนั้นอีกครั้ง (Two-way algorithm). เหมาะสำหรับเก็บข้อมูลความลับที่เราต้องนำกลับมาใช้งานอีก เช่น เลขบัตรเครดิต, ประวัติการรักษาพยาบาล, หรือเบอร์โทรศัพท์. 
*   **🥩 Hashing (การสับข้อมูลแบบวันเวย์):** 
    เปรียบเสมือน "เครื่องบดเนื้อ" ถ้าน้องเอาหมูชิ้นใส่เข้าไป มันจะออกมาเป็นหมูบด (Hash) และน้อง **ไม่สามารถ** ปั้นหมูบดให้กลับมาเป็นหมูชิ้นเหมือนเดิมได้อีกต่อไป! (One-way algorithm). การทำ Hash ข้อมูลที่เหมือนกัน จะได้ผลลัพธ์ออกมาเหมือนกันเสมอ. เหมาะที่สุดสำหรับการเก็บ **รหัสผ่าน (Passwords)** เพราะแม้แต่ตัวเราเอง (ผู้ดูแลระบบ) ก็ไม่ควรล่วงรู้รหัสผ่านของลูกค้า.
*   **⏳ ความช้าคือความปลอดภัย (Work Factor):**
    ในการทำ Hashing สำหรับรหัสผ่าน ยิ่งอัลกอริทึมทำงานช้าเท่าไหร่ยิ่งดีครับ! เพราะมันจะทำให้แฮกเกอร์ที่ใช้เทคนิค Brute-force (สุ่มเดารหัส) หรือ Rainbow tables ทำงานได้ยากและใช้เวลามหาศาล. ใน Laravel จะใช้ `Bcrypt` หรือ `Argon2` เป็นค่าเริ่มต้น ซึ่งเราสามารถปรับเพิ่มระยะเวลาการประมวลผล (Cost/Work factor) ให้ตามทันความแรงของคอมพิวเตอร์ในยุคปัจจุบันได้.

{{< figure src="laravel-11-security-encryption-hashing-diagram.jpg" alt="System flow diagram comparing Encryption (Two-way) vs Hashing (One-way)" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code):

### 🔥 พาร์ท A: การใช้ Hashing ปกป้องรหัสผ่าน
ใน Laravel เรามี `Hash` Facade ที่ครอบความซับซ้อนของการทำ Hashing ไว้ให้หมดแล้วครับ

```php
<?php
namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;

class AuthController extends Controller
{
    public function register(Request $request)
    {
        // 1. การสร้าง Hash สำหรับเก็บลง Database
        $user = User::create([
            'name' => $request->name,
            'email' => $request->email,
            // ห้ามเก็บ $request->password ตรงๆ เด็ดขาด! ให้ครอบด้วย Hash::make()
            'password' => Hash::make($request->password), //
        ]);
    }

    public function login(Request $request)
    {
        $user = User::where('email', $request->email)->first();

        // 2. การตรวจสอบรหัสผ่าน (Verify)
        // Hash::check จะเปรียบเทียบ Plain text กับ Hash string อย่างปลอดภัย
        if ($user && Hash::check($request->password, $user->password)) { 
            
            // 3. (Pro-Tip) ตรวจสอบว่าระบบเพิ่งอัปเกรดอัลกอริทึมให้แข็งแกร่งขึ้นหรือไม่
            // ถ้าใช่ ให้สับรหัสผ่านใหม่ (Rehash) แล้วเซฟทับของเก่าไปเลย
            if (Hash::needsRehash($user->password)) {
                $user->update(['password' => Hash::make($request->password)]);
            }

            return "ล็อกอินสำเร็จ!";
        }

        return "รหัสผ่านไม่ถูกต้อง";
    }
}
```

### 🚀 พาร์ท B: การใช้ Encryption ปกป้องข้อมูลส่วนตัว (PII)
สมมติว่าเราต้องเก็บข้อมูล "บัตรประจำตัวประชาชน" ซึ่งเราต้องดึงกลับมาแสดงผลให้แอดมินดู เราจะใช้ Encryption ผ่าน `Crypt` Facade ครับ โดยระบบจะใช้กุญแจจากค่า `APP_KEY` ในไฟล์ `.env`.

```php
use Illuminate\Support\Facades\Crypt;
use Illuminate\Contracts\Encryption\DecryptException;

// การเข้ารหัส (Encrypt) ข้อมูล
$encryptedId = Crypt::encryptString('1103400000000'); //

// การถอดรหัส (Decrypt) ข้อมูล
try {
    $decryptedId = Crypt::decryptString($encryptedId); //
} catch (DecryptException $e) {
    // ดักจับ Error เผื่อมีใครแอบแก้ไขข้อมูล Hash หรือกุญแจ APP_KEY เปลี่ยน
    abort(403, 'ข้อมูลถูกลักลอบดัดแปลง หรือกุญแจไม่ถูกต้อง!');
}
```

**✨ ไม้ตายสุดหล่อ: Auto-Encryption ใน Eloquent Model**
ถ้าตารางของน้องๆ มีหลายฟิลด์ที่ต้องเข้ารหัส การมานั่งเขียน `Crypt::encrypt()` ทุกครั้งคงเหนื่อยแย่ Laravel มีตัวช่วยที่เรียกว่า `HasEncryption` Trait ครับ.

เปิดไฟล์ `app/Models/Patient.php` ขึ้นมา:
```php
<?php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Concerns\HasEncryption; // 1. ดึง Trait มาใช้

class Patient extends Model
{
    use HasEncryption;

    /**
     * 2. กำหนดชื่อฟิลด์ (Attributes) ที่ต้องการให้ Laravel แอบเข้ารหัสและถอดรหัสให้อัตโนมัติ
     */
    protected $encryptable = [
        'social_security_number',
        'medical_notes', // ข้อมูลประวัติการรักษา
    ];
}
```
*เวทมนตร์:* หลังจากใส่อันนี้ พอน้องสั่ง `$patient->medical_notes = "ปวดหัว";` แล้วกด `save()` ลงฐานข้อมูล มันจะกลายเป็นสตริงเข้ารหัสที่อ่านไม่ออกทันที! แต่พอน้องดึงข้อมูลกลับมา `$patient->medical_notes` Laravel ก็จะถอดรหัสให้กลับมาเป็นคำว่า "ปวดหัว" อย่างแนบเนียน เราทำงานกับมันได้เหมือนข้อความปกติเลยครับ!.

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips):
*   **🔑 ปกป้อง `APP_KEY` ยิ่งชีพ:** กุญแจ `APP_KEY` ในไฟล์ `.env` คือหัวใจของการ Encryption ทั้งหมดในแอปครับ ถ้ารหัสนี้หลุดไป แฮกเกอร์จะไขข้อมูลความลับของเราได้ทั้งหมด และถ้าเราเผลอไปกด `php artisan key:generate` สร้างกุญแจใหม่ทับของเดิม ข้อมูลที่เคยเข้ารหัสไว้ใน Database จะพังและกู้กลับมาไม่ได้อีกเลยนะครับ! (ต้องระวังมากๆ ตอน Deploy).
*   **🧂 เกลือ (Salt) สำหรับ Hashing:** ทราบไหมครับว่า ทำไมรหัสผ่าน `123456` เหมือนกันของ 2 ยูสเซอร์ ถึงได้รหัส Hash ใน Database ไม่เหมือนกันเลย? นั่นเป็นเพราะอัลกอริทึม Bcrypt จะแอบเติม "Salt (สตริงสุ่ม)" เข้าไปผสมกับรหัสผ่านก่อนสับเสมอ เพื่อป้องกันการโจมตีด้วยตารางรหัสผ่านสำเร็จรูป (Rainbow tables).
*   **⏱️ การโจมตีด้วยเวลา (Timing Attacks):** เวลาเราเปรียบเทียบ Hash ห้ามใช้เครื่องหมาย `==` แบบปกตินะครับ เพราะแฮกเกอร์สามารถจับเวลาการประมวลผลระดับมิลลิวินาทีเพื่อแกะรอยได้ (ถ้าตัวอักษรแรกผิด ระบบจะคายผลลัพธ์เร็วกว่าปกติ) โชคดีที่ `Hash::check()` ของ Laravel ทำงานแบบ Constant-time comparison เพื่อป้องกันปัญหานี้ให้เราเรียบร้อยแล้ว.

## 6. 🏁 บทสรุป (To be continued...):
**Encryption** มีไว้สำหรับรักษาความลับที่เราต้องนำกลับมาอ่านอีก (ใช้กุญแจล็อกและปลดล็อก) ส่วน **Hashing** มีไว้เพื่อพิสูจน์ความถูกต้องโดยที่ไม่มีใครสามารถย้อนกลับไปดูข้อมูลต้นฉบับได้ (ใช้กับรหัสผ่าน).

ในฐานะสถาปนิกซอฟต์แวร์ การเลือกเครื่องมือปกป้องข้อมูลให้ถูกประเภท คือเส้นแบ่งระหว่าง "ระบบที่ปลอดภัย" กับ "ระบบที่รอวันพัง" ครับ 

เมื่อแอปพลิเคชันของเรามีระบบความปลอดภัยที่แน่นหนาแล้ว ในตอนต่อไป พี่จะพาไปงัดข้อกับหัวข้อสุดท้ายที่เป็นความท้าทายระดับสุดยอด นั่นคือ **"การทำ Deployment และ CI/CD Automation"** เปลี่ยนโค้ดจากเครื่องเราให้ขึ้นไปเฉิดฉายบน Production อย่างสง่างาม เตรียมกาแฟแก้วสุดท้าย แล้วพบกันครับ!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Automation ให้กับโรงงานของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและติดตั้งระบบแบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p