---
title: "Component Architecture: เปลี่ยนเว็บให้กลายเป็นตัวต่อ Lego"
weight: 150
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Vue3", "Component", "Architecture", "SFC", "Frontend"]
---
{{< figure src="vue-component-architecture-cover.jpg" alt="Aesthetic study notes about Vue Component Architecture on an iPad screen" >}}

## 1. 🎯 ตอนที่ 15: Component Architecture เปลี่ยนเว็บให้กลายเป็นตัวต่อ Lego

## 2. 📖 เปิดฉาก (The Hook)
น้องๆ เคยต่อเลโก้ (Lego) กันไหมครับ? เวลาเราซื้อเลโก้กล่องใหญ่ๆ มาประกอบเป็นปราสาท เราไม่ได้หล่อพลาสติกขึ้นมาเป็นปราสาทก้อนเดียวทั้งก้อน แต่เราใช้ "ตัวต่อชิ้นเล็กๆ" (เช่น ตัวต่อ 4 ปุ่ม, บานหน้าต่าง, ประตู) มาประกอบเข้าด้วยกันจนกลายเป็นโครงสร้างที่ยิ่งใหญ่

การทำเว็บยุคก่อนหน้านี้ เรามักจะเขียนโค้ด HTML, CSS และ JavaScript ยาวเป็นพันๆ บรรทัดยัดรวมกันไว้ในไฟล์เดียว (Monolithic) พอโปรเจกต์ใหญ่ขึ้น โค้ดก็พันกันเป็นสปาเกตตี เวลาลูกค้าสั่งให้แก้ปุ่ม "Add to Cart" ทีหนึ่ง โปรแกรมเมอร์ถึงกับต้องเลื่อนหาโค้ดกันตาแฉะ! 

แต่ในโลกของ Vue.js เรามีสถาปัตยกรรมที่เรียกว่า **"Component-based Architecture"** ที่จะมาช่วยให้เราหั่นหน้าเว็บที่ซับซ้อน ออกเป็น "ชิ้นส่วนตัวต่อ" เล็กๆ ที่ทำงานแยกกันอย่างเป็นอิสระ วันนี้พี่จะพามาดูวิธีคิดแบบ Component และการจัดโครงสร้างไฟล์ตามฉบับปรมาจารย์ (Vue Style Guide) กันครับ!

## 3. 🧠 แก่นวิชา (Core Concepts)

### 🧱 Component คืออะไร?
**Component** คือชิ้นส่วนของ User Interface (UI) ที่ถูกห่อหุ้ม (Encapsulate) การทำงานไว้ในตัวมันเอง ประกอบไปด้วย โครงสร้าง (HTML), หน้าตา (CSS) และลอจิก (JavaScript) 

เมื่อเราแบ่งหน้าเว็บออกเป็น Component เราจะได้รับพลังวิเศษ 2 อย่างครับ:
1.  **Reusability (การนำกลับมาใช้ซ้ำ):** เช่น เราสร้าง Component `Button` ขึ้นมาครั้งเดียว เราสามารถเรียกใช้มัน 100 ครั้งในหน้าเว็บได้เลย ถ้าวันหนึ่งอยากเปลี่ยนสีปุ่ม ก็แค่ไปแก้ที่ไฟล์ Component ต้นฉบับที่เดียว ปุ่มทั้งเว็บก็จะเปลี่ยนตามทันที!
2.  **Maintainability (การดูแลรักษาง่าย):** หากระบบมีปัญหาที่ส่วน "รายชื่อสินค้า" เราก็พุ่งเป้าไปแก้โค้ดที่ไฟล์ `ProductList.vue` ได้เลยโดยไม่ต้องกลัวว่าจะไปทำส่วนอื่นของเว็บพัง

### 🌳 The Component Tree (ต้นไม้แห่งตัวต่อ)
แอปพลิเคชัน Vue แทบทุกตัวจะถูกจัดโครงสร้างเป็น "ต้นไม้" (Tree structure) ครับ โดยมีจุดเริ่มต้นที่ราก แล้วแตกกิ่งก้านออกไป:
*   **Root Component:** เป็นผู้จัดการใหญ่สุด (ส่วนมากชื่อ `App.vue`)
*   **Parent Component (ตัวแม่):** Component ที่ไปเรียกใช้ Component อื่นมาทำงานอยู่ข้างในตัวเอง
*   **Child Component (ตัวลูก):** Component ย่อยที่ถูกเรียกมาใช้งาน (และอาจจะเป็น Parent ของตัวอื่นต่อไปได้อีก)

{{< figure src="vue-component-architecture-diagram.jpg" alt="Diagram showing a monolithic web page broken down into a Vue component tree structure like Lego blocks" >}}

### 📦 Single-File Components (SFC)
ใน Vue 3 เราจะเขียน Component ในรูปแบบไฟล์นามสกุล `.vue` หรือที่เรียกว่า **Single-File Component (SFC)** ซึ่งมันเกิดมาเพื่อแก้ปัญหาคลาสสิกที่ว่า *"การแยกไฟล์ตามประเภท (แยก HTML, แยก CSS, แยก JS) ไม่ได้แปลว่าคุณแยกหน้าที่การทำงานได้ดี"* SFC จึงจับเอาทุกอย่างที่เกี่ยวข้องกันในเชิง UI มารวมไว้ในไฟล์เดียวกัน ทำให้โค้ดมีความเป็นกลุ่มก้อน (Cohesive) สูงมาก

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code)

มาดูวิธีการสร้างและเรียกใช้ Component แบบ Modern ด้วย `<script setup>` กันครับ สมมติเรากำลังทำหน้าเว็บ Dashboard

**ชิ้นส่วนลูก (Child): `src/components/StatCard.vue`**
นี่คือตัวต่อที่เราจะเอาไว้ใช้แสดงการ์ดสถิติต่างๆ
```vue
<template>
  <div class="stat-card">
    <h3>{{ title }}</h3>
    <p class="value">{{ value }}</p>
  </div>
</template>

<script setup>
// รับค่า Props (ข้อมูล) มาจาก Parent
defineProps({
  title: String,
  value: [String, Number]
})
</script>

<style scoped>
/* scoped จะทำให้ CSS นี้มีผลเฉพาะไฟล์นี้ไฟล์เดียว ไม่กระทบที่อื่น! */
.stat-card {
  border: 1px solid #e2e8f0;
  padding: 20px;
  border-radius: 8px;
  text-align: center;
}
.value { font-size: 2rem; font-weight: bold; color: #42b983; }
</style>
```

**ชิ้นส่วนแม่ (Parent): `src/App.vue`**
เราจะนำตัวต่อ `StatCard` มาประกอบร่างที่นี่ครับ
```vue
<template>
  <div class="dashboard">
    <h1>สรุปยอดขายประจำวัน 📊</h1>
    
    <div class="grid">
      <!-- ใช้งาน Component ซ้ำๆ ได้อย่างง่ายดาย! -->
      <StatCard title="ยอดผู้เข้าชม" value="1,500" />
      <StatCard title="ยอดขาย (บาท)" value="45,000" />
      <StatCard title="ออเดอร์ใหม่" value="12" />
    </div>
  </div>
</template>

<script setup>
// แค่ import เข้ามา ก็สามารถเอาไปใช้ใน <template> ได้เลย (พลังของ script setup)
import StatCard from './components/StatCard.vue'
</script>

<style scoped>
.grid { display: flex; gap: 20px; }
</style>
```
เห็นไหมครับ! หน้า `App.vue` ของเราดูสะอาดตามาก อ่านปุ๊บรู้ปั๊บเลยว่าหน้านี้มี Header และมีการ์ดสถิติ 3 ใบ

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips)

การตั้งชื่อไฟล์และการจัดโฟลเดอร์คือ "ศิลปะขั้นสูง" ที่ Senior Dev ให้ความสำคัญมากครับ อ้างอิงจาก **Vue Official Style Guide** นี่คือกฎเหล็กที่คุณควรทำตามเพื่อให้โค้ดดูเป็นมืออาชีพ:

**1. Multi-word Component Names (ชื่อต้องมีมากกว่า 1 คำเสมอ)**
*   **❌ ผิด:** `Button.vue`, `Card.vue` 
*   **✅ ถูก:** `BaseButton.vue`, `ProductCard.vue`
*   *เหตุผล:* เพื่อป้องกันไม่ให้ชื่อ Component ของเราไปซ้ำกับแท็ก HTML มาตรฐานในอนาคต (เพราะแท็ก HTML จะเป็นคำเดียวเสมอ เช่น `<button>`)

**2. PascalCase เสมอเมื่อตั้งชื่อไฟล์**
*   **✅ ถูก:** `UserProfile.vue`, `ShoppingCart.vue` (ตัวพิมพ์ใหญ่ขึ้นต้นคำเสมอ)
*   *เหตุผล:* ช่วยให้ Auto-complete ใน Editor ทำงานได้ดี และสอดคล้องกับการเรียกใช้แบบ `<UserProfile />` ใน Template

**3. Tightly Coupled Components (การตั้งชื่อแบบเกาะกลุ่ม)**
หากคุณมี Component ย่อยที่ถูกสร้างมาเพื่อใช้กับ Parent ตัวใดตัวหนึ่งโดยเฉพาะ ให้ตั้งชื่อโดยใช้ชื่อ Parent นำหน้าครับ:
*   `TodoList.vue` (ตัวแม่)
*   `TodoListItem.vue` (ตัวลูก)
*   `TodoListButton.vue` (ปุ่มของตัวลูก)
*   *เหตุผล:* เวลาไฟล์เรียงกันตามตัวอักษรในโฟลเดอร์ มันจะจัดกลุ่มอยู่ด้วยกันโดยอัตโนมัติ ทำให้ค้นหาง่ายมาก!

**4. จัดโฟลเดอร์ตาม Feature (Domain-driven)**
เมื่อโปรเจกต์ใหญ่ขึ้น อย่าเอาไฟล์ร้อยไฟล์ไปยัดรวมกันในโฟลเดอร์ `components` ให้แบ่งแบบนี้ครับ:
```text
src/
  components/
    base/          # ตัวต่อพื้นฐาน (เช่น BaseButton, BaseInput)
    layout/        # เค้าโครงหลัก (เช่น TheHeader, TheSidebar)
  features/        # แบ่งตามฟีเจอร์ของธุรกิจ
    products/
      components/  # Component เฉพาะของระบบสินค้า (เช่น ProductCard)
```

## 6. 🏁 บทสรุป (To be continued...)
การคิดแบบ **Component Architecture** ไม่ใช่แค่การเขียนโค้ด แต่มันคือ "การออกแบบ (Design)" ครับ ยิ่งน้องๆ หั่น Component ได้อย่างพอดี (ไม่เล็กเกินจนยิบย่อย และไม่ใหญ่เกินจนดูแลยาก) โปรเจกต์ก็จะยิ่งปรับตัวง่าย และทำงานร่วมกันในทีมได้อย่างมีความสุข

ตอนนี้เรามีตัวต่อที่แยกเป็นอิสระแล้ว แต่คำถามถัดไปคือ... แล้วตัวต่อเหล่านี้มัน "คุยกัน" ได้อย่างไร? ถ้าตัวลูกอยากขอบางอย่างจากตัวแม่ หรือตัวแม่จะส่งข้อมูลให้ตัวลูก ต้องทำแบบไหน? 

ในตอนต่อไป เราจะมาเจาะลึกศาสตร์แห่งการสื่อสารของ Component อย่าง **Props (รับข้อมูล) และ Emits (ส่งสัญญาณ)** เตรียมตัวรับมือกับการส่งผ่านข้อมูลกันได้เลยครับ!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Web Application หรือ Frontend Architecture ให้กับธุรกิจของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและพัฒนาซอฟต์แวร์แบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p