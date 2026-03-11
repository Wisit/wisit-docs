---
title: "Custom Events ($emit): เมื่อลูกอยากบอกความลับให้พ่อรู้"
weight: 170
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Vue3", "Events", "Emit", "Component", "Frontend"]
---
{{< figure src="vue-custom-events-emit-cover.jpg" alt="Aesthetic study notes about Custom Events and $emit in Vue.js on an iPad screen" >}}

## 1. 🎯 ตอนที่ 17: Custom Events ($emit) วิทยุสื่อสารฉบับ Component ลูกขี้ฟ้อง

## 2. 📖 เปิดฉาก (The Hook)
ในบทที่แล้ว เราได้เรียนรู้วิชาการถ่ายทอดพลังจาก "แม่สู่ลูก" ผ่านการส่ง **Props** กันไปแล้วใช่ไหมครับ? แต่ในโลกความเป็นจริง การสื่อสารมันต้องเป็นแบบสองทาง (Two-way communication) 

ลองจินตนาการว่า คุณแม่ (Parent Component) สั่งให้คุณลูก (Child Component) ไปล้างจาน โดยส่งตะกร้าจาน (Props) ไปให้ แต่ปัญหาคือ... พอคุณลูกล้างจานเสร็จ หรือเผลอทำจานแตก คุณลูกจะบอกแม่ยังไงดีล่ะ? จะแอบไปแก้ข้อมูลในตะกร้าของแม่ตรงๆ ก็ไม่ได้ เพราะเรามีกฎเหล็ก "One-Way Data Flow" ค้ำคออยู่ (ห้ามแก้ Props โดยตรง)

ในโลกของ Vue.js เรามีอุปกรณ์สื่อสารที่เรียกว่า **Custom Events** หรือ **$emit** ครับ มันเปรียบเสมือน "วิทยุสื่อสาร" ที่คุณลูกเอาไว้วิทยุไปบอกแม่ว่า *"แม่จ๋า! หนูทำจานแตกไป 1 ใบ รหัสจานคือเบอร์ 5 แม่ช่วยลบออกจากรายการทีนะ!"* วันนี้พี่จะพามาดูวิธีการใช้ `$emit` เพื่อส่งความลับ (และข้อมูล) จากลูกกลับไปหาแม่กันครับ รับรองว่าสนุกแน่นอน!

## 3. 🧠 แก่นวิชา (Core Concepts)

### 📡 1. Props go down, Events go up
ท่องคาถานี้ไว้ให้ขึ้นใจเลยครับ: **"Props ไหลลงล่าง (ส่งข้อมูล), Events ลอยขึ้นบน (ส่งสัญญาณ)"** 
เมื่อ Child Component ต้องการให้ Parent ทำอะไรบางอย่าง (เช่น ลบข้อมูล, อัปเดตสถานะ, เปิดหน้าต่าง Modal) Child จะไม่ทำเอง แต่จะทำตัวเป็น "คนชี้เป้า" โดยการใช้คำสั่ง `emit` ปล่อยสัญญาณขึ้นไปบนฟ้า ส่วน Parent ที่ดักฟังอยู่ (Listening) ก็จะรับหน้าที่จัดการข้อมูลหลักให้เอง

### 📣 2. การประกาศใช้ `defineEmits`
ในยุคของ Vue 3 `<script setup>` เราจะใช้มาโครที่ชื่อว่า `defineEmits()` เพื่อประกาศว่า Component ลูกตัวนี้ "มีความสามารถในการส่งสัญญาณอะไรออกไปบ้าง" คล้ายๆ กับการลงทะเบียนวิทยุสื่อสารครับ โดยเราสามารถส่งผ่าน **Payload (ข้อมูลแนบ)** ไปกับ Event ได้ด้วย

### 🎧 3. การดักฟังด้วย `v-on` (หรือ `@`)
ฝั่ง Parent Component จะดักฟัง Event ที่ลูกส่งมาด้วยวิธีเดียวกับการดักฟัง DOM Event ปกติ (เช่น `@click`) เพียงแต่เปลี่ยนชื่อเป็น "ชื่อ Event ที่ลูกตั้งขึ้นมา (Custom Event)" แทน

{{< figure src="vue-custom-events-emit-diagram.jpg" alt="Diagram showing Vue component communication, Props going down and Custom Events emitting up" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code)

เพื่อให้เห็นภาพชัดเจนที่สุด เราจะมาสร้าง "รายชื่อพนักงาน" ที่มี "ปุ่มลบ (Delete)" อยู่ใน Component ลูกกันครับ

**ชิ้นส่วนลูก: `src/components/EmployeeItem.vue`**
หน้าที่ของลูกคือแสดงผล และ "วิทยุบอกแม่" เมื่อปุ่มถูกคลิก
```vue
<template>
  <li class="employee-card">
    <div class="info">
      <h4>{{ employee.name }}</h4>
      <p>ตำแหน่ง: {{ employee.position }}</p>
    </div>
    
    <!-- เมื่อปุ่มถูกคลิก เราจะเรียกฟังก์ชัน handleDelete -->
    <button class="btn-delete" @click="handleDelete">
      ไล่ออก! ❌
    </button>
  </li>
</template>

<script setup>
// 1. รับ Props จากแม่
const props = defineProps({
  employee: Object
})

// 2. ลงทะเบียนวิทยุสื่อสาร: ประกาศว่า Component นี้สามารถ emit Event ชื่อ 'delete-item' ได้
const emit = defineEmits(['delete-item'])

const handleDelete = () => {
  // 3. ปล่อยสัญญาณ! ร้องตะโกนบอกแม่ว่า "เกิดเหตุการณ์ delete-item นะ!"
  // และแนบ Payload (รหัสพนักงาน) ขึ้นไปให้แม่ด้วย
  emit('delete-item', props.employee.id)
}
</script>

<style scoped>
.employee-card { display: flex; justify-content: space-between; border: 1px solid #ccc; padding: 10px; margin-bottom: 10px; }
.btn-delete { background: red; color: white; border: none; padding: 5px 10px; cursor: pointer; }
</style>
```

**ชิ้นส่วนแม่: `src/App.vue`**
หน้าที่ของแม่คือถือครองข้อมูลหลัก (State) และรอรับฟังคำขอจากลูก
```vue
<template>
  <div class="dashboard">
    <h2>รายชื่อพนักงานบริษัท 🏢</h2>
    <ul>
      <!-- 4. ดักฟัง Event ที่ชื่อว่า @delete-item จากลูกแต่ละคน -->
      <EmployeeItem 
        v-for="emp in employees" 
        :key="emp.id"
        :employee="emp"
        @delete-item="removeEmployee" 
      />
    </ul>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import EmployeeItem from './components/EmployeeItem.vue'

// ข้อมูลหลัก (Single Source of Truth) อยู่ที่แม่
const employees = ref([
  { id: 1, name: 'สมชาย', position: 'Developer' },
  { id: 2, name: 'สมหญิง', position: 'Designer' },
  { id: 3, name: 'สมศรี', position: 'QA' }
])

// 5. เมื่อลูกวิทยุมาบอก ฟังก์ชันนี้จะทำงาน 
// สังเกตว่า 'employeeId' ที่รับเข้ามา ก็คือ Payload (props.employee.id) ที่ลูกส่งมานั่นเอง!
const removeEmployee = (employeeId) => {
  console.log(`ได้รับแจ้งจากลูก: ให้ลบพนักงานรหัส ${employeeId}`)
  // ทำการลบข้อมูลออกจาก Array หลัก
  employees.value = employees.value.filter(emp => emp.id !== employeeId)
}
</script>
```

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips)

มีหลุมพรางและเทคนิคระดับ Senior ที่น้องๆ ควรรู้เมื่อใช้งาน Events ครับ:

**🚨 เคล็ดลับที่ 1: กฎการตั้งชื่อ Event (Kebab-case เสมอ)**
สังเกตไหมครับว่าทำไมพี่ถึงตั้งชื่อ Event ว่า `delete-item` (ใช้ขีดกลาง) ทำไมไม่ตั้งเป็น `deleteItem` แบบ CamelCase?
เหตุผลก็คือ เวลาเรานำ Component ไปใช้ใน `<template>` ซึ่งอิงตามมาตรฐานของ HTML... **HTML มันแยกตัวพิมพ์เล็ก-พิมพ์ใหญ่ไม่ออกครับ! (Case-insensitive)** 
ถ้าลูก emit มาว่า `deleteItem` แต่ตอนแม่ไปเขียนในแท็ก HTML ว่า `@deleteItem="..."` Vue อาจจะหา Event ไม่เจอ การใช้ `kebab-case` (ตัวเล็กหมดคั่นด้วยขีด) จึงเป็น Best Practice ที่ Official Guide แนะนำเสมอครับ!

**✨ เคล็ดลับที่ 2: Custom Events "ไม่บับเบิล (Do not bubble)"**
ต่างจาก Native DOM Event อย่าง `click` หรือ `input` ที่สามารถทะลุ (Bubble) ขึ้นไปหา Element ชั้นปู่ย่าตายายได้... **Component Custom Events จะส่งได้แค่ 1 ชั้น (จากลูกไปหาพ่อแม่โดยตรงเท่านั้น)**
หากน้องๆ มี Component ซ้อนกัน 5 ชั้น แล้วหลานคนสุดท้องอยากส่ง Event ไปหาปู่ น้องๆ ต้องทำ Event Relay (รับแล้วส่งต่อขึ้นไปเรื่อยๆ) ซึ่งมันจะน่าเบื่อมาก (เราเรียกปัญหานี้ว่า Event Drilling) หากเจอเหตุการณ์แบบนั้น ให้พิจารณาไปใช้ **State Management อย่าง Pinia** หรือ **Provide/Inject** จะสวยงามกว่าครับ

**📦 เคล็ดลับที่ 3: ส่ง Payload ได้มากกว่า 1 อย่าง**
หากน้องๆ ต้องการส่งข้อมูลหลายๆ ตัวกลับไปให้แม่ ไม่ต้องเรียก `emit` ซ้ำหลายรอบครับ น้องสามารถจับมันยัดใส่ Object แล้วส่งไปก้อนเดียวได้เลย:
```javascript
// ฝั่งลูก
emit('update-user', { id: 1, name: 'John', status: 'Active' })

// ฝั่งแม่
const handleUpdate = (userData) => {
  console.log(userData.name) // ได้ 'John'
}
```

## 6. 🏁 บทสรุป (To be continued...)
**Custom Events ($emit)** คือจิ๊กซอว์ชิ้นสำคัญที่ทำให้ Component Architecture ของเราสมบูรณ์แบบครับ เมื่อประกอบร่างเข้ากับ **Props** เราจะได้วงจร "Props ลงล่าง - Events ขึ้นบน" ที่ทำให้ข้อมูลถูกจัดการอย่างเป็นระเบียบ อยู่ที่จุดเดียว (Single Source of Truth) ช่วยลดปัญหาบั๊กข้อมูลขัดแย้งกันได้อย่างชะงัดนัก

ตอนนี้เราสามารถส่งข้อมูลไปๆ มาๆ ระหว่าง Component ได้อย่างอิสระแล้ว แต่ความสนุกยังไม่จบแค่นั้นครับ! ถ้าเราต้องการส่ง "โครงสร้าง HTML (Template)" เข้าไปแทรกตรงกลาง Component ลูกล่ะ เราจะทำอย่างไร? 

ในตอนหน้า เราจะมาพบกับสิ่งที่เรียกว่า **Slots (สล็อต)** ฟีเจอร์สุดเทพที่จะทำให้ Component ของคุณยืดหยุ่นราวกับยางยืด เตรียมตัวไว้ให้ดีครับ!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Web Application หรือ Frontend Architecture ให้กับธุรกิจของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและพัฒนาซอฟต์แวร์แบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p