---
title: "Step-by-Step Workshop: ระบบบันทึกข้อมูลพนักงาน"
weight: 200
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Vue3", "Workshop", "CRUD", "Component", "v-model", "v-for"]
---
{{< figure src="vue-workshop-employee-system-cover.jpg" alt="Aesthetic study notes about Employee Data Registration System with Vue 3 on an iPad screen" >}}

## 1. 🎯 ตอนที่ 20: Step-by-Step Workshop สร้างระบบจัดการพนักงานฉบับจับมือทำ

## 2. 📖 เปิดฉาก (The Hook)
น้องๆ ครับ ตั้งแต่บทแรกจนถึงบทที่แล้ว เราได้ฝึกฝนวิทยายุทธแยกส่วนกันมาพอสมควร ไม่ว่าจะเป็นการทำ Data Binding, การจัดการ Form (`v-model`), การวนลูป (`v-for`), ไปจนถึงการให้ Component พ่อลูกคุยกันผ่าน `Props` และ `$emit` 

แต่วิชาเหล่านี้จะไม่มีประโยชน์เลย ถ้าเราจับมันมา "ประกอบร่าง" เป็นแอปพลิเคชันจริงๆ ไม่เป็น! 

วันนี้พี่จะพาน้องๆ สวมหมวก Senior Developer มาลุย **Workshop: ระบบบันทึกข้อมูลพนักงาน (Employee Management System)** เราจะสร้างแอปพลิเคชันที่สามารถกรอกข้อมูลพนักงานใหม่ (ชื่อ, เงินเดือน, ตำแหน่ง, เพศ, ทักษะ) กดบันทึกแล้วนำไปแสดงผลเป็นรายชื่อ และสามารถกดลบพนักงานที่ลาออกไปแล้วได้ด้วย จิบกาแฟให้พร้อม แล้วมากางพิมพ์เขียวระบบนี้ไปพร้อมๆ กันเลยครับ!

## 3. 🧠 แก่นวิชา (Core Concepts)

ก่อนจะลงมือเขียนโค้ด เราต้องมาวาง **Component Architecture** กันก่อนครับ การโยนโค้ดทุกอย่างไว้ในไฟล์เดียวคือฝันร้ายของการบำรุงรักษา เราจะแบ่งแอปพลิเคชันของเราออกเป็น 4 ชิ้นส่วน (Components) ดังนี้ครับ:

1.  🏢 **`App.vue` (ผู้จัดการใหญ่ / Root Component):** ทำหน้าที่เป็นศูนย์กลางเก็บ State หลัก นั่นคือ Array ที่ชื่อว่า `employees` คอยรับข้อมูลจากฟอร์มมาเก็บไว้ และส่งข้อมูลนี้ลงไปให้ส่วนแสดงผล
2.  📝 **`FormComponent.vue` (ฝ่ายบุคคล / Form):** มีหน้าที่รับ input จากผู้ใช้ผ่านช่องกรอกข้อมูลต่างๆ เมื่อกด "บันทึก" จะใช้ Custom Event (`$emit`) ส่งข้อมูลพนักงานคนใหม่ขึ้นไปให้ `App.vue`
3.  📋 **`ListData.vue` (บอร์ดประกาศ / List):** รับข้อมูล `employees` มาจาก `App.vue` ผ่าน `Props` จากนั้นใช้ `v-for` เพื่อวนลูปสร้างการ์ดพนักงานตามจำนวนข้อมูลที่มี
4.  🪪 **`Person.vue` (การ์ดพนักงาน / Item):** รับข้อมูลพนักงาน 1 คน (เช่น ชื่อ, เงินเดือน) มาจัดหน้าตาให้สวยงาม และมีปุ่ม "ลบข้อมูล" ที่เมื่อคลิก จะส่งสัญญาณ (`$emit`) พร้อม ID กลับขึ้นไปให้ `App.vue` ลบข้อมูลทิ้ง

{{< figure src="vue-workshop-employee-system-diagram.jpg" alt="Diagram showing Vue.js component architecture for an Employee Data System" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code)

เรามาดูการประกอบร่างโค้ดทีละชิ้นกันครับ (พี่จะเขียนด้วย `<script setup>` Composition API เพื่อความโมเดิร์นนะครับ)

### 📝 1. FormComponent.vue (รับข้อมูล)
ในส่วนนี้เราจะใช้ `v-model` เพื่อผูกข้อมูลแบบ Two-way Data Binding กับ Form Inputs ทุกรูปแบบ
```vue
<template>
  <form @submit.prevent="submitForm" class="card">
    <h3>เพิ่มพนักงานใหม่</h3>
    
    <!-- Text Input -->
    <div>
      <label>ชื่อพนักงาน:</label>
      <input type="text" v-model.trim="employeeForm.name" required />
    </div>

    <!-- Number Input -->
    <div>
      <label>เงินเดือน:</label>
      <input type="number" v-model="employeeForm.salary" required />
    </div>

    <!-- Select Dropdown -->
    <div>
      <label>ตำแหน่ง:</label>
      <select v-model="employeeForm.department">
        <option value="ฝ่ายขาย">ฝ่ายขาย</option>
        <option value="ฝ่ายการตลาด">ฝ่ายการตลาด</option>
        <option value="ฝ่ายไอที">ฝ่ายไอที</option>
      </select>
    </div>

    <button type="submit">บันทึกข้อมูล</button>
  </form>
</template>

<script setup>
import { ref } from 'vue'

// ประกาศ Event ที่ Component นี้จะส่งออกไปหาแม่
const emit = defineEmits(['add-employee'])

// State สำหรับเก็บข้อมูลฟอร์ม
const employeeForm = ref({
  name: '',
  salary: 0,
  department: 'ฝ่ายขาย'
})

const submitForm = () => {
  // สร้าง Object ใหม่ พร้อมแนบ ID เพื่อป้องกัน Reference Type Bug
  const newEmployee = { 
    id: Date.now(), 
    ...employeeForm.value 
  }
  
  // ยิงสัญญาณ 'add-employee' พร้อมแนบข้อมูลก้อนใหม่ขึ้นไป
  emit('add-employee', newEmployee)

  // เคลียร์ฟอร์มให้กลับเป็นค่าเริ่มต้น
  employeeForm.value = { name: '', salary: 0, department: 'ฝ่ายขาย' }
}
</script>
```

### 🪪 2. Person.vue (การ์ดพนักงาน 1 คน)
รับข้อมูลด้วย `Props` และมีปุ่มลบที่ส่ง `$emit` กลับไป
```vue
<template>
  <li class="person-card">
    <h4>{{ employee.name }}</h4>
    <p>ตำแหน่ง: {{ employee.department }}</p>
    <p>เงินเดือน: {{ employee.salary }} บาท</p>
    
    <!-- กดแล้วส่ง ID ขึ้นไปให้แม่ลบ -->
    <button @click="$emit('delete-emp', employee.id)">ลบข้อมูล ❌</button>
  </li>
</template>

<script setup>
// รับ Props เป็น Object ของพนักงาน
defineProps({
  employee: {
    type: Object,
    required: true
  }
})

// ประกาศสิทธิ์ในการยิง Event ลบข้อมูล
defineEmits(['delete-emp'])
</script>
```

### 📋 3. ListData.vue (พื้นที่วนลูป)
รับ Array มาวนลูปด้วย `v-for` สร้าง `Person.vue` หลายๆ ใบ
```vue
<template>
  <div>
    <h3>รายชื่อพนักงานทั้งหมด ({{ employees.length }} คน)</h3>
    <ul>
      <!-- วนลูปข้อมูลพนักงาน และรับ Event ลบข้อมูลเพื่อส่งต่อให้ App.vue -->
      <Person 
        v-for="emp in employees" 
        :key="emp.id" 
        :employee="emp" 
        @delete-emp="$emit('delete-emp', $event)" 
      />
    </ul>
  </div>
</template>

<script setup>
import Person from './Person.vue'

defineProps({
  employees: Array
})

defineEmits(['delete-emp'])
</script>
```

### 🏢 4. App.vue (ผู้จัดการใหญ่ ประกอบร่าง)
เก็บ State หลัก และเป็นคนจัดการ Logic การเพิ่มและลบ
```vue
<template>
  <div class="app-container">
    <h1>ระบบจัดการข้อมูลพนักงาน 🏢</h1>
    
    <!-- รอรับ Event @add-employee จากฟอร์ม -->
    <FormComponent @add-employee="insertEmployee" />
    
    <hr />

    <!-- ส่ง Props employees ลงไป และรอรับ Event ลบข้อมูล -->
    <ListData 
      v-if="employees.length > 0" 
      :employees="employees" 
      @delete-emp="removeEmployee" 
    />
    <p v-else>ยังไม่มีข้อมูลพนักงานในระบบ</p>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import FormComponent from './components/FormComponent.vue'
import ListData from './components/ListData.vue'

// State หลักของแอปพลิเคชัน (ก้อนข้อมูลพนักงาน)
const employees = ref([])

// ฟังก์ชันรับข้อมูลจาก Form แล้ว Push ใส่ Array
const insertEmployee = (newEmpData) => {
  employees.value.push(newEmpData)
}

// ฟังก์ชันลบข้อมูล โดยใช้ Filter คัดคนที่ ID ไม่ตรงกันไว้ และทิ้งคนที่ ID ตรงกัน
const removeEmployee = (id) => {
  employees.value = employees.value.filter(emp => emp.id !== id)
}
</script>
```

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips)

มีบั๊กสุดคลาสสิกที่มือใหม่มักจะเจอใน Workshop นี้ครับ ซึ่งพี่ต้องขอเตือนไว้ก่อนเลย:

**🚨 ภัยเงียบจาก Reference Type (ข้อมูลเปลี่ยนตามกันโดยไม่ได้ตั้งใจ)**
หากใน `FormComponent` จังหวะที่น้องๆ กดบันทึกข้อมูล แล้วน้องสั่ง `emit('add-employee', employeeForm.value)` โต้งๆ เลย สิ่งที่จะเกิดขึ้นคือ... เมื่อพนักงานคนแรกถูกเพิ่มเข้าไปในลิสต์ แล้วน้องกลับมาพิมพ์ชื่อพนักงานคนที่สองในฟอร์ม **รายชื่อคนแรกในลิสต์จะถูกเปลี่ยนตามตัวอักษรที่น้องกำลังพิมพ์ทันที!**

*เหตุผล:* เพราะ Object ใน JavaScript เป็น Pass by Reference ครับ! Array ใน `App.vue` กำลังจ้องมองไปที่ Memory Address เดียวกับตัวแปรในฟอร์ม

*วิธีแก้ระดับ Pro:* น้องต้องทำการ "สร้าง Object ใหม่" (Clone) ก่อนส่งข้อมูลขึ้นไปเสมอ ด้วยท่า Spread Operator `...` แบบนี้ครับ:
```javascript
const newEmployee = { ...employeeForm.value, id: Date.now() }
emit('add-employee', newEmployee) // ส่งของ Copy ไปแทนของจริง
```

**🗑️ การลบข้อมูลด้วย Array.filter()**
เวลาเราจะลบข้อมูลออกจาก Array ใน Vue เรามักจะไม่ใช้คำสั่งที่ไปเปลี่ยนแปลง Array ต้นฉบับตรงๆ (Mutate) แต่เราจะใช้วิธี **"คัดออก"** ด้วย `.filter()` ครับ โดยสร้างเงื่อนไขว่า *"ให้เก็บพนักงานทุกคนไว้ ยกเว้นคนที่มี ID ตรงกับที่เรากดลบ"* แล้วเอา Array ชุดใหม่ไปทับชุดเก่าแทน โค้ดจะคลีนและเกิดบั๊กน้อยกว่ามากครับ!

## 6. 🏁 บทสรุป (To be continued...)
จบกันไปแล้วครับสำหรับ Workshop ระบบพนักงาน! น้องๆ ได้เห็นภาพรวมทั้งหมดแล้วว่า แนวคิด Component Architecture มันทรงพลังขนาดไหน การแยก `Form`, `List`, `Item` ออกจากกัน ทำให้โค้ดของเราอ่านง่าย แก้ไขง่าย (Maintainability) และสามารถนำกลับมาใช้ซ้ำ (Reusability) ได้อย่างสมบูรณ์แบบ 

ทบทวนกระบวนท่าให้ดีนะครับ: **`v-model` ดึงข้อมูลจากฟอร์ม -> `$emit` ส่งข้อมูลขึ้นไปหาแม่ -> แม่อัปเดต `Array` -> แม่ส่ง `Props` ลงมาให้ลูก -> ลูกใช้ `v-for` สร้างหน้าจอ** นี่คือวงจรชีวิตการไหลของข้อมูล (Data Flow) ที่น้องๆ จะต้องใช้หากินไปตลอดชีวิตการเป็น Vue Developer ครับ!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Web Application หรือ Frontend Architecture ให้กับธุรกิจของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและพัฒนาซอฟต์แวร์แบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p