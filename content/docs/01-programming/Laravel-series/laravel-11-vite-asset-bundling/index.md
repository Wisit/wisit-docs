---
title: "Vite & Asset Bundling: จัดการ CSS/JS ในโลกสมัยใหม่"
weight: 130
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Laravel 11", "Vite", "Frontend", "Tailwind", "Vue", "React"]
---

{{< figure src="laravel-11-vite-asset-bundling-cover.jpg" alt="A futuristic developer desk showing Vite as a high-speed bullet train transporting code" >}}

## 1. 🎯 ชื่อตอน (Title): ตอนที่ 34: Vite & Asset Bundling รถไฟความเร็วสูงแห่งโลก Frontend บอกลาการรอคอย

## 2. 📖 เปิดฉาก (The Hook):
มาครับน้องๆ ลากเก้าอี้มานั่งจิบกาแฟกัน... ถ้าน้องๆ เคยเขียน Laravel มาตั้งแต่ยุคก่อนๆ น่าจะคุ้นเคยกับเครื่องมือที่ชื่อว่า **Laravel Mix (ซึ่งทำงานอยู่บน Webpack)** ใช่ไหมครับ? ปัญหาความเจ็บปวด (Pain Point) คลาสสิกของยุคนั้นคือ เวลาโปรเจกต์เราเริ่มใหญ่ มีไฟล์ CSS/JS เยอะๆ พอน้องกด Save ไฟล์ปุ๊บ... พัดลมคอมพิวเตอร์จะดังกระหึ่มเหมือนเครื่องบินเจ็ท ✈️ แล้วต้องรอไปอีก 5-10 วินาที กว่าเบราว์เซอร์จะรีเฟรชให้เห็นผลลัพธ์ใหม่

การรอคอยแค่ไม่กี่วินาที แต่ถ้าทำวันละร้อยรอบ มันก็ตัดอารมณ์ศิลปิน (Flow state) ของโปรแกรมเมอร์ได้เลยนะครับ!

จนกระทั่งในโลกของ Frontend สมัยใหม่ มีฮีโร่ตัวใหม่ถือกำเนิดขึ้นมา ชื่อว่า **"Vite"** (อ่านว่า "วีท" เป็นภาษาฝรั่งเศสแปลว่า "รวดเร็ว") ตั้งแต่ Laravel เวอร์ชัน 9 เป็นต้นมาจนถึง **Laravel 11** ปัจจุบัน สถาปนิกของ Laravel ได้ตัดสินใจเปลี่ยนโครงสร้างพื้นฐาน โดยถอด Laravel Mix ออก แล้วนำ Vite เข้ามาเป็นตัวจัดการ Asset Bundling หลักแทน

ผลลัพธ์คืออะไรน่ะหรือครับ? โค้ดของน้องจะถูกคอมไพล์ในระดับ "มิลลิวินาที" เซฟปุ๊บ หน้าเว็บเปลี่ยนปั๊บแบบไม่ต้องกดรีเฟรช! วันนี้พี่จะพาไปดูว่าเวทมนตร์ของ Vite มันทำงานอย่างไร และเราจะต่อมันเข้ากับ Tailwind CSS หรือ Vue/React ได้หรูหราขนาดไหน ลุยกันเลยครับ!

## 3. 🧠 แก่นวิชา (Core Concepts):
การทำงานของ Vite ใน Laravel 11 อาศัยหลักการและเครื่องมือหลักๆ ดังนี้ครับ:

*   **⚡ HMR (Hot Module Replacement):** 
    นี่คือไม้ตายของ Vite! ในขณะที่ Webpack ต้องปั้นไฟล์ทั้งหมดรวมกันใหม่ทุกครั้งที่แก้โค้ด Vite จะใช้ Native ES Modules ของเบราว์เซอร์ เพื่อเปลี่ยนเฉพาะ "ชิ้นส่วน (Module)" ที่น้องเพิ่งแก้ ส่งตรงเข้าเบราว์เซอร์ทันที ทำให้มันเร็วทะลุขีดจำกัด
*   **⚙️ ไฟล์ `vite.config.js`:**
    ศูนย์บัญชาการของ Vite ที่มาแทนที่ `webpack.mix.js` ไฟล์นี้จะบอก Vite ว่า "จุดเริ่มต้น (Entry points)" ของไฟล์ CSS และ JS ของเราอยู่ที่ไหน (เช่น `resources/css/app.css` และ `resources/js/app.js`)
*   **🪄 Blade Directive `@vite`:**
    สะพานเชื่อมระหว่างโลกของ Backend และ Frontend แทนที่เราจะเขียน `<link>` หรือ `<script>` เรียกไฟล์ตรงๆ เราจะใช้คำสั่ง `@vite(['...'])` แปะไว้ที่ `<head>` ของ Blade เมื่ออยู่ในโหมด Dev มันจะวิ่งไปดึงไฟล์จาก HMR Server แต่ถ้าอยู่บน Production มันจะฉลาดพอที่จะชี้ไปหาไฟล์ที่ผ่านการบีบอัดและทำ Versioning ไว้แล้ว!

{{< figure src="laravel-11-vite-asset-bundling-diagram.jpg" alt="System flow diagram comparing old Webpack Mix vs Modern Vite with HMR" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code):
เรามาดูวิธีการใช้งานตั้งแต่ระดับพื้นฐาน ไปจนถึงการประกอบร่างกับ Framework ยอดฮิตกันครับ

**สเต็ปที่ 1: ศูนย์บัญชาการ `vite.config.js`**
ในโปรเจกต์ Laravel 11 เปล่าๆ เราจะมีไฟล์นี้มาให้เลย หน้าตาโคตรคลีน:
```javascript
// ไฟล์ vite.config.js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin'; // ดึงปลั๊กอินของ Laravel มาใช้

export default defineConfig({
    plugins: [
        laravel({
            // กำหนดจุดเริ่มต้นของ Asset เรา
            input: ['resources/css/app.css', 'resources/js/app.js'],
            // รีเฟรชหน้าเว็บอัตโนมัติเมื่อแก้ไฟล์ Blade หรือ Route!
            refresh: true, 
        }),
    ],
});
```

**สเต็ปที่ 2: การเรียกใช้ในหน้า Blade**
เปิดไฟล์ `resources/views/layouts/app.blade.php` ของน้องๆ ขึ้นมา แล้วร่ายมนต์แค่บรรทัดเดียวครับ:
```html
<!DOCTYPE html>
<html>
<head>
    <title>My Awesome App</title>
    <!-- ร่ายมนต์เรียก Vite (มันจะสร้างแท็ก link และ script ให้เองอย่างชาญฉลาด) -->
    @vite(['resources/css/app.css', 'resources/js/app.js'])
</head>
<body>
    <h1>สวัสดีโลกแห่ง Vite!</h1>
</body>
</html>
```

**สเต็ปที่ 3: คำสั่งคุมเครื่องยนต์ (NPM Scripts)**
เวลาเราจะเดินเครื่อง Vite เราจะรันผ่าน Terminal ด้วย 2 คำสั่งหลัก:
```bash
# 🚀 1. โหมด Development: เปิดเซิร์ฟเวอร์ HMR เซฟปุ๊บเว็บเปลี่ยนปั๊บ (ใช้ตอนเขียนโค้ด)
npm run dev

# 📦 2. โหมด Production: บีบอัด (Minify) และลบโค้ดขยะ เอาไปวางในโฟลเดอร์ public/build (ใช้ตอนเอาขึ้นเซิร์ฟเวอร์จริง)
npm run build
```

---
**🎨 โบนัส: การเชื่อมต่อกับ Tailwind CSS**
Tailwind คือเพื่อนซี้ของ Laravel ครับ วิธีเชื่อมกับ Vite นั้นง่ายมาก:
1. ติดตั้งแพ็กเกจ: `npm install -D tailwindcss postcss autoprefixer`
2. สร้างไฟล์คอนฟิก: `npx tailwindcss init -p` (จะได้ไฟล์ `tailwind.config.js` และ `postcss.config.js`)
3. ไปที่ `resources/css/app.css` แล้วใส่โค้ดนี้:
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```
จบ! Vite จะจัดการแปลงคลาส Tailwind ทั้งหมดให้เราอัตโนมัติ

---
**⚛️ โบนัส: การเชื่อมต่อกับ Vue หรือ React**
หากโปรเจกต์ของน้องอยากใช้ Vue หรือ React เข้ามาทำหน้า UI เราแค่ต้องลง "Plugin" เพิ่มให้ Vite รู้จักครับ:

*ตัวอย่างการติดตั้ง React:*
1. ติดตั้ง Plugin: `npm install --save-dev @vitejs/plugin-react`
2. แก้ไข `vite.config.js`:
```javascript
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import react from '@vitejs/plugin-react'; // นำเข้า React Plugin

export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/css/app.css', 'resources/js/app.jsx'], // เปลี่ยนเป็น .jsx
            refresh: true,
        }),
        react(), // เรียกใช้งาน Plugin
    ],
});
```
3. ที่ไฟล์ Blade ตอนเรียกใช้ อย่าลืมเติม `@viteReactRefresh` เข้าไปก่อน `@vite` ด้วยนะครับ:
```html
@viteReactRefresh
@vite(['resources/css/app.css', 'resources/js/app.jsx'])
```

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips):
*   **รูปภาพก็จัดการได้นะ (Static Assets):** 
    ถ้าน้องอยากให้ Vite จัดการอัปเดตเวอร์ชันของรูปภาพในโฟลเดอร์ `resources/images` ด้วย (เพื่อให้เบราว์เซอร์ไม่ติดแคชรูปเก่า) ให้น้องใส่คำสั่ง `import.meta.glob(['../images/**']);` ไว้ในไฟล์ `app.js` ครับ แล้วเวลาเรียกใช้รูปใน Blade ก็ใช้ Facade สุดหล่อ `{{ Vite::asset('resources/images/logo.png') }}` ได้เลย
*   **ปิด Vite ตอนเขียน Test:** 
    เวลาเรารัน Automated Tests (เช่น ใช้ Pest หรือ PHPUnit) บางทีระบบจะพยายามวิ่งไปหาไฟล์ Vite ที่ยังไม่ได้คอมไพล์แล้วเกิด Error (เพราะไม่ได้รัน `npm run dev` ทิ้งไว้) ให้น้องเรียกเมธอด `$this->withoutVite();` ไว้ใน Test ของน้องครับ เพื่อบอกให้มันข้ามการโหลด Asset ไปเลย โคตรมีประโยชน์!
*   **ปัญหา CORS ตอนรันร่วมกับ Sail / Docker:**
    ถ้าน้องใช้ Laravel Sail หรือรันโปรเจกต์ผ่าน IP อื่นที่ไม่ใช่ localhost เบราว์เซอร์อาจจะบล็อก HMR ของ Vite (ขึ้น Error CORS สีแดงเถือกใน Console) น้องสามารถไปตั้งค่า `server: { cors: { origin: ['https://my-app.test'] } }` ในไฟล์ `vite.config.js` เพื่ออนุญาตโดเมนได้ครับ

## 6. 🏁 บทสรุป (To be continued...):
**Vite** คือจิ๊กซอว์ชิ้นสำคัญที่เปลี่ยน Developer Experience (DX) ในฝั่ง Frontend ของคนทำ Laravel ให้ดีขึ้นแบบก้าวกระโดดครับ ความเร็วในการตอบสนองของมันช่วยให้เราไม่ต้องหลุดโฟกัสจากการเขียนโค้ด แถมการตั้งค่า (Configuration) ก็สั้นและเข้าใจง่ายกว่ายุค Webpack เป็นไหนๆ

เมื่อเราสามารถจัดการทั้งฐานข้อมูล, ลอจิกหลังบ้าน, คิวงาน, ระบบ Test, และ Asset หน้าบ้านได้อย่างสมบูรณ์แบบแล้ว... แอปพลิเคชันของเราก็พร้อมที่จะโบยบินสู่โลกกว้าง! ในตอนหน้า (ซึ่งน่าจะเป็นโค้งสุดท้ายแล้ว) พี่จะพาไปดูศิลปะของการ **"Deployment"** เอาโค้ดของเราขึ้นสู่เซิร์ฟเวอร์ Production จริงอย่างมืออาชีพ เตรียมตัวให้พร้อม แล้วพบกันครับ!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Automation ให้กับโรงงานของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและติดตั้งระบบแบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p