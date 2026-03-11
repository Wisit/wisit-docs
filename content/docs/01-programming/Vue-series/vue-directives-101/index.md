---
title: "Directives 101: คำสั่งวิเศษ v-bind และ v-model ที่จะทำให้ชีวิตคุณง่ายขึ้น"
weight: 80
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Vue3", "Directives", "v-model", "v-bind", "Frontend"]
---
{{< figure src="vue-directives-101-cover.jpg" alt="Aesthetic study notes about Vue Directives v-bind and v-model on an iPad screen" >}}

## 1. 🎯 ตอนที่ 8: Directives 101 สยบความวุ่นวายของฟอร์มด้วย v-bind และ v-model

## 2. 📖 เปิดฉาก (The Hook)
น้องๆ เคยต้องทำหน้า "แบบฟอร์มลงทะเบียน" ด้วย JavaScript แบบดั้งเดิม (Vanilla JS) ไหมครับ? ขั้นตอนมันจะน่าปวดหัวประมาณนี้: 
1. สั่ง `document.getElementById()` เพื่อดึงค่าจากช่อง Input
2. ต้องมานั่งเขียน `addEventListener('input', ...)` เพื่อคอยดักฟังว่าผู้ใช้พิมพ์อะไรลงไปบ้าง
3. พอได้ค่ามาแล้ว ก็ต้องเอาค่าไปยัดใส่ `<p>` หรือ `<img>` เพื่อแสดงผลกลับไปที่หน้าจออีก

แค่คิดก็เหนื่อยแล้วใช่ไหมครับ! โค้ดของเราจะเต็มไปด้วยการจับโยง DOM จนพันกันเป็นสปาเกตตี แต่ในโลกของ Vue.js เรามีเวทมนตร์ที่เรียกว่า **Directives** ครับ มันคือ "คำสั่งพิเศษ" ที่เราแปะลงไปในแท็ก HTML เพื่อบอกให้ Vue จัดการเชื่อมข้อมูล (Data Binding) ให้เราแบบอัตโนมัติ วันนี้พี่จะพามาดู 2 คำสั่งที่ถูกใช้งานบ่อยที่สุดในจักรวาล Vue นั่นก็คือ `v-bind` และ `v-model` ครับ จิบกาแฟให้พร้อม แล้วมาดูความมหัศจรรย์นี้กัน!

## 3. 🧠 แก่นวิชา (Core Concepts)

คำว่า **Directives** ใน Vue.js จะสังเกตง่ายมากครับ เพราะมันจะขึ้นต้นด้วยคำว่า `v-` เสมอ (เช่น `v-if`, `v-for`, `v-bind`) หน้าที่ของมันคือการใส่ "พฤติกรรมอัจฉริยะ" ให้กับ HTML ธรรมดาๆ

### ➡️ 1. `v-bind` (ท่อส่งน้ำทางเดียว / One-Way Binding)
ปกติแล้ว ค่าที่อยู่ใน Attribute ของ HTML เช่น `src="..."`, `href="..."`, `class="..."` มันจะเป็นแค่ข้อความคงที่ (Static) เปลี่ยนแปลงไม่ได้ แต่ถ้าเราใส่ `v-bind:` เข้าไปข้างหน้า มันจะกลายเป็น "ท่อส่งน้ำ" ที่รับเอาตัวแปรจาก Script (State) ไหลมาแสดงผลที่ Attribute นั้นๆ ทันที
*   **ทิศทาง:** Script -> Template (เปลี่ยนแปลงที่โค้ด หน้าจอเปลี่ยนตาม)
*   **ตัวย่อ (Shorthand):** เนื่องจากเราใช้บ่อยมาก Vue จึงให้เราย่อคำว่า `v-bind:` เหลือแค่เครื่องหมายโคลอน `:` ตัวเดียวได้เลยครับ (เช่น `:src="imageUrl"`)

### 🔄 2. `v-model` (กระจกสะท้อนเงา / Two-Way Binding)
นี่คือพระเอกตัวจริงของการทำแบบฟอร์ม (Forms) ครับ! `v-model` คือการผูกข้อมูลแบบ "สองทิศทาง" 
*   **ทิศทาง:** Script <-> Template 
*   **อธิบายให้เห็นภาพ:** ถ้าเราพิมพ์ข้อความลงในช่อง Input (`<input>`) บนหน้าจอ ตัวแปรในโค้ดของเราก็จะเปลี่ยนตามทันที! และในทางกลับกัน ถ้าเราสั่งเคลียร์ตัวแปรในโค้ด ช่อง Input บนหน้าจอก็จะว่างเปล่าทันทีเช่นกัน! มันออกแบบมาให้ใช้กับฟอร์มโดยเฉพาะ ไม่ว่าจะเป็น Text Input, Checkbox, Radio Button หรือ Dropdown Select

{{< figure src="vue-directives-101-diagram.jpg" alt="Diagram showing the difference between One-way data binding (v-bind) and Two-way data binding (v-model)" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code)

มาดูตัวอย่างการสร้าง "การ์ดโปรไฟล์ผู้ใช้" ที่ให้ผู้ใช้กรอกข้อมูลแล้วหน้าจออัปเดตแบบ Real-time กันครับ เราจะใช้ทั้ง `v-bind` และ `v-model` ร่วมกัน

```vue
<template>
  <div class="profile-card">
    <!-- 1. ใช้ v-bind (ตัวย่อคือ :) ผูก URL รูปภาพเข้ากับ src attribute -->
    <img :src="profileImage" alt="Profile Picture" class="avatar" />
    
    <h2>สวัสดีคุณ: {{ username }}</h2>
    <p>สถานะ: {{ role }}</p>

    <hr />
    
    <div class="form-group">
      <label>เปลี่ยนชื่อของคุณ:</label>
      <!-- 2. ใช้ v-model ผูกช่องกรอกข้อความเข้ากับตัวแปร username -->
      <!-- พิมพ์ปุ๊บ ข้อความใน <h2> ด้านบนจะเปลี่ยนปั๊บ! -->
      <input type="text" v-model="username" placeholder="กรอกชื่อที่นี่..." />
    </div>

    <div class="form-group">
      <label>เลือกบทบาท:</label>
      <!-- 3. v-model ใช้กับ dropdown <select> ได้เนียนๆ -->
      <!-- Vue จะรู้ทันทีว่าคุณเลือก option ไหน แล้วเอาไปใส่ในตัวแปร role -->
      <select v-model="role">
        <option value="มือใหม่หัดโค้ด">มือใหม่หัดโค้ด</option>
        <option value="Senior Developer">Senior Developer</option>
      </select>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

// ประกาศ Reactive State (ตัวแปรที่เชื่อมกับหน้าจอ)
const profileImage = ref('https://api.dicebear.com/7.x/avataaars/svg?seed=VueMaster')
const username = ref('นักพัฒนาปริศนา')
const role = ref('มือใหม่หัดโค้ด')
</script>

<style scoped>
.profile-card { border: 1px solid #ccc; padding: 20px; border-radius: 8px; width: 300px; }
.avatar { width: 100px; height: 100px; border-radius: 50%; }
.form-group { margin-top: 15px; }
input, select { width: 100%; padding: 8px; margin-top: 5px; }
</style>
```

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips)

เมื่อน้องๆ เริ่มเขียน Vue ไปสักพัก นี่คือ "ความลับ" ที่ Senior Dev ทุกคนต้องรู้เพื่อไม่ให้ตกหลุมพรางครับ:

**🚨 ความลับที่ 1: `v-model` แท้จริงแล้วคือ "ภาพลวงตา" (Syntactic Sugar)**
Vue นั้นฉลาดมากครับ การที่เราพิมพ์แค่ `v-model="username"` เบื้องหลังการทำงาน (Under the hood) Vue จะแปลงร่างโค้ดนั้นให้กลายเป็นการใช้ `v-bind` และ `v-on` (Event Listener) ประกอบร่างกันครับ:
```html
<!-- สิ่งที่เราเขียน -->
<input v-model="username" />

<!-- สิ่งที่ Vue ทำงานจริงๆ เบื้องหลัง -->
<input 
  :value="username" 
  @input="username = $event.target.value" 
/>
```
*เห็นไหมครับ! แท้จริงแล้ว `v-model` เป็นแค่ทางลัด (Shortcut) ที่เอาไว้ผูก `:value` เข้ากับช่องกรอก และคอยดักจับ Event `@input` ให้เราโดยอัตโนมัติเท่านั้นเอง!*

**✨ ความลับที่ 2: ของวิเศษที่เรียกว่า "Modifiers"**
เวลาเรากรอกฟอร์ม บางครั้งเราต้องการจัดระเบียบข้อมูลก่อนเข้าตัวแปร `v-model` มีตัวช่วยเสริม (Modifiers) ที่ต่อท้ายด้วยจุด (`.`) ให้เราใช้สบายๆ ครับ:
*   `v-model.trim="..."` : ช่วยตัดช่องว่าง (Spacebar) ที่เผลอพิมพ์เกินมาข้างหน้าและข้างหลังข้อความให้ทิ้งไปโดยอัตโนมัติ
*   `v-model.number="..."` : ค่าที่รับจากช่อง `<input>` มักจะเป็น String เสมอ ถ้าเราอยากให้มันแปลงเป็นตัวเลข (Number) อัตโนมัติ (เช่น ฟอร์มกรอกอายุ) ให้ใส่ตัวนี้ครับ
*   `v-model.lazy="..."` : ปกติ `v-model` จะอัปเดตแบบ Real-time ทุกตัวอักษรที่เรากดคีย์บอร์ด แต่ถ้าใส่ `.lazy` มันจะรอจนกว่าผู้ใช้จะคลิกเมาส์ออกจากช่อง (Blur / Change event) แล้วค่อยอัปเดตข้อมูลทีเดียว (เหมาะสำหรับลดภาระการทำงานหนักๆ)

## 6. 🏁 บทสรุป (To be continued...)
แค่มี `v-bind` และ `v-model` ชีวิตการทำ Frontend ของเราก็ง่ายขึ้นไปกว่า 80% แล้วครับ! เราไม่ต้องไปแตะต้อง DOM ตรงๆ อีกต่อไป แค่โฟกัสที่ "ข้อมูล (State)" แล้วปล่อยให้ Vue จัดการดูแลหน้าจอให้เราแทน นี่แหละครับคือพลังของ Declarative Rendering อย่างแท้จริง

ในตอนถัดไป เราจะมาเจาะลึกวิธีการรับมือกับ "การกระทำของผู้ใช้" กันบ้าง (เช่น การคลิกปุ่ม, การกด Enter) ด้วยพระเอกอีกคนที่ชื่อว่า `v-on` (หรือ `@`) เตรียมตัวร่ายมนต์โต้ตอบกับผู้ใช้กันได้เลยครับ!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Web Application หรือ Frontend Architecture ให้กับธุรกิจของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและพัฒนาซอฟต์แวร์แบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p