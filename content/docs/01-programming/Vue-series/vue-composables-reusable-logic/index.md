---
title: "สร้าง Reusable Logic ด้วย Composables: วิธีลด Code ซ้ำซ้อนแบบเซียน"
weight: 300
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Vue3", "Composables", "Composition API", "Frontend Architecture", "Hooks"]
---
{{< figure src="vue-composables-reusable-logic-cover.jpg" alt="Aesthetic study notes about Vue.js Composables and reusable logic on an iPad screen" >}}

## 1. 🎯 ตอนที่ 30: ร่ายมนต์สร้าง Reusable Logic ด้วย Composables เคล็ดวิชาลดโค้ดซ้ำซ้อนแบบเซียน

## 2. 📖 เปิดฉาก (The Hook)
น้องๆ เคยตกอยู่ในวงจร "ก๊อป-วาง (Copy-Paste) มหาภัย" ไหมครับ? 

สมมติว่าเรากำลังทำหน้าเว็บ Dashboard ที่ต้องมีการดึงข้อมูล (Fetch API) จากเซิร์ฟเวอร์ เราก็เลยเขียนโค้ดสร้างตัวแปร `isLoading`, `data`, และ `error` พร้อมกับฟังก์ชันดึงข้อมูล เอาไว้ใน `UserList.vue` พรุ่งนี้หัวหน้าสั่งให้ทำหน้า `ProductList.vue` เราก็... ก๊อปโค้ดชุดเดิมไปแปะ! มะรืนนี้ทำหน้า `Settings.vue` ก็ก๊อปไปแปะอีก! 

ผลคือ แอปของเรามีโค้ดดึง API ซ้ำๆ กันอยู่ 20 ไฟล์ วันดีคืนดีทีม Backend บอกว่า *"เฮ้ย เราเปลี่ยนวิธีรับ Error Message ใหม่นะ"*... ฝันร้ายมาเยือนเลยครับ เราต้องตามไปแก้โค้ดทั้ง 20 ไฟล์!

ปัญหาเหล่านี้นี่แหละครับที่ทำให้ Vue 3 ได้ปลดปล่อยอาวุธลับที่เรียกว่า **"Composables"** (ถ้าใครเคยเขียน React มันก็คือ Custom Hooks นั่นแหละครับ) วันนี้พี่จะพามาจิบกาแฟ และเรียนรู้วิธีการสกัด Logic ที่ใช้บ่อยๆ ออกมาเป็น "แคปซูลอเนกประสงค์" ที่พกไปใช้ที่ไหนก็ได้ โค้ดของเราจะสะอาดขึ้นเป็นกองเลยล่ะ!

## 3. 🧠 แก่นวิชา (Core Concepts)

### 🧩 Composables คืออะไร?
ในบริบทของ Vue.js นั้น **Composables** คือฟังก์ชันที่ใช้ความสามารถของ Composition API เพื่อห่อหุ้ม (Encapsulate) และนำ **"Stateful Logic" (ลอจิกที่มีการจัดเก็บสถานะ)** กลับมาใช้ซ้ำ

ถ้าเราเขียนฟังก์ชันบวกเลขหรือจัดฟอร์แมตวันที่ธรรมดาๆ เราเรียกว่า *Stateless Logic* (ฟังก์ชันทั่วไป) แต่เมื่อไหร่ก็ตามที่ลอจิกนั้นต้องมีการจัดการค่าที่ตอบสนองต่อหน้าจอ (Reactive State อย่าง `ref` หรือ `computed`) และมีการยุ่งเกี่ยวกับ Lifecycle (เช่น `onMounted`, `onUnmounted`) เราจะใช้แพตเทิร์นของ Composables เข้ามาจัดการครับ

### 🧰 เปรียบเทียบให้เห็นภาพ
Composables ก็เหมือนกับ **"กระเป๋าเครื่องมือช่าง (Utility Belt)"** ครับ แทนที่เราจะสร้างเครื่องมือใหม่ทุกครั้งที่เดินเข้าห้อง (Component) เราแค่หยิบกระเป๋าเครื่องมือที่จัดเตรียมไว้อย่างดี (Composables) เข้าไปด้วย ต้องการเช็กพิกัดเมาส์หรอ? หยิบตลับเมตรออกมา! ต้องการดึงข้อมูลหรอ? หยิบโทรศัพท์ออกมา!

{{< figure src="vue-composables-reusable-logic-diagram.jpg" alt="Diagram showing the architecture of Vue Composables encapsulating stateful logic" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code)

เรามาดูการสกัด (Extract) ลอจิกออกไปเป็น Composables 2 ตัวอย่างสุดคลาสสิกกันครับ

### 🐭 ตัวอย่างที่ 1: ระบบติดตามเมาส์ (Mouse Tracker)
สมมติว่าเราอยากรู้ตำแหน่งแกน X, Y ของเมาส์ เพื่อเอาไปทำแอนิเมชัน แทนที่เราจะเขียน Event Listener ลงใน Component ตรงๆ เราสกัดมันออกมาเป็นไฟล์แยกได้เลยครับ

**ไฟล์ `src/composables/useMouse.js` (โรงงานผลิตแคปซูลเมาส์):**
```javascript
import { ref, onMounted, onUnmounted } from 'vue'

// 🌟 กฎเหล็ก: ตั้งชื่อฟังก์ชันให้ขึ้นต้นด้วยคำว่า 'use' เสมอ
export function useMouse() {
  // สร้าง State แบบ Reactive
  const x = ref(0)
  const y = ref(0)

  // ฟังก์ชันอัปเดตค่า State
  function update(event) {
    x.value = event.pageX
    y.value = event.pageY
  }

  // ผูก Event Listener ทันทีที่ Component ถูกวาดเสร็จ
  onMounted(() => window.addEventListener('mousemove', update))
  
  // 🧹 อย่าลืม "ทำความสะอาด (Clean up)" ทุกครั้งเมื่อ Component ถูกทำลาย
  onUnmounted(() => window.removeEventListener('mousemove', update))

  // ส่งคืนค่า State ออกไปให้ชาวโลกใช้งาน
  return { x, y }
}
```

**วิธีนำไปใช้ใน `Component.vue`:**
```vue
<template>
  <div class="tracker-box">
    <!-- นำ x, y มาใช้ได้เลยเหมือนเป็นตัวแปรในไฟล์นี้! -->
    <h2>ตำแหน่งเมาส์ปัจจุบัน: {{ x }}, {{ y }} 🎯</h2>
  </div>
</template>

<script setup>
// นำเข้า Composable ที่เราสร้างไว้
import { useMouse } from '@/composables/useMouse.js'

// เรียกใช้งานและแกะกล่อง (Destructure) ค่าออกมา
const { x, y } = useMouse()
</script>
```

---

### 🌐 ตัวอย่างที่ 2: ระบบจัดการการดึงข้อมูล (Async API Fetcher)
นี่คือพระเอกตัวจริงที่ช่วยลดโค้ดในโปรเจกต์ได้มหาศาลครับ เราจะสร้างตัวจัดการสถานะการดึง API ที่มีครบทั้ง โหลดดิ้ง (Loading) และ เออเร่อ (Error)

**ไฟล์ `src/composables/useFetch.js`:**
```javascript
import { ref } from 'vue'

export function useFetch(url) {
  const data = ref(null)
  const error = ref(null)
  const isLoading = ref(true)

  // เริ่มต้นดึงข้อมูล
  fetch(url)
    .then((res) => res.json())
    .then((json) => (data.value = json))
    .catch((err) => (error.value = err))
    .finally(() => (isLoading.value = false))

  return { data, error, isLoading }
}
```

**วิธีนำไปใช้ใน `Component.vue`:**
```vue
<template>
  <div>
    <!-- จัดการสถานะ UI ได้อย่างสวยงามและสะอาดตา -->
    <div v-if="isLoading">กำลังโหลดข้อมูล... ⏳</div>
    <div v-else-if="error">เกิดข้อผิดพลาด: {{ error.message }} ❌</div>
    <ul v-else>
      <li v-for="item in data" :key="item.id">{{ item.name }}</li>
    </ul>
  </div>
</template>

<script setup>
import { useFetch } from '@/composables/useFetch.js'

// ดึงข้อมูล User โดยใช้ Composable แค่บรรทัดเดียว!
const { data, error, isLoading } = useFetch('https://api.example.com/users')
</script>
```
*เห็นไหมครับ! โค้ดใน Component ของเราเหลือแค่บรรทัดเดียว ลอจิกอันซับซ้อนถูกเก็บกวาดไปซ่อนไว้ใน Composable อย่างหมดจด!*

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips)

มีหลุมพรางและเทคนิคระดับ Senior ที่น้องๆ ต้องรู้เมื่อใช้งาน Composables ครับ:

**💡 1. ธรรมเนียมการตั้งชื่อ (The "use" Convention)**
นักพัฒนาทั่วโลกตกลงกันว่า ชื่อฟังก์ชันของ Composable จะต้องขึ้นต้นด้วยคำว่า **`use`** เสมอครับ (เช่น `useMouse`, `useFetch`, `useAuth`) เพื่อให้เพื่อนร่วมทีมที่มาอ่านโค้ดรู้ทันทีว่า "อ๋อ! ฟังก์ชันนี้มีการเรียกใช้ Reactivity API ของ Vue อยู่ข้างในนะ!"

**🚨 2. State เป็นเอกเทศต่อกัน (Independent State)**
เมื่อ Component A และ Component B เรียกใช้ `useMouse()` พร้อมกัน **แต่ละ Component จะได้รับตัวแปร `x` และ `y` ของใครของมันแยกจากกันโดยสิ้นเชิงครับ!** มันไม่ได้แชร์ข้อมูลก้อนเดียวกันนะ 
*(ถ้าต้องการแชร์ State ให้เหมือนกันทุกหน้า อันนั้นเราต้องไปพึ่งพาระบบ State Management อย่าง Pinia ครับ)*

**✨ 3. Composables ซ้อน Composables ได้นะ!**
ความเจ๋งคือ ในไฟล์ Composable ของเรา เราสามารถอิมพอร์ต Composable ตัวอื่นมาใช้งานร่วมกันได้! (Composition แปลว่าการประกอบร่าง) สิ่งนี้ช่วยให้เราเขียนลอจิกที่ซับซ้อนมากๆ ได้โดยแบ่งย่อยเป็นจิ๊กซอว์ชิ้นเล็กๆ ครับ

## 6. 🏁 บทสรุป (To be continued...)
**Composables** ถือเป็นสิ่งประดิษฐ์ที่ยอดเยี่ยมที่สุดที่มาพร้อมกับ Vue 3 Composition API ครับ มันช่วยแก้ปัญหาฝันร้ายของการแชร์ลอจิกในอดีต (อย่าง Mixins ที่มักจะทำให้เกิดปัญหาชื่อตัวแปรชนกัน) ช่วยให้โค้ดของเรามีสถาปัตยกรรมที่คลีน (Clean Architecture) ดูแลรักษาง่าย และเทสต์แยกส่วนได้อย่างสบายใจ

หากน้องๆ ขี้เกียจเขียน Composables เอง ในชุมชน Vue มีไลบรารีระดับเทพที่ชื่อว่า **VueUse** (https://vueuse.org/) ซึ่งรวบรวม Composables สำเร็จรูปไว้ให้ใช้เป็นร้อยๆ ตัวเลยครับ แนะนำให้ไปลองส่องดู!

ตอนนี้เราจัดระเบียบโค้ดหลังบ้านกันไปอย่างสวยงามแล้ว ในตอนต่อไป เราจะมาคุยเรื่องประสบการณ์ของผู้ใช้งาน (UX) กันบ้าง กับการเสกแอนิเมชันให้การเปลี่ยนหน้าเว็บลื่นไหลเนียนตาด้วยวิชา **Transitions และ Animations** เตรียมตัววาดลวดลายกันได้เลยครับ!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Web Application หรือ Frontend Architecture ให้กับธุรกิจของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและพัฒนาซอฟต์แวร์แบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p