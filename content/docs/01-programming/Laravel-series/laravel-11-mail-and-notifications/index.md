---
title: "Mail & Notifications: ส่งอีเมลและแจ้งเตือนหา User"
weight: 170
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Laravel 11", "Mail", "Notifications", "Mailtrap", "Slack"]
---

{{< figure src="laravel-11-mail-and-notifications-cover.jpg" alt="A futuristic developer desk with holographic envelopes representing Mail and Notifications" >}}

## 1. 🎯 ชื่อตอน (Title): ตอนที่ 30: Mail & Notifications ศิลปะแห่งการสื่อสาร แจ้งเตือนผู้ใช้ให้ถูกที่ ถูกเวลา

## 2. 📖 เปิดฉาก (The Hook):
มาครับน้องๆ ลากเก้าอี้มานั่งจิบกาแฟกัน... ลองจินตนาการดูนะครับว่า น้องๆ เข้าไปซื้อของในเว็บ E-Commerce แห่งหนึ่ง กดจ่ายเงินตัดบัตรเครดิตไปหลักหมื่นเรียบร้อย หน้าเว็บขึ้นว่า "จ่ายเงินสำเร็จ" แล้วก็... เงียบกริบ ไม่มีอีเมลยืนยัน ไม่มีแจ้งเตือนในระบบ ไม่มีอะไรเลย! วินาทีนั้นน้องๆ จะรู้สึกอย่างไรครับ? ใจคอไม่ดี คิดว่าโดนโกงแน่ๆ ใช่ไหมล่ะ!

การสื่อสารกับผู้ใช้งาน (Communication) คือหัวใจสำคัญของ User Experience (UX) ที่ดีครับ ในฐานะสถาปนิกซอฟต์แวร์ เราต้องมีระบบแจ้งเตือนที่ไว้ใจได้ 

สมัยก่อนเวลาจะส่งอีเมลด้วย PHP ดิบๆ เราต้องมานั่งเซ็ต Header ปวดหัวกับปัญหาภาษาไทยเป็นภาษาต่างดาว หรือส่งไปแล้วตกถัง Spam แต่ในโลกของ Laravel เรามีอาวุธหนักสองชิ้นคือ **"Mailables"** สำหรับการเขียนจดหมายอีเมลแบบสวยงาม และ **"Notifications"** ที่เปรียบเสมือน "หอกระจายข่าว" ที่ตะโกนครั้งเดียวแต่วิ่งไปดังทั้งใน Email, ระบบ Database, ไปจนถึงแอปแชทอย่าง Slack! วันนี้พี่จะพาไปเจาะลึกระบบเหล่านี้กันครับ

## 3. 🧠 แก่นวิชา (Core Concepts):
ก่อนจะไปเขียนโค้ด เราต้องแยกให้ออกก่อนว่า 2 ตัวนี้ต่างกันอย่างไร:

*   **✉️ Mailables (จดหมายจ่าหน้าซอง):** 
    เปรียบเหมือน "บุรุษไปรษณีย์" มีหน้าที่ส่งอีเมลโดยเฉพาะ เหมาะกับการส่งข้อความที่ซับซ้อน มีไฟล์แนบ (Attachments) หรือมีหน้าตา HTML สวยงาม (Markdown) โดยโครงสร้างคลาสจะแบ่งเป็น `envelope` (จ่าหน้าซอง) และ `content` (เนื้อหาจดหมาย)
*   **📢 Notifications (หอกระจายข่าวหลายมิติ):** 
    เปรียบเหมือน "ระบบประกาศข่าวสาร" เหมาะสำหรับข้อความสั้นๆ เพื่อบอกว่า "เกิดอะไรขึ้นในระบบ" (เช่น มีคนมากดไลก์, จ่ายเงินสำเร็จ) ความเทพคือ มันส่งไปได้หลายช่องทาง (Channels) พร้อมกันในคลาสเดียว ไม่ว่าจะเป็น Email, Database (แจ้งเตือนรูปกระดิ่งในเว็บ), SMS, หรือ Slack
*   **🪤 Mailtrap (กล่องทรายดักจับอีเมล):**
    กฎเหล็กข้อห้ามตอนกำลังพัฒนา (Development) คือ "ห้ามส่งอีเมลหาลูกค้าจริงเด็ดขาด!" เราจึงต้องใช้บริการอย่าง **Mailtrap** หรือ **Mailpit** ซึ่งเป็น SMTP Server จำลอง มันจะดูดซับอีเมลทุกฉบับที่เราเทสต์ส่ง ไม่ให้หลุดออกสู่อินเทอร์เน็ตของจริง ทำให้เราเปิดดูและจัดหน้าตาอีเมลได้อย่างปลอดภัย

{{< figure src="laravel-11-mail-and-notifications-diagram.jpg" alt="System flow diagram comparing Mailable direct path vs Notifications multiple channels" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code):

เรามาเริ่มจากการตั้งค่าเครื่องมือทดสอบ และลองส่งแจ้งเตือน "เมื่อลูกค้าสั่งซื้อสินค้าสำเร็จ" กันครับ

**สเต็ปที่ 1: ติดตั้งกระบะทราย Mailtrap**
ไปสมัครสมาชิกฟรีที่ mailtrap.io เอาค่า Credentials มาใส่ในไฟล์ `.env` ของโปรเจกต์เรา:
```env
# ตั้งค่าให้ Laravel ส่งผ่าน SMTP ของ Mailtrap
MAIL_MAILER=smtp
MAIL_HOST=sandbox.smtp.mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=your_mailtrap_username
MAIL_PASSWORD=your_mailtrap_password
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS="noreply@myshop.com"
MAIL_FROM_NAME="${APP_NAME}"
```

---

### 🔥 พาร์ท A: การสร้างและใช้งาน Mailable (เน้นอีเมลล้วนๆ)
ใช้คำสั่งเสกคลาสขึ้นมา:
```bash
# สร้าง Mailable พร้อมกับไฟล์เทมเพลต Markdown เพื่อให้หน้าตาสวยงามตั้งแต่เกิด!
php artisan make:mail OrderShipped --markdown=emails.orders.shipped
```

เปิดไฟล์ `app/Mail/OrderShipped.php` ขึ้นมาแต่งหน้าซอง:
```php
<?php
namespace App\Mail;

use App\Models\Order;
use Illuminate\Mail\Mailable;
use Illuminate\Mail\Mailables\Content;
use Illuminate\Mail\Mailables\Envelope;

class OrderShipped extends Mailable
{
    // รับค่า Order เข้ามาเก็บไว้ (ใช้ Constructor Property Promotion ของ PHP 8)
    public function __construct(public Order $order) {}

    // จ่าหน้าซอง (Subject)
    public function envelope(): Envelope
    {
        return new Envelope(
            subject: 'สินค้าของคุณกำลังเดินทางไปหาแล้ว!',
        );
    }

    // กำหนดไฟล์แสดงผล (ใช้ Markdown ที่สร้างไว้)
    public function content(): Content
    {
        return new Content(
            markdown: 'emails.orders.shipped',
        );
    }
}
```

และเวลาจะใช้งานใน Controller ก็ส่งง่ายๆ แบบนี้เลยครับ:
```php
use Illuminate\Support\Facades\Mail;
use App\Mail\OrderShipped;

// ส่งหา Email ของ User คนนี้!
Mail::to($request->user())->send(new OrderShipped($order));
```

---

### 🚀 พาร์ท B: ยกระดับด้วย Notifications (ส่งทีเดียว ออกทุกช่องทาง)
คราวนี้เราจะใช้หอกระจายข่าวบ้าง สมมติเราอยากแจ้งเตือนไปที่ **Email**, เก็บลง **Database** (ให้ขึ้นแจ้งเตือนหน้าเว็บ), และยิงเข้าห้อง **Slack** ของแอดมิน!

```bash
php artisan make:notification OrderCompleted
```

เปิดไฟล์ `app/Notifications/OrderCompleted.php` แล้วร่ายมนต์ตามนี้:
```php
<?php
namespace App\Notifications;

use Illuminate\Notifications\Notification;
use Illuminate\Notifications\Messages\MailMessage;
use Illuminate\Notifications\Slack\SlackMessage;

class OrderCompleted extends Notification
{
    public function __construct(public $order) {}

    // 1. กำหนดว่าจะยิงไปช่องทางไหนบ้าง (Channels)
    public function via(object $notifiable): array
    {
        return ['mail', 'database', 'slack']; 
    }

    // 2. รูปแบบของ Email
    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
            ->subject('คำสั่งซื้อเสร็จสมบูรณ์')
            ->greeting('สวัสดีครับคุณ ' . $notifiable->name)
            ->line('เราได้รับคำสั่งซื้อหมายเลข ' . $this->order->id . ' เรียบร้อยแล้ว')
            ->action('ดูรายละเอียดคำสั่งซื้อ', url('/orders/'.$this->order->id))
            ->line('ขอบคุณที่อุดหนุนร้านของเราครับ!');
    }

    // 3. รูปแบบที่จะเก็บลงตาราง Database (แปลงเป็น JSON)
    public function toArray(object $notifiable): array
    {
        return [
            'order_id' => $this->order->id,
            'amount' => $this->order->total,
            'message' => 'มีคำสั่งซื้อใหม่เข้ามา!'
        ];
    }

    // 4. รูปแบบที่จะยิงเข้า Slack
    public function toSlack(object $notifiable): SlackMessage
    {
        return (new SlackMessage)
            ->text('🎉 เย้! มีลูกค้าสั่งซื้อสินค้าเข้ามาใหม่ มูลค่า ฿' . $this->order->total);
    }
}
```

**วิธีสั่งให้หอกระจายข่าวทำงาน:** 
Model `User` ของ Laravel นั้นมี Trait ที่ชื่อว่า `Notifiable` ติดตั้งมาให้แล้ว เราจึงสั่งงานผ่าน Object ของ User ได้เลย!
```php
use App\Notifications\OrderCompleted;

// สั่งให้ User คนนี้ รับรู้ข่าวสารนี้ซะ!
$user->notify(new OrderCompleted($order));
```

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips):
ในสเกลงานระดับ Production นี่คือจุดตายที่น้องๆ ห้ามพลาดครับ:

*   **⏳ อย่าให้ลูกค้าต้องรอ (ShouldQueue):** การติดต่อกับ SMTP Server เพื่อส่งอีเมล อาจใช้เวลา 2-5 วินาที ถ้าน้องเขียนโค้ดส่งสดๆ (Synchronous) หน้าเว็บจะค้างจนกว่าอีเมลจะส่งเสร็จ! **ทางแก้คือ:** ให้เพิ่ม `implements ShouldQueue` เข้าไปที่หัวคลาส Mailable หรือ Notification ของน้องครับ Laravel จะโยนงานนี้ไปทำเป็น Background Process อัตโนมัติ หน้าเว็บลูกค้าจะโหลดเสร็จในพริบตา!
*   **🗄️ การเตรียม Database ให้รองรับ Notification:** ถ้าน้องเลือกใช้ Channel `database` แบบโค้ดด้านบน น้องต้องสร้างตารางมารองรับมันก่อนนะครับ โดยพิมพ์ร่ายมนต์คำสั่ง `php artisan make:notifications-table` แล้วตามด้วย `php artisan migrate` ระบบจะสร้างตาราง `notifications` ไว้เก็บประวัติการอ่าน (Read/Unread) ให้ฟรีๆ!
*   **🛠️ ไม้ตาย On-Demand Notifications:** บางทีเราอยากส่งแจ้งเตือนหรืองานแอดมิน โดยที่คนๆ นั้นไม่ได้อยู่ในตาราง `users` ของเรา (เช่น ส่งแจ้งเตือนเข้า Email หรือ Slack ของตัวเราเอง) เราสามารถใช้ `Notification::route()` ได้เลยครับ เช่น `Notification::route('mail', 'admin@shop.com')->notify(new SystemAlert());` หล่อและสะดวกมาก!

## 6. 🏁 บทสรุป (To be continued...):
จะเห็นได้ว่า Laravel ออกแบบ **Mail และ Notifications** มาได้อย่างหมดจดงดงามจริงๆ ครับ การใช้ Mailable ช่วยให้เราจัดการความซับซ้อนของอีเมลได้ดี ในขณะที่ Notifications ก็ช่วยให้การเขียนโค้ดสื่อสารหลายๆ ช่องทาง (Omni-channel) จบลงได้ในคลาสเดียว ช่วยยกระดับระบบของเราให้ดูเป็นมืออาชีพสุดๆ 

แต่อย่างที่พี่เกริ่นไว้ในทริคข้อแรก... การส่งอีเมลมัน "กินเวลา" และเราควรโยนมันไปทำงานอยู่เบื้องหลัง! ในตอนต่อไป พี่จะพาไปเจาะลึกระบบจัดการเบื้องหลังที่ว่า นั่นก็คือสุดยอดระบบ **"Job Queues (คิวงาน)"** มาดูกันว่าแอปพลิเคชันระดับโลกเขารับมือกับงานหนักๆ โดยที่เว็บไม่ล่มและไม่ค้างได้อย่างไร... ชงกาแฟแก้วใหม่ แล้วพบกันครับ!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Automation ให้กับโรงงานของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและติดตั้งระบบแบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p