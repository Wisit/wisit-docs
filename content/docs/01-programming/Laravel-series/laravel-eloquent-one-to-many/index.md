---
title: "Eloquent Relationships: เชื่อมโยงข้อมูลแบบ 1-to-Many"
weight: 130
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Laravel 11", "Eloquent ORM", "Database", "Relationships", "One-to-Many"]
---

{{< figure src="laravel-eloquent-one-to-many-cover.jpg" alt="A holographic diagram showing one User connected to multiple Posts" >}}

## 1. 🎯 ชื่อตอน (Title): ตอนที่ 26: Eloquent Relationships (1-to-Many) ถักทอสายใยข้อมูล เมื่อหนึ่งคนสร้างสรรค์ได้หลายบทความ

## 2. 📖 เปิดฉาก (The Hook):
มาครับน้องๆ ลากเก้าอี้มานั่งจิบกาแฟกัน... ในบทความก่อนๆ เราเรียนรู้วิธีการใช้ Eloquent ORM ดึงข้อมูลจากตารางเดี่ยวๆ กันไปแล้ว แต่ในโลกความเป็นจริงของฐานข้อมูลเชิงสัมพันธ์ (Relational Database) ตารางมันแทบจะไม่เคยอยู่โดดเดี่ยวเลยครับ มันต้องมีการ "เชื่อมโยง" กันเสมอ

ลองนึกภาพว่าเรากำลังทำระบบ Blog ที่เพิ่งเริ่มไปใน Workshop ตอนที่แล้ว เรามีตาราง `users` (ผู้ใช้งาน) และตาราง `posts` (บทความ) ถ้าน้องเขียน PHP แบบเพียวๆ เวลาจะหาว่า "นาย A เขียนบทความอะไรบ้าง?" น้องคงต้องเขียนคำสั่ง SQL ยาวๆ แบบ `SELECT * FROM posts WHERE user_id = 1` แล้วเอามาวนลูปแสดงผลใช่ไหมครับ? 

บอกเลยว่าถ้าทำแบบนั้นใน Laravel ถือว่ายังดึงพลังของมันออกมาไม่สุดครับ! เพราะ Laravel มีฟีเจอร์ระดับเทพที่เรียกว่า **"Eloquent Relationships"** ที่จะเปลี่ยนการ Join ตารางที่น่าปวดหัว ให้กลายเป็นการเขียนโค้ดเชิงวัตถุ (Object-Oriented) ที่อ่านง่ายเหมือนภาษาอังกฤษ วันนี้พี่จะพาไปเจาะลึกความสัมพันธ์ที่ใช้บ่อยที่สุด นั่นคือ **"One-to-Many (หนึ่งต่อหลาย)"** เตรียมเปิดคัมภีร์กันได้เลย!

## 3. 🧠 แก่นวิชา (Core Concepts):
ความสัมพันธ์แบบ **One-to-Many** คือการที่ข้อมูล 1 แถวในตารางหลัก สามารถเชื่อมโยงกับข้อมูลได้หลายแถวในตารางย่อย เปรียบเทียบง่ายๆ คือ "ผู้ใช้ 1 คน (User) สามารถมีบทความได้หลายบทความ (Posts)" 

สถาปัตยกรรมของ Eloquent อนุญาตให้เรากำหนดความสัมพันธ์นี้ผ่าน "Method (เมธอด)" ใน Model คลาสครับ โดยมีกฎเหล็ก 2 ข้อที่ต้องจำให้ขึ้นใจ:

*   **👨‍👧 ฝั่งพ่อแม่ (The Parent):** ฝั่งที่มี "หนึ่งเดียว" (เช่น User) จะใช้เมธอด **`hasMany`** (มีหลายๆ อย่าง) เพื่อบอกว่าฉันเป็นเจ้าของข้อมูลลูกๆ เหล่านี้นะ
*   **👶 ฝั่งลูก (The Child):** ฝั่งที่เป็น "หลายๆ อย่าง" (เช่น Post) จะใช้เมธอด **`belongsTo`** (เป็นของ...) เพื่อบอกว่าฉันถูกสร้างและเป็นกรรมสิทธิ์ของพ่อแม่คนนี้นะ (Inverse Relationship)
*   **🔑 กุญแจเชื่อมโยง (Foreign Key Magic):** ความฉลาดของ Eloquent คือมันจะเดาชื่อคอลัมน์กุญแจเชื่อมโยงให้เราอัตโนมัติ! โดยมันจะเอาชื่อ Model ฝั่งพ่อแม่ (แบบตัวพิมพ์เล็ก) มาต่อด้วย `_id` เช่น `user_id` ถ้าน้องออกแบบฐานข้อมูลตามกฎนี้ น้องแทบไม่ต้องตั้งค่าอะไรเพิ่มเลย!

{{< figure src="laravel-eloquent-one-to-many-diagram.jpg" alt="System flow diagram illustrating User hasMany Posts and Post belongsTo User" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code):
เรามาดูวิธีการเสกโค้ดให้ User และ Post รู้จักกันครับ

**ขั้นตอนที่ 1: กำหนดความเป็นเจ้าของที่ฝั่ง Parent (`app/Models/User.php`)**
```php
<?php
namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Database\Eloquent\Relations\HasMany;
use App\Models\Post;

class User extends Authenticatable
{
    /**
     * บอกว่า User 1 คน มีได้หลาย Post
     */
    public function posts(): HasMany
    {
        // ร่ายมนต์ hasMany ชี้ไปที่คลาส Post
        return $this->hasMany(Post::class); 
    }
}
```

**ขั้นตอนที่ 2: กำหนดการเป็นกรรมสิทธิ์ที่ฝั่ง Child (`app/Models/Post.php`)**
```php
<?php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;
use App\Models\User;

class Post extends Model
{
    /**
     * บอกว่า Post นี้ เป็นของ User คนไหน
     */
    public function user(): BelongsTo
    {
        // ร่ายมนต์ belongsTo ชี้กลับไปที่คลาส User
        return $this->belongsTo(User::class);
    }
}
```

**ขั้นตอนที่ 3: การดึงข้อมูลมาใช้งานจริง (ใน Controller หรือ Tinker)**
ความมหัศจรรย์มันอยู่ตรงนี้ครับน้องๆ พอเราประกาศความสัมพันธ์เสร็จ เราสามารถเรียกใช้มันผ่าน **"Dynamic Properties"** ได้เลย:

```php
use App\Models\User;
use App\Models\Post;

// 🟢 1. หาบทความทั้งหมดของนาย A
$user = User::find(1);
// สังเกตว่าเราเรียก ->posts (ไม่มีวงเล็บ) มันจะคืนค่ากลับมาเป็น Collection ของบทความทั้งหมด!
foreach ($user->posts as $post) {
    echo $post->title . "<br>"; 
}

// 🔵 2. หาว่าบทความนี้ ใครเป็นคนเขียน? (Inverse)
$post = Post::find(5);
// เรียก ->user จะคืนค่ากลับมาเป็น Object ของ User
echo "บทความนี้เขียนโดย: " . $post->user->name;
```

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips):
ในฐานะ Senior Dev พี่มีเรื่องที่ต้องเตือนน้องๆ อย่างจริงจังเกี่ยวกับการดึงข้อมูล Relationships ครับ!

*   **⚠️ หายนะของ N+1 Query Problem:** 
    ถ้าเราต้องการแสดงรายชื่อบทความทั้งหมด พร้อมชื่อคนเขียน แล้วเราเขียนโค้ดแบบนี้:
    ```php
    $posts = Post::all();
    foreach ($posts as $post) {
        echo $post->user->name; // ตรงนี้แหละคือจุดสลบ!
    }
    ```
    นี่คือการทำ **"Lazy Loading"** ครับ Eloquent จะดึง Post มาก่อน 1 Query จากนั้นพอเข้าลูป ถ้ามี 100 บทความ มันจะยิง Query ไปหา User อีก 100 ครั้ง!! รวมเป็น 101 Queries (N+1) เว็บหน่วงแน่นอน!

*   **🚀 วิธีแก้: ใช้เวทมนตร์ Eager Loading (`with`)**
    เราต้องบอก Eloquent ล่วงหน้าเลยว่า "เฮ้ย ดึง User มาเตรียมไว้รอเลยนะ" โดยใช้เมธอด `with()` แบบนี้ครับ:
    ```php
    // แค่เติม with('user') มันจะใช้การดึงข้อมูลเพียงแค่ 2 Queries เท่านั้น!
    $posts = Post::with('user')->get();
    
    foreach ($posts as $post) {
        echo $post->user->name; // เร็วปรู๊ดปร๊าดเพราะข้อมูลอยู่ใน Memory แล้ว!
    }
    ```
*   **💡 Method vs Property (`posts()` vs `posts`):** 
    จำไว้ว่า ถ้าเรียก `$user->posts` (ไม่มีวงเล็บ) จะได้ข้อมูลเป็นก้อน Collection แต่ถ้าเรียก `$user->posts()` (มีวงเล็บ) มันจะคืนค่าเป็น Query Builder ซึ่งแปลว่าน้องสามารถ **เขียนเงื่อนไขต่อได้!** เช่น ดึงเฉพาะบทความที่ Active: `$user->posts()->where('active', 1)->get();` โคตรหล่อบอกเลย!

## 6. 🏁 บทสรุป (To be continued...):
เป็นอย่างไรกันบ้างครับกับ **One-to-Many Relationship** แค่ประกาศเมธอดสั้นๆ 2 ฝั่ง เราก็สามารถดึงข้อมูลไปมาหากันได้อย่างสวยงาม ไม่ต้องไปยุ่งกับ JOIN SQL ดิบๆ อีกต่อไป แถมยังได้โค้ดที่สะอาดและอ่านง่ายขึ้นเยอะ

แต่ในโลกของฐานข้อมูล ความสัมพันธ์มันมีซับซ้อนกว่านี้ครับ! ถ้าน้องทำระบบ e-Commerce ที่ "สินค้า 1 ชิ้นอยู่ในได้หลายหมวดหมู่" และ "หมวดหมู่ 1 หมวดก็มีสินค้าได้หลายชิ้น" ล่ะ? ในตอนหน้า พี่จะพาไปเจาะลึกความสัมพันธ์ปราบเซียนอย่าง **"Many-to-Many"** และการจัดการ Pivot Table... เตรียมกาแฟให้พร้อม แล้วพบกันครับ!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Automation ให้กับโรงงานของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและติดตั้งระบบแบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p