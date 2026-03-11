---
title: "Lifecycle Hooks: จังหวะชีวิตของ Component ที่ Dev ต้องคุมให้ได้"
weight: 180
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Vue3", "Lifecycle Hooks", "Composition API", "Frontend", "Performance"]
---
{{< figure src="vue-lifecycle-hooks-cover.jpg" alt="Aesthetic study notes about Vue.js Lifecycle Hooks on an iPad screen" >}}

## 1. 🎯 ตอนที่ 18: Lifecycle Hooks จับจังหวะชีวิต Component ตั้งแต่เกิดจนตาย

## 2. 📖 เปิดฉาก (The Hook)
น้องๆ ลองจินตนาการถึง "นักแสดงละครเวที" ดูนะครับ กว่าที่นักแสดงคนหนึ่งจะออกมาแสดงให้เราดูได้ เขาต้องผ่านหลายขั้นตอน: เริ่มตั้งแต่อ่านบทในห้องแต่งตัว -> เดินขึ้นมาบนเวที (ไฟส่อง) -> เปลี่ยนชุดตามฉากที่เปลี่ยนไป -> และสุดท้ายคือการโค้งคำนับแล้วเดินลงจากเวทีไป

Component ใน Vue.js ก็มี "ชีวิต" แบบนั้นเป๊ะเลยครับ! ตั้งแต่ตอนที่มันถูกสร้างขึ้นมา (Initialization) ถูกนำไปแปะบนหน้าจอ (Mounting) มีการอัปเดตข้อมูล (Updating) และถูกทำลายทิ้งเมื่อเราเปลี่ยนหน้าเว็บ (Unmounting) 

ปัญหาคลาสสิกที่มือใหม่มักจะเจอคือ... *"พี่ครับ ทำไมผมดึงกราฟมาวาดแล้วมันพัง? (ก็ไปวาดตอนที่หน้าจอยังไม่เกิดไง!)"* หรือ *"ทำไมเปลี่ยนหน้าเว็บไปแล้ว แต่ระบบนับเวลายังวิ่งอยู่เบื้องหลังจนเบราว์เซอร์ค้าง? (ก็ลืมล้างความจำตอนมันตายไง!)"*

วันนี้พี่จะพามาดูความลับของ **Lifecycle Hooks** ซึ่งเป็นเสมือน "จุดดักฟัง" ที่ให้เราแทรกโค้ดเข้าไปในแต่ละช่วงจังหวะชีวิตของ Component รับรองว่าถ้ารู้ความลับนี้ น้องๆ จะควบคุมแอปพลิเคชันได้ดั่งใจนึกเลยครับ!

## 3. 🧠 แก่นวิชา (Core Concepts)

ในยุคของ Vue 3 Composition API เราจะใช้ฟังก์ชันที่ขึ้นต้นด้วยคำว่า `on...` เพื่อดักจับจังหวะชีวิตเหล่านี้ (โดยต้อง `import` มาจาก `vue` เสมอ) มาดูไทม์ไลน์ชีวิตของมันกันครับ:

### 👶 1. ช่วงกำเนิด (Initialization)
*   **`setup()` (หรือ `<script setup>`):** นี่คือจุดเริ่มต้นของทุกสิ่ง ทำงานก่อนใครเพื่อน เทียบเท่ากับ `beforeCreate` และ `created` ในอดีต ช่วงนี้เรามี State (`ref`, `reactive`) แล้ว แต่ **"หน้าจอ (DOM) ยังไม่เกิด"** ครับ 

### 🌟 2. ช่วงปรากฏตัวบนเวที (Mounting)
*   **`onBeforeMount()`:** ก่อนที่ HTML จะถูกนำไปแปะบนหน้าจอ (แทบไม่ได้ใช้เลย ข้ามไปได้ครับ)
*   **`onMounted()` (สำคัญมาก🔥):** วินาทีที่ Component ปรากฏตัวบนหน้าจอแบบสมบูรณ์! (DOM Nodes ถูกสร้างแล้ว) 
    *   *เหมาะสำหรับ:* การไปดึงแท็ก HTML (เช่น `document.getElementById`), การสั่งเริ่มวาดกราฟ (Charts), การเริ่มจับตำแหน่งแผนที่ (Maps), การดักฟัง Event ของเบราว์เซอร์ (เช่น `window.addEventListener('resize')`) และการยิง API (Fetch Data) เพื่อนำข้อมูลมาแสดงผล

### 🔄 3. ช่วงเปลี่ยนผ่าน (Updating)
*   **`onBeforeUpdate()`:** เมื่อ State เปลี่ยนแปลง แต่หน้าจอยังไม่ทันวาดใหม่ (ใช้เช็คสถานะเก่าก่อนเปลี่ยน)
*   **`onUpdated()`:** เมื่อหน้าจอ (DOM) ถูกวาดใหม่เสร็จเรียบร้อยตาม State ที่เปลี่ยนไป
    *   *เหมาะสำหรับ:* การสั่ง Scroll หน้าจอลงมาล่างสุดเวลามีแชทใหม่เด้งเข้ามา หรือสั่งให้ Library กราฟอัปเดตตัวเองตามข้อมูลใหม่

### 💀 4. ช่วงสิ้นอายุขัย (Unmounting)
*   **`onBeforeUnmount()`:** จังหวะสุดท้ายก่อนที่ Component จะถูกลบทิ้ง (ยังมีชีวิตอยู่ครบถ้วน)
*   **`onUnmounted()` (สำคัญมาก🔥):** Component ถูกทำลายและถอดออกจากหน้าจอแล้ว
    *   *เหมาะสำหรับ:* **"การเก็บกวาดขยะ (Cleanup)"** เช่น การยกเลิกตัวจับเวลา (`clearInterval`), การลบ Event Listener ที่ผูกไว้กับ Window, หรือการทำลาย Instance ของ Library กราฟ เพื่อป้องกันปัญหา **Memory Leak** (อาการสูบแรมเครื่องจนเว็บค้าง)

{{< figure src="vue-lifecycle-hooks-diagram.jpg" alt="Diagram showing the Vue 3 Composition API Lifecycle Hooks timeline" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code)

เรามาดูตัวอย่าง "แผงแสดงราคาสต็อกหุ้น" ที่ต้องยิง API โหลดข้อมูล และมีระบบจับเวลาอัปเดตอัตโนมัติ (Polling) กันครับ โค้ดนี้จะแสดงให้เห็นถึงการใช้ Lifecycle อย่างสมบูรณ์:

```vue
<template>
  <div class="stock-card" ref="cardRef">
    <h2>📈 หุ้นของบริษัท WP Solution</h2>
    <div v-if="loading">กำลังโหลดข้อมูลจากตลาดหลักทรัพย์...</div>
    <div v-else class="price">
      ราคาปัจจุบัน: <strong>{{ price }}</strong> บาท
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, onUpdated } from 'vue'

// 1. ช่วง setup() - กำหนด State เตรียมไว้
const price = ref(0)
const loading = ref(true)
let timerId = null // เก็บ ID ของตัวจับเวลาเอาไว้ทำลายทิ้ง

// ฟังก์ชันจำลองการยิง API
const fetchStockPrice = async () => {
  console.log('Fetching new price...')
  // สมมติว่านี่คือ axios.get('/api/stock')
  setTimeout(() => {
    price.value = Math.floor(Math.random() * 100) + 500
    loading.value = false
  }, 1000)
}

// 2. 🌟 ช่วง onMounted - เมื่อหน้าจอพร้อม
onMounted(() => {
  console.log('✅ Component ถูก Mount แล้ว! DOM เกิดแล้ว!')
  
  // 2.1 ยิง API ครั้งแรกทันที
  fetchStockPrice()
  
  // 2.2 ตั้งเวลาให้ดึงข้อมูลใหม่ทุกๆ 5 วินาที
  timerId = setInterval(fetchStockPrice, 5000)
})

// 3. 🔄 ช่วง onUpdated - เมื่อราคาเปลี่ยนและหน้าจอขยับ
onUpdated(() => {
  console.log(`จอแสดงผลราคาอัปเดตเป็น: ${price.value} บาท`)
})

// 4. 💀 ช่วง onUnmounted - เมื่อเราเปลี่ยนหน้าเว็บหนีไปที่อื่น
onUnmounted(() => {
  console.log('❌ Component กำลังจะตาย... เก็บกวาดขยะด่วน!')
  
  // 🚨 สำคัญมาก: ถ้าไม่ ClearInterval ตรงนี้ มันจะยิง API ไปเรื่อยๆ เป็นผีดิบสูบแรมเครื่อง!
  if (timerId) {
    clearInterval(timerId)
  }
})
</script>

<style scoped>
.stock-card { border: 2px solid #ccc; padding: 20px; border-radius: 8px; }
.price { font-size: 1.5rem; color: #2c3e50; }
</style>
```

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips)

**🚨 หลุมพรางที่ 1: "Infinite Loop แห่งความตายใน `onUpdated`"**
กฎเหล็กระดับ Senior Dev คือ: **ห้ามทำการเปลี่ยนค่า State (Mutate State) ภายใน `onUpdated()` เด็ดขาด!** 
ลองคิดดูนะครับ: ข้อมูลเปลี่ยน -> `onUpdated` ทำงาน -> น้องไปสั่งแก้ข้อมูลในนั้น -> ข้อมูลเปลี่ยนอีก -> `onUpdated` ทำงานซ้ำ -> ลูปนรกเกิดขึ้น หน้าเว็บค้างพังทลายทันที! (ถ้าต้องอัปเดตข้อมูล ให้ไปใช้ Computed หรือ Watchers แทนครับ)

**💡 หลุมพรางที่ 2: ดึง API ตรงไหนดี ระหว่าง `setup` กับ `onMounted`?**
*   **สายดึงไว (Fetch in setup):** น้องสามารถเรียกฟังก์ชันดึง API ไว้ในโค้ดระดับบนสุดของ `<script setup>` ได้เลย (ไม่ต้องรอ `onMounted`) วิธีนี้ดึงข้อมูลได้ไวกว่าเล็กน้อย เพราะไม่ต้องรอสร้าง DOM
*   **สายปลอดภัย (Fetch in onMounted):** หาก API ของน้องต้องใช้ข้อมูลบางอย่างจาก DOM ที่เรนเดอร์แล้ว (เช่น ต้องวัดความสูงของกล่องเพื่อส่งไปให้ API) หรือต้องการยิง API ให้ตรงกับฝั่ง Client ชัวร์ๆ (ในกรณีทำ Server-Side Rendering - SSR) การดึงใน `onMounted` คือคำตอบที่ปลอดภัยที่สุดครับ

**🗑️ หลุมพรางที่ 3: อย่าลืมล้าง Event ของเบราว์เซอร์**
ถ้าตอน `onMounted` น้องไปสั่ง `window.addEventListener('scroll', ...)` เพื่อจับการเลื่อนเมาส์ ตอน `onUnmounted` น้อง **"ต้อง"** สั่ง `window.removeEventListener('scroll', ...)` เสมอนะครับ! ไม่อย่างนั้น พอน้องเปลี่ยนไปเปิดหน้าอื่น Event ตัวนี้จะยังคงแอบรันอยู่เบื้องหลัง ทำให้เว็บอืดลงเรื่อยๆ (Memory Leak)

## 6. 🏁 บทสรุป (To be continued...)
**Lifecycle Hooks** คือจุดที่เราเชื่อมต่อโลกของตัวแปร (State) เข้ากับโลกของหน้าจอ (DOM) ในเวลาที่เหมาะสมที่สุด จำง่ายๆ ครับ: **"เกิด (setup) -> โผล่ (onMounted) -> เปลี่ยน (onUpdated) -> เก็บกวาดก่อนตาย (onUnmounted)"**

เมื่อน้องๆ เข้าใจจังหวะเหล่านี้ การทำความสะอาดโค้ดและจัดการประสิทธิภาพ (Performance) ของเว็บก็จะกลายเป็นเรื่องง่ายดายครับ! ในตอนหน้า เราจะมาดูวิชาขั้นสูงที่เรียกว่า **Slots** ที่จะเปิดโอกาสให้เราโยน HTML เข้าไปแทรกกลาง Component ลูกได้ตามใจชอบ เตรียมตัวให้พร้อมรับความยืดหยุ่นระดับสุดยอดกันเลยครับ!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Web Application หรือ Frontend Architecture ให้กับธุรกิจของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและพัฒนาซอฟต์แวร์แบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p