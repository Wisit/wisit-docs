---
title: "พลังของ Slots: ออกแบบ Component ให้ยืดหยุ่นถึงขีดสุด"
weight: 220
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Vue3", "Slots", "Component", "Frontend Architecture", "UI Design"]
---
{{< figure src="vue-slots-flexibility-cover.jpg" alt="Aesthetic study notes about Vue.js Slots and Component Flexibility on an iPad screen" >}}

## 1. 🎯 ตอนที่ 22: พลังของ Slots ปลดล็อกขีดจำกัดการออกแบบ Component

## 2. 📖 เปิดฉาก (The Hook)
น้องๆ ครับ ในตอนที่ผ่านๆ มา เราเรียนรู้วิธีการส่งข้อมูลจากแม่ไปหาลูกด้วย **Props** กันไปแล้วใช่ไหมครับ? Props นั้นยอดเยี่ยมมากเวลาที่เราต้องการส่งข้อมูลประเภท String, Number หรือ Boolean (เช่น `title="สินค้าใหม่"`) 

แต่ในโลกความเป็นจริงของการทำ Frontend... บางครั้งเราไม่ได้อยากส่งแค่ "ข้อความ" ครับ! 

ลองจินตนาการว่าเรากำลังสร้าง Component `BaseModal` (หน้าต่าง Pop-up) เราอยากให้ตรงกลางของ Modal สามารถใส่ **"อะไรก็ได้"** บางหน้าอยากใส่แค่ข้อความธรรมดา บางหน้าอยากใส่รูปภาพพร้อมปุ่มกด หรือบางหน้าอยากยัดตารางลงไปทั้งดุ้น! ถ้าเราจะพยายามส่งโครงสร้าง HTML เหล่านี้ผ่าน Props โค้ดของเราคงเละเทะและดูแลยากสุดๆ

โชคดีที่ Vue.js มีเวทมนตร์ที่เรียกว่า **"Slots (สล็อต)"** ครับ! 
เปรียบเทียบง่ายๆ Component ที่มี Slot ก็เหมือนกับ **"กรอบรูป"** ครับ ตัว Component มีหน้าที่สร้างกรอบที่สวยงาม (โครงสร้างและ CSS) แต่เว้น "ช่องว่าง" ตรงกลางเอาไว้ เพื่อให้ Component แม่เป็นคนตัดสินใจว่าจะเอา "รูปภาพ (HTML Content)" อะไรมาใส่ วันนี้พี่จะพามาจิบกาแฟ และเจาะลึกวิชา Slots ทั้ง 3 รูปแบบที่จะทำให้ Component ของน้องๆ ยืดหยุ่นระดับเทพกันครับ!

## 3. 🧠 แก่นวิชา (Core Concepts)

Slots ใน Vue.js ถูกแบ่งออกเป็น 3 ขั้นความแอดวานซ์ครับ:

### 🕳️ 1. Default Slots (ช่องว่างพื้นฐาน)
เป็นรูปแบบที่เรียบง่ายที่สุด เราแค่เขียนแท็ก `<slot></slot>` เอาไว้ใน Component ลูก (Child) มันจะทำหน้าที่เป็น "หลุมหลบภัย" เมื่อฝั่งแม่ (Parent) เรียกใช้ Component นี้และพิมพ์ HTML อะไรก็ตามแทรกไว้ตรงกลาง มันจะไหลลงไปแทนที่แท็ก `<slot>` ทันที

### 🍱 2. Named Slots (กล่องเบนโตะแบ่งช่อง)
ถ้า Component ของเราซับซ้อนขึ้น เช่น หน้าต่าง Modal ที่มีทั้งส่วน "หัว (Header)", "เนื้อหา (Body)", และ "ท้าย (Footer)" การมีหลุมเดียวคงไม่พอ เราสามารถใช้แท็ก `<slot name="header">` เพื่อตั้งชื่อให้ช่องว่างแต่ละช่องได้ 
ฝั่งแม่ก็จะใช้แท็ก `<template v-slot:header>` (หรือตัวย่อ `#header`) เพื่อบอกว่า "เฮ้! เอา HTML ก้อนนี้ไปใส่ในช่อง header นะ"

### 🔄 3. Scoped Slots (สล็อตทวนกระแส)
นี่คือท่าไม้ตายระดับ Senior ครับ! ปกติแล้วข้อมูลจะไหลจากแม่ลงลูก แต่ในบางกรณี Component ลูกเป็นคนถือข้อมูล (เช่น ลูกทำหน้าที่ดึงข้อมูล Array มาวนลูป) แต่ลูกไม่รู้ว่า "แม่หน้าตาเป็นยังไง" ลูกจึงเอาข้อมูลที่ตัวเองมี โยนใส่สล็อตคืนกลับไปให้แม่ เพื่อให้แม่ตัดสินใจว่าจะวาด UI (Render) ด้วยข้อมูลก้อนนั้นอย่างไร

{{< figure src="vue-slots-flexibility-diagram.jpg" alt="Diagram showing Vue.js Slot architecture: Default, Named, and Scoped Slots" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code)

เรามาดูตัวอย่างการสร้าง **`BaseModal.vue`** ที่ใช้พลังของ **Named Slots** กันครับ รับรองว่าเอาไปใช้จริงได้เลย!

**ชิ้นส่วนลูก: `src/components/BaseModal.vue` (กรอบรูป)**
```vue
<template>
  <div class="modal-backdrop">
    <div class="modal-content">
      
      <!-- 1. ช่องหัวข้อ (Named Slot: header) -->
      <!-- ถ้าแม่ไม่ส่งอะไรมาเลย จะแสดงคำว่า "Information" เป็นค่าเริ่มต้น (Fallback content) -->
      <header class="modal-header">
        <slot name="header">
          <h3>Information</h3> 
        </slot>
      </header>

      <!-- 2. ช่องเนื้อหาหลัก (Default Slot) -->
      <main class="modal-body">
        <slot></slot> 
      </main>

      <!-- 3. ช่องปุ่มกด (Named Slot: footer) -->
      <footer class="modal-footer">
        <slot name="footer">
          <button class="btn" @click="$emit('close')">ปิดหน้าต่าง</button>
        </slot>
      </footer>

    </div>
  </div>
</template>

<script setup>
// แจ้งให้แม่รู้ว่าลูกสามารถกดปิดได้
defineEmits(['close'])
</script>

<style scoped>
/* ละ CSS พื้นหลังสีดำและกรอบสีขาวไว้เพื่อความกระชับ */
.modal-backdrop { background: rgba(0,0,0,0.5); padding: 20px; }
.modal-content { background: white; border-radius: 8px; padding: 20px; }
</style>
```

**ชิ้นส่วนแม่: `src/App.vue` (คนเอารูปมาใส่กรอบ)**
```vue
<template>
  <div class="app">
    <button @click="showModal = true">ลบข้อมูลบัญชี</button>

    <!-- เรียกใช้ Modal -->
    <BaseModal v-if="showModal" @close="showModal = false">
      
      <!-- เล็งไปที่ช่อง Header (ใช้ตัวย่อ #) -->
      <template #header>
        <h2 style="color: red;">⚠️ ยืนยันการลบข้อมูล</h2>
      </template>

      <!-- เล็งไปที่ช่อง Default (เนื้อหาหลัก) -->
      <template #default>
        <p>คุณแน่ใจหรือไม่ว่าต้องการลบข้อมูลทั้งหมด?</p>
        <img src="/warning.png" alt="warning" width="100" />
      </template>

      <!-- เล็งไปที่ช่อง Footer (ปุ่มกดยืนยัน/ยกเลิก) -->
      <template #footer>
        <button @click="showModal = false">ยกเลิก</button>
        <button style="background: red; color: white;">ลบถาวร!</button>
      </template>

    </BaseModal>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import BaseModal from './components/BaseModal.vue'

const showModal = ref(false)
</script>
```
*เห็นไหมครับ! เราไม่ต้องเขียน HTML ซ้ำๆ เลย เพียงแค่มี `BaseModal` ตัวเดียว เราสามารถเนรมิต Modal หน้าตาแบบไหนก็ได้ผ่านการหยอดโค้ดลง Slots!*

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips)

มีกฎเหล็ก 2 ข้อที่น้องๆ มือใหม่มักจะสับสนเวลาใช้งาน Slots พี่ขอเคลียร์ให้กระจ่างตรงนี้เลยครับ:

**🚨 กฎข้อที่ 1: ขอบเขตการมองเห็นตัวแปร (Compilation Scope)**
ท่องประโยคคลาสสิกนี้ไว้เลยครับ: **"ทุกสิ่งที่เขียนในไฟล์แม่ จะมองเห็นตัวแปรของแม่... ทุกสิ่งที่เขียนในไฟล์ลูก จะมองเห็นตัวแปรของลูก"**
*   ถึงแม้ HTML ของเราจะถูกจับไปยัดไว้ใน Slot ของลูก แต่ถ้าเราเขียน `{{ message }}` ไว้ในไฟล์ `App.vue` (แม่) มันก็จะดึงตัวแปร `message` จากไฟล์แม่เสมอครับ มันไม่สามารถแอบไปทะลวงดึงตัวแปรจากไฟล์ลูกได้ (เว้นแต่เราจะใช้ Scoped Slots)

**💡 กฎข้อที่ 2: เนื้อหาสำรอง (Fallback Content)**
เราสามารถใส่เนื้อหา Default เอาไว้ **"ระหว่าง"** แท็ก `<slot>...</slot>` ได้เลยครับ (เหมือนที่พี่ใส่ปุ่ม "ปิดหน้าต่าง" ไว้ในโค้ดด้านบน) เนื้อหาตรงกลางนี้จะถูกแสดงผลก็ต่อเมื่อ **"Component แม่ ไม่ได้ส่งอะไรมาให้เลย"** ถือเป็นเทคนิคการทำ Fallback ที่ทรงพลังมากครับ ช่วยให้ Component เราไม่แหว่งเวลาคนเอาไปใช้แล้วลืมใส่ข้อมูล

## 6. 🏁 บทสรุป (To be continued...)
**Slots** คือพระเอกตัวจริงที่ทำให้เกิด "Component Library" ดังๆ อย่าง Vuetify หรือ Element Plus ขึ้นมาได้บนโลกใบนี้ครับ เพราะมันช่วยแบ่งแยกหน้าที่อย่างชัดเจน: **"ลูกจัดการโครงสร้างหลักและ CSS (Structure & Style) ส่วนแม่รับผิดชอบเนื้อหาที่จะนำมาแสดง (Content)"**

เมื่อน้องๆ ฝึกใช้ Slots จนคล่อง น้องๆ จะสามารถสร้างชิ้นส่วน UI ที่นำกลับมาใช้ซ้ำ (Reusable) ได้เป็นร้อยๆ หน้าโดยที่โค้ดไม่พังเลยล่ะครับ!

ในตอนต่อไป เราจะมาดูวิชาที่เกี่ยวกับการสลับสับเปลี่ยน Component แบบ Real-time หรือที่เรียกว่า **Dynamic Components (`<component :is="...">`)** รับรองว่าจะช่วยลดการเขียน `v-if` ยาวเป็นหางว่าวได้มหาศาลเลย เตรียมตัวให้พร้อมครับ!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Web Application หรือ Frontend Architecture ให้กับธุรกิจของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและพัฒนาซอฟต์แวร์แบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p