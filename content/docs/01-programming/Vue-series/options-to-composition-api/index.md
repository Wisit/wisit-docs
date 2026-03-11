---
title: "ย้ายจาก Options API สู่ Composition API: อาวุธลับของ Vue 3"
weight: 250
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Vue3", "Composition API", "Options API", "ref", "reactive", "Frontend Architecture"]
---
{{< figure src="options-to-composition-api-cover.jpg" alt="Aesthetic study notes about Options API vs Composition API in Vue 3 on an iPad screen" >}}

## 1. 🎯 ตอนที่ 25: ย้ายจาก Options API สู่ Composition API ปลดล็อกขีดจำกัดของโค้ดสปาเกตตี

## 2. 📖 เปิดฉาก (The Hook)
น้องๆ เคยตกอยู่ในสถานการณ์นี้ไหมครับ? เราได้รับมอบหมายให้ไปแก้บั๊กในหน้าเว็บ Dashboard ที่รุ่นพี่เขียนทิ้งไว้เมื่อ 2 ปีที่แล้ว ไฟล์ Component นั้นมีความยาวทะลุ 1,000 บรรทัด! 

ปัญหาคือ เมื่อเราต้องการแก้ฟีเจอร์ "ช่องค้นหา (Search)" เราต้องเลื่อนเมาส์ไปหาตัวแปรที่ส่วน `data()` เลื่อนลงไปดูสูตรคำนวณที่ `computed` เลื่อนลงไปอีกเพื่อดูฟังก์ชันที่ `methods` และต้องลงไปดักจับการเปลี่ยนแปลงที่ `watch`... การแก้ฟีเจอร์เดียว ทำให้เราต้องไถหน้าจอขึ้นลง (Scrolling up and down) เป็นสิบๆ รอบจนตาลาย โค้ดที่เกี่ยวข้องกันดันถูกจับแยกส่วนกันหมด!

นี่แหละครับคือ "ความเจ็บปวด" คลาสสิกของ **Options API** ใน Vue 2 เมื่อโปรเจกต์สเกลใหญ่ขึ้น แต่ไม่ต้องห่วงครับ เพราะใน Vue 3 ทีมผู้สร้างได้มอบอาวุธลับที่ชื่อว่า **Composition API** มาให้เราแล้ว วันนี้พี่จะพามาจิบกาแฟ และชำแหละให้ดูว่า ทำไมการย้ายมาใช้ Composition API ถึงเป็นจุดเปลี่ยนที่ทำให้ชีวิตโปรแกรมเมอร์ดีขึ้นแบบก้าวกระโดด!

## 3. 🧠 แก่นวิชา (Core Concepts)

### 🗄️ ปัญหาของ Options API (จัดของตามประเภท)
ใน Options API เราถูกบังคับให้เขียนโค้ดโดย "จัดกลุ่มตามประเภทของออปชัน (Option types)" ครับ ลองจินตนาการว่าน้องกำลังจัดกระเป๋าเดินทาง แล้วเอาเสื้อเชิ้ตทุกตัวใส่ถุงนึง เอากางเกงทุกตัวใส่ถุงนึง เอาถุงเท้าใส่ถุงนึง เวลาจะหยิบชุดใส่ไปเที่ยวทะเล 1 ชุด น้องต้องรื้อเปิดถุงทุกใบ! ในโค้ดก็เช่นกัน ฟีเจอร์ A ถูกจับแยกส่วนไปอยู่ทั้งใน `data`, `methods`, และ `mounted` ทำให้การอ่าน (Readability) และการแยกโค้ดไปใช้ซ้ำ (Logic Reuse) ทำได้ยากมาก

### 📦 กำเนิด Composition API (จัดของตามการใช้งาน)
**Composition API** เกิดมาเพื่อแก้ปัญหานี้โดยเฉพาะครับ! มันอนุญาตให้เรา "จัดกลุ่มโค้ดตามฟีเจอร์ (Logical concerns)" ได้เลย (เช่น เอาเสื้อ กางเกง ถุงเท้า สำหรับชุดเที่ยวทะเล จัดไว้ในเซ็ตเดียวกัน) โค้ดที่เกี่ยวข้องกันจะอยู่ติดกันเสมอ ทำให้อ่านง่าย และสามารถแยกออกไปเป็นไฟล์ต่างหาก (Composables) ได้อย่างง่ายดาย

### 🛠️ อาวุธประจำกายของ Composition API
การจะใช้ Composition API เราจะทิ้งระบบ `this` แบบเดิมไปทั้งหมด และหันมาใช้ ฟังก์ชัน (Functions) ที่อิมพอร์ตมาจาก Vue แทน:
1. **`setup()` / `<script setup>`:** เป็นจุดเริ่มต้น (Entry point) ของ Composition API โค้ดทั้งหมดจะถูกเขียนในนี้ โดยจะทำงานก่อนที่ Component จะถูกสร้างขึ้นมา
2. **`ref()`:** ฟังก์ชันสำหรับสร้าง Reactive State (ข้อมูลที่ตอบสนองกับหน้าจอ) มักใช้กับข้อมูลพื้นฐาน (Primitives) เช่น String, Number, Boolean เวลาจะใช้งานในสคริปต์ต้องเติม `.value` ต่อท้ายเสมอ
3. **`reactive()`:** ฟังก์ชันสำหรับสร้าง Reactive State เหมือนกัน แต่ใช้สำหรับข้อมูลประเภท Object หรือ Array เท่านั้น (ภายใต้เบื้องหลังมันใช้ระบบ JavaScript Proxies)

{{< figure src="options-to-composition-api-diagram.jpg" alt="Diagram showing the code organization comparison between Options API and Composition API" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code)

เรามาดูการเปรียบเทียบโค้ดฟีเจอร์ "ช่องค้นหาสินค้า" แบบหมัดต่อหมัดกันครับ:

### ❌ แบบดั้งเดิม: Options API (โค้ดกระจัดกระจาย)
```vue
<script>
export default {
  // 1. ตัวแปรอยู่ที่นี่
  data() {
    return {
      searchQuery: '',
      products: ['MacBook', 'iPhone', 'iPad']
    }
  },
  // 2. การคำนวณอยู่นี่
  computed: {
    filteredProducts() {
      return this.products.filter(p => p.includes(this.searchQuery))
    }
  },
  // 3. ฟังก์ชันดันมาอยู่นี่
  methods: {
    clearSearch() {
      this.searchQuery = '' // ต้องใช้ this เสมอ
    }
  }
}
</script>
```

### ✅ แบบยุคใหม่: Composition API ด้วย `<script setup>` (รวมเป็นก้อนเดียว)
ใน Vue 3 เราใช้คำสั่ง `<script setup>` (Syntactic sugar) ซึ่งช่วยให้โค้ดสั้นลง ไม่ต้องมานั่ง `export default` หรือ `return` ตัวแปรออกไปให้ Template อีกต่อไป!

```vue
<template>
  <div class="search-box">
    <input v-model="searchQuery" placeholder="ค้นหา..." />
    <button @click="clearSearch">ล้างคำค้น</button>
    <ul>
      <li v-for="item in filteredProducts" :key="item">{{ item }}</li>
    </ul>
  </div>
</template>

<script setup>
// อิมพอร์ตเครื่องมือที่ต้องใช้จาก vue
import { ref, computed } from 'vue'

// --- เริ่มโซนฟีเจอร์: ค้นหาสินค้า (จัดรวมไว้ที่เดียวกันทั้งหมด!) ---

// 1. สร้าง State ด้วย ref()
const searchQuery = ref('')
const products = ref(['MacBook', 'iPhone', 'iPad'])

// 2. สร้าง Computed Property (ไม่ต้องใช้ this แล้ว ใช้ .value แทน)
const filteredProducts = computed(() => {
  return products.value.filter(p => p.includes(searchQuery.value))
})

// 3. สร้าง Method ด้วยฟังก์ชันธรรมดา
const clearSearch = () => {
  searchQuery.value = '' 
}

// --- จบโซนฟีเจอร์ ---
</script>
```
*เห็นความแตกต่างไหมครับ! ด้วย Composition API โค้ดของฟีเจอร์ค้นหาจะอยู่ติดกันเป็นก้อนเดียว อ่านจากบนลงล่างจบเลย ไม่ต้องกระโดดข้ามบรรทัดไปมาแล้ว!*

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips)

เมื่อก้าวเข้าสู่โลกของ Composition API นี่คือกฎเหล็กและข้อควรระวังระดับ Senior ที่น้องๆ ต้องรู้ครับ:

**💡 1. ศึกชิงแชมป์ `ref` vs `reactive` (ใช้ตัวไหนดี?)**
*   **ใช้ `ref()` เป็นหลัก:** แม้ว่า `ref()` จะต้องพิมพ์ `.value` ตอนใช้งานใน Script (ใน Template ไม่ต้องพิมพ์ Vue จัดการแกะให้เอง) แต่มันยืดหยุ่นกว่าครับ มันใช้ได้กับข้อมูลทุกประเภท และสามารถเขียนทับตัวมันเองได้ทั้งหมด (Reassign)
*   **ใช้ `reactive()` เมื่อจัดกลุ่ม Object:** `reactive()` เกิดมาเพื่อ Object ล้วนๆ ข้อควรระวังสูงสุดคือ **ห้ามเขียนทับ (Destructure หรือ Reassign) ตัวมันเด็ดขาด!** เพราะจะทำให้ความสามารถในการเป็น Reactive (การอัปเดตหน้าจอ) หลุดหายไปทันที หากต้องการแกะข้อมูลออกจาก `reactive` ต้องใช้ตัวช่วยอย่าง `toRefs()` เท่านั้นครับ

**🎯 2. สวรรค์ของ TypeScript**
Options API ไม่ได้ถูกออกแบบมาเพื่อ Type Inference (การคาดเดาชนิดข้อมูล) ทำให้การใช้ร่วมกับ TypeScript ในยุคก่อนเป็นเรื่องที่น่าปวดหัวมาก แต่ Composition API ถูกเขียนขึ้นมาใหม่ทั้งหมดด้วย TypeScript ฟังก์ชันอย่าง `ref` และ `reactive` จึงรองรับ Type อย่างสมบูรณ์แบบ ช่วยให้เราเขียนแอปสเกลใหญ่ได้อย่างปลอดภัยไร้บั๊กชนิดข้อมูลครับ!

## 6. 🏁 บทสรุป (To be continued...)
**Composition API** ไม่ใช่แค่รูปแบบการเขียนโค้ดที่เปลี่ยนไป แต่มันคือ "กระบวนทัศน์ (Paradigm)" ใหม่ในการออกแบบสถาปัตยกรรม Frontend ครับ การละทิ้ง `this` และหันมาใช้ฟังก์ชัน จะช่วยให้น้องๆ สามารถจับมัดรวมโค้ด (Encapsulate) และนำไปใช้ซ้ำได้อย่างอิสระเสรี

ในเมื่อเราสามารถรวมโค้ดของฟีเจอร์ใดฟีเจอร์หนึ่งไว้เป็นก้อนเดียวกันได้แล้ว คำถามต่อไปคือ... "เราจะย้ายก้อนโค้ดเนี้ย ออกไปไว้อีกไฟล์นึง เพื่อให้ Component อื่นๆ สามารถเรียกใช้ความสามารถนี้ด้วยได้ไหม?" 

คำตอบคือ **ได้แน่นอนครับ!** และนั่นคือวิชาขั้นสูงที่เรียกว่า **"Composables"** ซึ่งเราจะมาเจาะลึกการสร้าง Custom Hooks สไตล์ Vue กันในบทถัดไป เตรียมตัวรับความสนุกได้เลยครับ!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Web Application หรือ Frontend Architecture ให้กับธุรกิจของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและพัฒนาซอฟต์แวร์แบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p