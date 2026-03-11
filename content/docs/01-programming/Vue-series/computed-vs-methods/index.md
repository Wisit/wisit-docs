---
title: "Computed Properties vs Methods: ใช้ต่างกันอย่างไร? เลือกผิดชีวิตเปลี่ยน"
weight: 110
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Vue3", "Computed", "Methods", "Performance", "Composition API"]
---
{{< figure src="computed-vs-methods-cover.jpg" alt="Aesthetic study notes about Computed Properties vs Methods in Vue.js on an iPad screen" >}}

## 1. 🎯 ตอนที่ 11: ศึกตัดสิน Computed vs Methods เมื่อ Caching คือเส้นแบ่งระหว่างเว็บลื่นกับเว็บอืด

## 2. 📖 เปิดฉาก (The Hook)
น้องๆ เคยเจอเหตุการณ์นี้ไหมครับ? หน้าเว็บของเรามีลิสต์ข้อมูลยาวเหยียดที่ต้องเอามาคำนวณหรือกรอง (Filter) ตอนแรกที่ข้อมูลยังน้อยๆ มันก็ใช้งานได้ปกติดี เราเลยเอา Logic การคำนวณทั้งหมดไปยัดไว้ใน `Methods` แล้วเรียกใช้ใน Template โต้งๆ เลย 

แต่พอดันขึ้น Production ข้อมูลเริ่มเยอะขึ้น ปรากฏว่าแค่ผู้ใช้กดปุ่ม "เพิ่มจำนวนสินค้า" (ซึ่งไม่เกี่ยวอะไรกับลิสต์นั้นเลย) หน้าเว็บกลับกระตุกและค้างไปชั่วขณะ! 

นี่คือหลุมพรางคลาสสิกที่ Front-end Developer มือใหม่มักจะตกลงไปครับ เพราะในโลกของ Vue.js การเลือกระหว่าง **Computed Properties** กับ **Methods** ในการคำนวณข้อมูลเพื่อแสดงผลนั้น **"เลือกผิด ชีวิต(เว็บ)เปลี่ยน"** จริงๆ 

วันนี้พี่จะพามานั่งจิบกาแฟ แล้วชำแหละให้ดูชัดๆ ว่าสองตัวนี้มันต่างกันอย่างไร และทำไมคำว่า **"Caching"** ถึงเป็นเวทมนตร์ที่ช่วยชีวิตแอปพลิเคชันของคุณไว้ครับ!

## 3. 🧠 แก่นวิชา (Core Concepts)

หากมองเผินๆ เวลาเราเรียกใช้ `computed` หรือ `methods` ในหน้า Template เพื่อแสดงผลลัพธ์บางอย่าง (เช่น การนำชื่อและนามสกุลมาต่อกัน หรือกรองข้อมูลสินค้า) ผลลัพธ์ที่ได้ออกมาบนหน้าจอ **"มันเหมือนกันเป๊ะ"** ครับ 

แต่เบื้องหลังการทำงาน (Under the hood) นั้น แตกต่างกันราวฟ้ากับเหว:

### 👨‍🍳 Methods: "เชฟผู้ขยันขันแข็ง (แต่ไม่จำอะไรเลย)"
เวลาเราใช้ฟังก์ชันจาก Methods ไปแสดงผลใน Template (เช่น `{{ getFilteredList() }}`) ทุกครั้งที่หน้าจอเกิดการ Re-render (ไม่ว่าจะเกิดจากการเปลี่ยน State ตัวไหนก็ตาม) **Vue จะสั่งให้ Methods รันโค้ดใหม่ตั้งแต่บรรทัดแรกเสมอ** 
*   **ข้อดี:** ได้ข้อมูลสดใหม่เสมอ รับ Parameter ได้
*   **ข้อเสีย:** เปลืองแรง CPU หากเป็นการคำนวณที่หนักหน่วง (Expensive Operation) 

### 🧮 Computed Properties: "นักบัญชีอัจฉริยะ (ที่มีสมุดจด Caching)"
`computed` ถูกออกแบบมาเพื่อสร้าง "Derived State" (ข้อมูลที่ได้มาจากการคำนวณข้อมูลอื่น) ความฉลาดของมันคือระบบ **Caching (การแคช/จดจำผลลัพธ์)**
*   มันจะรันโค้ดคำนวณ **แค่ครั้งแรกครั้งเดียว** จากนั้นมันจะจดจำผลลัพธ์ไว้
*   มันจะยอมคำนวณใหม่อีกครั้ง ก็ต่อเมื่อ **Reactive Dependencies (ตัวแปรต้นทางที่มันพึ่งพาอยู่) มีการเปลี่ยนแปลงเท่านั้น**! 
*   ถ้า State อื่นๆ ในหน้าเว็บเปลี่ยน แต่ State ที่มันเฝ้าดูอยู่ไม่ได้เปลี่ยน มันจะหยิบ "ค่าเดิมที่จำไว้" โยนกลับไปให้ทันทีโดยไม่ประมวลผลซ้ำ

{{< figure src="computed-vs-methods-diagram.jpg" alt="Diagram showing the performance comparison between Vue.js Computed Properties and Methods" >}}

### 📊 ตารางเปรียบเทียบ: ตัดสินใจอย่างไรดี?

| คุณสมบัติ | 🧮 Computed Properties | 👨‍🍳 Methods |
| :--- | :--- | :--- |
| **ระบบ Caching** | ✅ **มี** (จำค่าไว้จนกว่า Dependency จะเปลี่ยน) | ❌ **ไม่มี** (รันใหม่ทุกครั้งที่ Re-render) |
| **การเรียกใช้ใน Template** | เรียกแบบตัวแปร: `{{ fullName }}` | เรียกแบบฟังก์ชัน: `{{ getFullName() }}` |
| **การรับ Parameters** | ❌ **ไม่ได้** (รับค่าผ่าน State ที่พึ่งพาเท่านั้น) | ✅ **ได้** (เช่น `{{ formatPrice(100) }}`) |
| **เหมาะสำหรับ...** | การคำนวณข้อมูลที่ซับซ้อน, กรอง Array, จัดเรียงข้อมูลเพื่อแสดงผล | การกระทำ (Actions), การคลิกปุ่ม, การคำนวณที่ต้องส่ง Parameter เข้าไป |

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code)

มาดูตัวอย่างที่เห็นภาพชัดเจนที่สุดครับ เราจะจำลองตารางสินค้าที่มีฟังก์ชันค้นหา และมีปุ่ม "กดนับจำนวน" ที่ไม่เกี่ยวข้องกันเลย เพื่อดูว่าใครจะกินทรัพยากรเครื่องมากกว่ากัน!

```vue
<template>
  <div class="dashboard">
    <h2>ตัวอย่างความต่าง Computed vs Methods</h2>
    
    <!-- ปุ่มนี้แค่เพิ่มตัวเลข ไม่เกี่ยวกับรายชื่อด้านล่างเลย -->
    <button @click="counter++">คลิกไปแล้ว: {{ counter }} ครั้ง</button>
    <hr>

    <input v-model="searchQuery" placeholder="ค้นหาสินค้า..." />

    <div class="columns">
      <div class="col">
        <h3>ใช้ Methods (❌ ไม่แนะนำ)</h3>
        <!-- สังเกตว่ามีวงเล็บ () -->
        <ul>
          <li v-for="item in getFilteredItemsMethod()" :key="item.id">
            {{ item.name }}
          </li>
        </ul>
      </div>

      <div class="col">
        <h3>ใช้ Computed (✅ แนะนำสุดๆ)</h3>
        <!-- สังเกตว่าไม่มีวงเล็บ ปฏิบัติเหมือนเป็นตัวแปรหนึ่ง -->
        <ul>
          <li v-for="item in filteredItemsComputed" :key="item.id">
            {{ item.name }}
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const counter = ref(0)
const searchQuery = ref('')
const items = ref([
  { id: 1, name: 'MacBook Pro' },
  { id: 2, name: 'iPhone 15' },
  { id: 3, name: 'iPad Air' }
])

// ❌ แบบที่ 1: ใช้ Method
const getFilteredItemsMethod = () => {
  console.log('🔴 Method รันใหม่แล้ว! (เสียทรัพยากร)')
  // สมมติว่ามีรายการเป็นหมื่นๆ รายการ ตรงนี้จะช้ามาก
  return items.value.filter(item => 
    item.name.toLowerCase().includes(searchQuery.value.toLowerCase())
  )
}

// ✅ แบบที่ 2: ใช้ Computed
const filteredItemsComputed = computed(() => {
  console.log('🟢 Computed คำนวณใหม่เฉพาะตอน Search เปลี่ยน!')
  return items.value.filter(item => 
    item.name.toLowerCase().includes(searchQuery.value.toLowerCase())
  )
})
</script>
```

**🔍 สิ่งที่จะเกิดขึ้นเมื่อคุณรันโค้ดนี้:**
1. ตอนที่พิมพ์ช่องค้นหา (`searchQuery` เปลี่ยน): ทั้งคู่จะทำงานและล็อกข้อความออกมา อันนี้ปกติครับ
2. **🚨 แต่ตอนที่คุณกดปุ่ม "คลิกไปแล้ว" (`counter` เปลี่ยน):**
   * ทันทีที่ `counter` เปลี่ยน หน้าจอจะเกิดการ Re-render เพื่ออัปเดตตัวเลข
   * สิ่งที่เกิดขึ้นคือ `console.log('🔴 Method รันใหม่แล้ว!')` **จะถูกพ่นออกมาทุกครั้งที่คุณกดปุ่ม!** ทั้งๆ ที่ลิสต์สินค้าไม่ได้เปลี่ยนเลย
   * แต่ `filteredItemsComputed` จะนิ่งเงียบ ไม่ประมวลผลซ้ำ เพราะมันรู้ว่า `searchQuery` กับ `items` ยังเหมือนเดิม นี่แหละครับพลังของ **Caching**!

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips)

**💡 กฎเหล็กของ Computed (Pure Functions เท่านั้น)**
Senior Dev ต้องรู้ไว้เลยว่า ภายในฟังก์ชันของ `computed` จะต้องเป็น **"Pure Function"** เท่านั้นครับ! 
*   **ห้ามทำ Side Effects:** ห้ามทำการเปลี่ยนแปลงค่า State อื่นๆ (Mutate state), ห้ามยิง API (Asynchronous fetch), หรือห้ามดัดแปลง DOM โดยตรงใน Computed เด็ดขาด! 
*   Computed มีหน้าที่แค่ "รับค่า (Dependencies) -> คำนวณ -> และส่งคืนค่า (Return)" เท่านั้น 
*   และอย่าลืมว่า ผลลัพธ์ที่ได้จาก Computed ถือเป็นค่าแบบ **Read-only** หากเป็น Array ก็อย่าไปใช้คำสั่งอย่าง `.reverse()` หรือ `.sort()` ตรงๆ เพราะมันจะไปเปลี่ยนค่า Array ต้นทาง (Mutate original array) แนะนำให้สร้าง Copy ก่อนเสมอ เช่น `[...items.value].reverse()` ครับ

## 6. 🏁 บทสรุป (To be continued...)
สรุปสั้นๆ ให้จำขึ้นใจ: **"ถ้าแค่คำนวณและคืนค่า ให้เชื่อใจ Computed แต่ถ้าต้องรับ Parameter หรือทำแอคชั่นเมื่อถูกคลิก ให้ใช้ Methods"**

การเข้าใจเรื่อง Dependency Tracking และ Caching ของ Computed Properties คือกุญแจสำคัญที่แบ่งแยกแยกระหว่าง "โค้ดที่ทำงานได้" กับ "โค้ดที่มีประสิทธิภาพสูง" ครับ ถ้าน้องๆ นำเทคนิคนี้ไปปรับใช้ รับรองว่าเว็บจะลื่นไหลขึ้น ลดภาระเครื่องผู้ใช้ไปได้มหาศาลเลยทีเดียว

ในบทต่อไป เราจะเจาะลึกลงไปอีกขั้น กับเพื่อนซี้ของ Computed อย่าง **Watchers (`watch`)** มาดูกันว่าถ้าเราต้องการ "ทำอะไรบางอย่าง" แบบมี Side Effects เมื่อข้อมูลเปลี่ยน เราจะต้องใช้เวทมนตร์บทไหน รอติดตามได้เลยครับ!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Web Application หรือ Frontend Architecture ให้กับธุรกิจของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและพัฒนาซอฟต์แวร์แบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p