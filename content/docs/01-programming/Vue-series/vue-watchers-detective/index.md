---
title: "Watchers: นักสืบส่วนตัวที่คอยเฝ้ามองความเปลี่ยนแปลงของข้อมูล"
weight: 120
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Vue3", "Watchers", "Reactivity", "Composition API", "API"]
---
{{< figure src="vue-watchers-detective-cover.jpg" alt="Aesthetic study notes about Vue.js Watchers on an iPad screen" >}}

## 1. 🎯 ตอนที่ 12: Watchers นักสืบส่วนตัวที่คอยเฝ้ามองความเปลี่ยนแปลงของข้อมูล

## 2. 📖 เปิดฉาก (The Hook)
ลองจินตนาการว่าคุณเป็นเจ้าของร้านค้าครับ คุณคงไม่สามารถไปยืนจ้องประตูหน้าร้านตลอดเวลาเพื่อดูว่ามีลูกค้าเดินเข้ามาไหม สิ่งที่คุณทำคือ "ติดกระดิ่ง" ไว้ที่ประตู พอมีคนเปิดประตู (ข้อมูลเกิดการเปลี่ยนแปลง) กระดิ่งก็จะดังเตือนให้คุณหันไปกล่าวคำต้อนรับ (เกิดการกระทำบางอย่าง)

ในโลกของ Vue.js ก็เช่นกันครับ เมื่อแอปพลิเคชันใหญ่ขึ้น บางครั้งเราต้องการให้ระบบ "ทำอะไรบางอย่าง" ทันทีที่ข้อมูลตัวหนึ่งเปลี่ยนไป เช่น พอผู้ใช้พิมพ์คำค้นหาเสร็จปุ๊บ ให้ระบบวิ่งไปดึงข้อมูลจาก Server ปั๊บ! หน้าที่นี้ไม่ใช่หน้าที่ของ Template หรือ Computed ครับ แต่มันคือหน้าที่ของ **Watchers (`watch`)** 

Watchers เปรียบเสมือน **"นักสืบส่วนตัว"** ที่คุณจ้างมาให้เฝ้าจับตาดูตัวแปร (State) ตัวใดตัวหนึ่งโดยเฉพาะ ทันทีที่เป้าหมายขยับตัว นักสืบจะโทรรายงานคุณทันที วันนี้พี่จะพาไปดูวิธีใช้งานนักสืบคนนี้ เพื่อให้แอปพลิเคชันของเราโต้ตอบกับโลกภายนอกได้อย่างสมบูรณ์แบบครับ!

## 3. 🧠 แก่นวิชา (Core Concepts)

### 🕵️‍♂️ Watcher คืออะไร?
`watch` คือฟังก์ชันใน Composition API ที่ใช้สำหรับสังเกตการณ์ (Observe) การเปลี่ยนแปลงของ Reactive Data (เช่น `ref` หรือ `reactive`) เมื่อข้อมูลนั้นถูกแก้ไข มันจะสั่งรัน "Callback Function" ที่เราเตรียมไว้ให้โดยอัตโนมัติ

จุดเด่นของนักสืบคนนี้คือ เขามักจะมารายงานเราด้วยข้อมูล 2 อย่างเสมอ นั่นคือ **ค่าใหม่ (`newValue`)** และ **ค่าเก่า (`oldValue`)** ทำให้เราสามารถเปรียบเทียบได้ว่าเกิดอะไรขึ้น

### ⚔️ Computed vs Watchers (ศึกนี้ต้องเลือกให้ถูก)
แม้ทั้งคู่จะทำงานเมื่อข้อมูลเปลี่ยน แต่วัตถุประสงค์นั้นต่างกันราวฟ้ากับเหว:
*   🧮 **Computed (นักบัญชี):** มีหน้าที่แค่ "คำนวณค่าใหม่ (Derived values)" จากข้อมูลเดิมแบบ Synchronous และต้อง `return` ค่ากลับมาเสมอ **ห้ามทำ Side Effects เด็ดขาด**
*   🕵️‍♂️ **Watchers (นักสืบ):** เกิดมาเพื่อทำ **Side Effects (ผลข้างเคียง)** เช่น การดึงข้อมูลจาก API (Asynchronous), การดัดแปลง DOM โดยตรง, หรือการบันทึกลง LocalStorage 

### 🔍 พลังพิเศษของนักสืบ (Watcher Options)
เราสามารถติดอาวุธเสริมให้ Watcher ได้ด้วย Options พิเศษ:
1.  **`immediate: true` (ทำงานทันที):** ปกติ Watcher จะขี้เกียจ (Lazy) คือรอให้ข้อมูลเปลี่ยนก่อนถึงจะเริ่มรัน แต่ถ้าเราใส่ Option นี้ มันจะรัน Callback ทันทีที่ Component ถูกสร้างขึ้นมาเลย 1 รอบ
2.  **`deep: true` (สืบทะลวงไส้):** ถ้าเราเฝ้าดูข้อมูลที่เป็น Object หรือ Array แบบลึกๆ ปกติ Watcher จะไม่รู้ถ้าข้อมูลข้างใน (Nested properties) ถูกเปลี่ยน การเปิดโหมด `deep` จะบังคับให้ Vue มุดลงไปตรวจจับการเปลี่ยนแปลงในทุกๆ ชั้นของ Object

{{< figure src="vue-watchers-detective-diagram.jpg" alt="Diagram showing the difference between Computed Properties and Watchers" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code)

สถานการณ์คลาสสิกที่สุดของการใช้ Watcher คือ **การดึงข้อมูลจาก API ตาม ID ของผู้ใช้** ลองมาดูโค้ดที่รวมพลังของ Watcher ควบคู่กับการทำงานแบบ Asynchronous (ดึงข้อมูล) กันครับ

```vue
<template>
  <div class="user-card">
    <h2>ค้นหาพนักงาน</h2>
    <!-- ผูกข้อมูลกับ userId -->
    <select v-model="userId">
      <option value="1">พนักงานคนที่ 1</option>
      <option value="2">พนักงานคนที่ 2</option>
      <option value="3">พนักงานคนที่ 3</option>
    </select>

    <hr />

    <!-- แสดงสถานะการโหลด -->
    <div v-if="loading">กำลังสืบประวัติ... 🕵️‍♂️</div>
    <div v-else-if="error" class="error">{{ error }}</div>
    
    <!-- แสดงผลข้อมูลที่ได้จาก API -->
    <div v-else-if="userData">
      <h3>ชื่อ: {{ userData.name }}</h3>
      <p>อีเมล: {{ userData.email }}</p>
    </div>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'

// 1. เป้าหมายที่นักสืบต้องเฝ้าดู (State)
const userId = ref('1')

// 2. State สำหรับเก็บข้อมูลและสถานะหน้าจอ
const userData = ref(null)
const loading = ref(false)
const error = ref(null)

// 3. จ้างนักสืบ (Watch) มาเฝ้าดู userId
watch(
  userId, // เป้าหมาย (Source)
  async (newId, oldId) => { // Callback Function ทำงานเมื่อ userId เปลี่ยน
    console.log(`นักสืบรายงาน: เปลี่ยนการค้นหาจาก ID ${oldId} เป็น ${newId}`)
    
    if (!newId) return;

    loading.value = true
    error.value = null
    
    try {
      // ทำ Side Effect: ยิง API ไปดึงข้อมูล (Asynchronous)
      const response = await fetch(`https://jsonplaceholder.typicode.com/users/${newId}`)
      if (!response.ok) throw new Error('ไม่พบข้อมูลพนักงาน')
      
      userData.value = await response.json()
    } catch (err) {
      error.value = err.message
    } finally {
      loading.value = false
    }
  },
  { immediate: true } // ✨ Option: สั่งให้ดึงข้อมูลคนแรกทันทีที่เปิดหน้าเว็บ!
)
</script>
```

**เกิดอะไรขึ้นในโค้ดนี้?**
ทันทีที่ผู้ใช้เปลี่ยน Dropdown (`userId` เปลี่ยน) ตัว Watcher จะถูกปลุกให้ตื่น มันจะรับค่า `newId` ไปผสมกับ URL แล้วยิง API เพื่อดึงข้อมูลพนักงานคนใหม่มาแสดงผลทันที โค้ดนี้มีความยืดหยุ่นสูงและจัดการ Loading/Error ได้อย่างเป็นธรรมชาติครับ

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips)

**🚨 ข้อควรระวังเรื่องประสิทธิภาพกับ `deep: true`**
Senior Dev หลายคนมักจะเตือนเรื่องการใช้ `deep: true` กับ Object ขนาดใหญ่ (เช่น ข้อมูล User ที่มีฟิลด์ย่อยซ้อนกันหลายสิบชั้น) การสั่งให้ Vue เดินตรวจตราหาความเปลี่ยนแปลงในทุกซอกทุกมุมของ Object (Traversal) เป็นเรื่องที่ **กินทรัพยากร (Performance overhead) สูงมาก**

*วิธีแก้ปัญหา (The Elegant Way):*
แทนที่จะเฝ้าดู Object ตัวใหญ่ทั้งก้อน ให้เราเปลี่ยนไปเฝ้าดู **"เฉพาะ Property ย่อย"** ที่เราสนใจจริงๆ โดยการใช้ **Getter Function**:

```javascript
const formData = reactive({
  user: { name: 'John', age: 30 },
  settings: { theme: 'dark' }
})

// ❌ แบบนี้กินแรงเครื่อง (Watch ทั้งก้อน ต้องใช้ deep: true)
watch(formData, (newVal) => { ... }, { deep: true })

// ✅ แบบมือโปร (สร้าง Getter Function ชี้เป้าไปที่ field ที่ต้องการเลย)
watch(
  () => formData.user.name, 
  (newName, oldName) => {
    console.log('สืบเฉพาะชื่อเท่านั้น แจ่มแมว!', newName)
  }
)
```
การเขียนแบบ Getter `() => formData.user.name` นอกจากจะไม่ต้องใช้ `deep: true` แล้ว ตัว Watcher ยังรู้แน่ชัดว่าต้องทำงานเมื่อ `name` เปลี่ยนเท่านั้น ช่วยเซฟแรงเครื่องไปได้มหาศาลครับ!

## 6. 🏁 บทสรุป (To be continued...)
**Watchers** คือสะพานเชื่อมที่ยอดเยี่ยมระหว่าง "ข้อมูลภายในแอป (Reactive State)" กับ "โลกภายนอก (Side Effects)" ครับ กฎเหล็กที่พี่อยากให้น้องๆ จำไว้คือ: **"ถ้าจะคำนวณข้อมูลเพื่อแสดงผล ให้เรียกใช้ Computed... แต่ถ้าต้องทำแอคชันที่ส่งผลกระทบภายนอก (API, LocalStorage, DOM) ให้เรียกใช้ Watcher"**

ตอนนี้เราได้เรียนรู้วิธีจัดการกับข้อมูลทั้งแบบนิ่งๆ (Refs), แบบคำนวณ (Computed) และแบบติดตามผล (Watchers) กันไปแล้ว ในบทต่อไป เราจะกระโดดเข้าไปในวัฏจักรชีวิตของ Component หรือที่เรียกว่า **"Lifecycle Hooks"** มาดูกันว่าตั้งแต่วินาทีแรกที่ Component เกิดขึ้นมา จนถึงตอนที่มันถูกทำลาย เราสามารถแทรกแซงอะไรมันได้บ้าง รอติดตามเลยครับ!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Web Application หรือ Frontend Architecture ให้กับธุรกิจของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและพัฒนาซอฟต์แวร์แบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p