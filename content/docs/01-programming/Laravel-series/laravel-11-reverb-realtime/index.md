---
title: "Real-time Marvels: เริ่มต้นกับ Laravel Reverb"
weight: 180
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Laravel 11", "Reverb", "WebSocket", "Real-time", "Broadcasting"]
---

{{< figure src="laravel-11-reverb-realtime-cover.jpg" alt="A glowing holographic network representing Laravel Reverb WebSocket connections" >}}

## 1. 🎯 ชื่อตอน (Title): ตอนที่ 39: Real-time Marvels เสกแอปพลิเคชันให้มีชีวิตด้วย Laravel Reverb (WebSocket Server ตัวจบ!)

## 2. 📖 เปิดฉาก (The Hook):
มาครับน้องๆ ลากเก้าอี้มานั่งจิบกาแฟกัน... น้องๆ เคยทำระบบแชท (Chat) หรือระบบแจ้งเตือน (Notifications) ในเว็บไหมครับ? สมัยก่อนเวลาเราอยากให้หน้าเว็บมันอัปเดตข้อมูลแบบทันทีทันใดโดยที่ลูกค้าไม่ต้องกดปุ่ม F5 (Refresh) เราต้องพึ่งพาสิ่งที่เรียกว่า **WebSockets** 

แต่ความเจ็บปวดคือ การตั้งค่า WebSocket Server ขึ้นมาใช้เองนั้นเป็นเรื่องที่ "โคตรปราบเซียน" ครับ! โปรแกรมเมอร์ส่วนใหญ่เลยยอมแพ้แล้วหันไปใช้บริการภายนอก (Third-party SaaS) อย่าง Pusher หรือ Ably ซึ่งมันก็ใช้ง่ายดีหรอกครับ แต่พอกราฟผู้ใช้งานเราพุ่งสูงขึ้น (Scale) บิลค่าบริการรายเดือนก็พุ่งทะยานตามไปด้วย เล่นเอาบอสปาดเหงื่อเลยทีเดียว

แต่ในที่สุด สวรรค์ก็ทรงโปรด! สถาปนิกของ Laravel ได้เปิดตัวอาวุธหนักชิ้นใหม่ใน **Laravel 11** ที่มีชื่อว่า **"Laravel Reverb"** มันคือ WebSocket Server แบบ First-party ที่เร็วทะลุขีดจำกัด (Blazing-fast) และรองรับการขยายสเกล (Scalable) แบบเต็มตัว ที่สำคัญคือ... มันรันอยู่บนเซิร์ฟเวอร์ของเราเอง ฟรี! ไม่ต้องไปง้อบริการภายนอกอีกต่อไป วันนี้พี่จะพาไปสร้างระบบแชทแบบ Real-time ด้วย Reverb กันครับ เตรียมตัวลุยได้เลย!

## 3. 🧠 แก่นวิชา (Core Concepts):
เพื่อให้น้องๆ เห็นภาพ พี่ขอเปรียบเทียบสถาปัตยกรรมแบบนี้ครับ:

*   **🌐 Broadcasting (การกระจายสัญญาณ):** ใน Laravel การทำ Real-time จะใช้คอนเซปต์ "Broadcasting" ครับ เปรียบเหมือนเรามี "สถานีวิทยุ" เมื่อมีอะไรเกิดขึ้นในระบบหลังบ้าน (Events) เราก็จะกระจายข่าว (Broadcast) ออกไป
*   **📡 WebSockets (คลื่นวิทยุ):** เป็นเทคโนโลยีการสื่อสารแบบสองทาง (Bidirectional) ระหว่าง Server และ Client ทำให้ Server สามารถ "ผลัก (Push)" ข้อมูลไปหาเบราว์เซอร์ได้ทันทีโดยไม่ต้องรอให้เบราว์เซอร์ร้องขอ
*   **🚀 Laravel Reverb (เสาส่งสัญญาณตัวใหม่):** นี่คือพระเอกของเราใน Laravel 11 ครับ มันจะทำหน้าที่เป็นเสาส่งสัญญาณ WebSocket ที่ทำงานด้วย ReactPHP เพื่อจัดการการเชื่อมต่อได้มหาศาล แม้เราจะไม่ต้องใช้ Pusher ของจริงแล้ว แต่ Reverb ถูกออกแบบมาให้สื่อสารด้วย "Pusher Protocol" ทำให้เราใช้เครื่องมือฝั่งหน้าบ้านตัวเดิมได้อย่างแนบเนียน
*   **📢 Laravel Echo (วิทยุรับสัญญาณ):** เป็นไลบรารี JavaScript ฝั่งหน้าบ้าน (Frontend) ที่มีหน้าที่คอยจูนคลื่นรับสัญญาณ (Listen) จากช่องทาง (Channels) ต่างๆ ที่เราปล่อยมาจาก Reverb

{{< figure src="laravel-11-reverb-realtime-diagram.jpg" alt="System flow diagram comparing old external SaaS architecture vs the new built-in Laravel Reverb WebSocket Server" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code):
เรามาลองทำระบบ "แจ้งเตือนข้อความแชท" แบบง่ายๆ กันครับ เริ่มจากติดตั้งจนถึงใช้งานหน้าบ้าน

**สเต็ปที่ 1: ปลุกเสก Reverb เข้าสู่โปรเจกต์**
ใน Laravel 11 การติดตั้งง่ายจนน่าตกใจ เพียงแค่รันคำสั่งเดียวผ่าน Artisan Console:
```bash
php artisan install:broadcasting
```
ระบบจะแอบถามว่าให้ติดตั้ง `laravel/reverb` ด้วยไหม ให้เราตอบตกลงไปเลยครับ คำสั่งนี้จะจัดการสร้างไฟล์คอนฟิก ติดตั้งแพ็กเกจ และอัปเดตไฟล์ `.env` ให้เรามีตัวแปร `REVERB_APP_KEY` ต่างๆ ครบถ้วน

**สเต็ปที่ 2: รัน WebSocket Server ทิ้งไว้**
ตอนนี้เรามีเซิร์ฟเวอร์แยกอีกตัวนึงอยู่ในแอปเราแล้ว ให้เปิด Terminal หน้าต่างใหม่ แล้วพิมพ์คำสั่งนี้เพื่อเดินเครื่อง Reverb (ค่าเริ่มต้นมันจะรันที่พอร์ต 8080):
```bash
php artisan reverb:start
```

**สเต็ปที่ 3: สร้าง Event ส่งข้อความหลังบ้าน (`app/Events/MessageSent.php`)**
เราจะสร้าง Event เพื่อกระจายข่าวสาร โดยใช้คำสั่ง `php artisan make:event MessageSent` จากนั้นใส่โค้ดนี้ลงไป:
```php
<?php
namespace App\Events;

use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

// กฎเหล็ก: ต้องใส่ implements ShouldBroadcast เพื่อให้ Laravel รู้ว่านี่คือการส่งเข้า WebSocket!
class MessageSent implements ShouldBroadcast
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    // ข้อมูลสาธารณะตรงนี้ (Public Properties) จะถูกส่งไปหาเบราว์เซอร์อัตโนมัติ!
    public function __construct(
        public string $user,
        public string $message
    ) {}

    // กำหนดช่องวิทยุที่จะปล่อยสัญญาณออกไป
    public function broadcastOn(): array
    {
        // ส่งเข้าช่องสาธารณะ (Public Channel) ที่ชื่อว่า 'chat-room'
        return [
            new Channel('chat-room'),
        ];
    }
}
```

**สเต็ปที่ 4: ยิงสัญญาณจาก Controller**
ลองสร้าง Route ง่ายๆ เพื่อเทสต์การยิงสัญญาณครับ:
```php
use App\Events\MessageSent;
use Illuminate\Support\Facades\Route;

Route::get('/send-message', function () {
    // เมื่อมีคนเข้า URL นี้ ให้ยิง Event กระจายข้อความ!
    MessageSent::dispatch('พี่วิสิทธิ์', 'สวัสดีทุกคน Reverb โคตรแจ่ม!');
    return 'Message broadcasted!';
});
```

**สเต็ปที่ 5: ติดตั้งและตั้งค่าฝั่งหน้าบ้าน (Laravel Echo)**
ระบบจะติดตั้ง `laravel-echo` และ `pusher-js` ให้เราแล้วในสเต็ปแรก (Reverb ใช้ Pusher protocol คุยกับฝั่ง Client) ให้เราไปที่ไฟล์ `resources/js/bootstrap.js` น้องๆ จะเห็นโค้ดที่คอมเมนต์ไว้ ให้เอาคอมเมนต์ออกครับ:
```javascript
import Echo from 'laravel-echo';
import Pusher from 'pusher-js';

window.Pusher = Pusher;

// ตกแต่งหน้าปัดวิทยุให้ชี้ไปหา Reverb Server ของเรา
window.Echo = new Echo({
    broadcaster: 'reverb',
    key: import.meta.env.VITE_REVERB_APP_KEY,
    wsHost: import.meta.env.VITE_REVERB_HOST,
    wsPort: import.meta.env.VITE_REVERB_PORT,
    wssPort: import.meta.env.VITE_REVERB_PORT,
    forceTLS: (import.meta.env.VITE_REVERB_SCHEME ?? 'https') === 'https',
    enabledTransports: ['ws', 'wss'],
});
```
อย่าลืมรัน `npm run dev` เพื่อคอมไพล์โค้ด JavaScript ด้วยนะ!

**สเต็ปที่ 6: ดักฟังและรับข้อความหน้าเว็บ**
เปิดไฟล์ Blade View ของน้องๆ (เช่น `resources/views/welcome.blade.php`) แล้วเขียนสคริปต์นี้ต่อท้าย (อย่าลืมเรียก `@vite(['resources/js/app.js'])` ด้วยล่ะ):
```javascript
<script type="module">
    // จูนวิทยุไปที่ช่อง 'chat-room'
    window.Echo.channel('chat-room')
        .listen('MessageSent', (e) => {
            // เมื่อมีสัญญาณส่งมา ให้ทำสิ่งนี้!
            console.log("ข้อความใหม่จาก: " + e.user);
            console.log("เนื้อหา: " + e.message);
            alert("📩 " + e.user + " บอกว่า: " + e.message);
        });
</script>
```

**ทดสอบเลย:** เปิดเบราว์เซอร์ไปที่หน้าแรก (ที่มีสคริปต์ Echo) ทิ้งไว้ จากนั้นเปิดแท็บใหม่ยิงไปที่ `/send-message` ถ้าน้องกลับมาดูที่แท็บแรก จะเห็น Alert เด้งขึ้นมาทันทีโดยไม่ต้องรีเฟรชหน้าเลยครับ! เวทมนตร์ของจริง!

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips):
เมื่อระบบของเราต้องรับผู้ใช้งานหลักพันหรือหลักหมื่นคน นี่คือเทคนิคที่สถาปนิกซอฟต์แวร์ต้องรู้ครับ:

*   **🔒 ปกป้องห้องส่วนตัว (Private & Presence Channels):** ถ้าเป็นระบบแชทส่วนตัว เราใช้ `new Channel()` ไม่ได้นะครับ เพราะใครจูนคลื่นมาก็แอบฟังได้หมด! เราต้องเปลี่ยนไปใช้ `new PrivateChannel('chat.' . $roomId)` ซึ่ง Laravel Echo จะทำการวิ่งไปเช็คสิทธิ (Authentication) ที่หลังบ้านเราก่อนว่า User คนนี้มีสิทธิฟังคลื่นนี้หรือเปล่า ระบบปลอดภัย 100%!
*   **🛠️ ทะลุขีดจำกัดด้วย Event Loop (`ext-uv`):** โดยค่าเริ่มต้น Reverb ใช้ `stream_select` ในการรัน Event Loop ซึ่งมักจะถูกจำกัดจำนวนไฟล์ที่เปิดได้ (Open Files) ไว้ที่ประมาณ 1,024 การเชื่อมต่อ ถ้าน้องอยากให้ Server เดียวรับแขกได้หลักหมื่นคน พี่แนะนำให้ลง PHP Extension ที่ชื่อว่า `ext-uv` (`pecl install uv`) Reverb จะเปลี่ยนไปใช้เอนจิ้นนี้อัตโนมัติ ทำให้สเกลทะลุเพดานได้เลยครับ!
*   **📈 Horizontal Scaling ด้วย Redis:** ถ้าน้องมี Reverb Server หลายเครื่อง จะทำยังไงให้คนที่ต่อเครื่อง A พิมพ์แชทไปหาคนที่ต่อเครื่อง B ได้? ง่ายมากครับ! แค่แก้ไฟล์ `.env` แล้วเซ็ต `REVERB_SCALING_ENABLED=true` Reverb จะใช้พลัง Publish/Subscribe ของ Redis ในการตะโกนกระจายข้อความข้ามเครื่องเซิร์ฟเวอร์ให้เอง โคตรล้ำ!
*   **🩺 ตรวจสุขภาพด้วย Laravel Pulse:** ถ้าน้องติดตั้งแพ็กเกจ Laravel Pulse น้องสามารถมอนิเตอร์จำนวนคนออนไลน์ (Connections) และปริมาณข้อความที่วิ่งอยู่ใน Reverb ได้แบบ Real-time จากหน้า Dashboard เลยครับ (แค่ต้องไปเพิ่ม `ReverbConnections::class` ในไฟล์คอนฟิก `pulse.php`)

## 6. 🏁 บทสรุป (To be continued...):
เป็นยังไงกันบ้างครับกับ **Laravel Reverb**? ฟีเจอร์นี้ได้มาอุดช่องโหว่ที่ทำให้หลายคนลังเลที่จะทำระบบ Real-time ในอดีตไปจนหมดสิ้น ตอนนี้การสร้างแอปที่มีความ Interactive สูงๆ ไม่ใช่เรื่องยากและไม่ต้องเสียเงินแพงๆ ให้กับบริการภายนอกอีกต่อไป (Developer Experience ดีงามมากๆ)

จากพื้นฐาน CRUD, การจัดฐานข้อมูล, ยันลอจิกหลังบ้าน คิวงาน และวันนี้มาจนถึงระบบหน้าบ้านแบบ Real-time... ตอนนี้น้องๆ ก็เปรียบเสมือนสถาปนิกที่มีเครื่องมือครบมือแล้วล่ะครับ! นำความรู้ตรงนี้ไปประกอบร่างสร้างสรรค์ผลงานชิ้นเอกกันได้เลย สำหรับซีรีส์ Laravel 11 นี้เราเดินทางกันมาไกลมาก หากมีอัปเดตใหม่ๆ หรือเทคนิคเจ๋งๆ พี่จะนำมาเล่าให้ฟังอีกเรื่อยๆ ครับ ขอให้สนุกกับการเขียนโค้ดนะครับทุกคน!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Automation ให้กับโรงงานของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและติดตั้งระบบแบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p