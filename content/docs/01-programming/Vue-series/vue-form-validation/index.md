---
title: "Form Validation ใน Vue: ตรวจสอบข้อมูลให้ถูกต้องก่อนส่งถึงมือ Server"
weight: 210
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Vue3", "Form", "Validation", "Vuelidate", "Frontend"]
---
{{< figure src="vue-form-validation-cover.jpg" alt="Aesthetic study notes about Form Validation in Vue.js on an iPad screen" >}}

## 1. 🎯 ตอนที่ 21: Form Validation สร้างด่านตรวจคนเข้าเมืองให้ข้อมูลของคุณ

## 2. 📖 เปิดฉาก (The Hook)
น้องๆ เคยเจอประสบการณ์แบบนี้ไหมครับ? นั่งกรอกฟอร์มลงทะเบียนยาว 20 ช่องจนเสร็จ พอกดปุ่ม Submit ปุ๊บ... หน้าเว็บหมุนติ้วๆ โหลดไป 5 วินาที แล้วก็เด้งข้อความสีแดงกลับมาบอกว่า *"กรุณากรอกอีเมลให้ถูกต้อง"* (อ้าว! แล้วทำไมไม่บอกตั้งแต่ตอนพิมพ์เสร็จล่ะ!)

นี่คือตัวอย่างของ User Experience (UX) ที่แย่มากครับ และในฝั่งของ Backend Developer การต้องมารับมือกับข้อมูลขยะ (เช่น รหัสผ่านสั้นเกินไป, อีเมลไม่มีตัว @) ก็เป็นการเปลืองทรัพยากร Server (Bandwidth) โดยใช่เหตุ 

ในฐานะ Frontend Developer เรามีหน้าที่เป็น **"การ์ดหน้าผับ" (Bouncer)** ครับ! เราต้องทำ **Form Validation** (การตรวจสอบความถูกต้องของฟอร์ม) เพื่อคัดกรองข้อมูลให้เป๊ะที่สุดก่อนจะปล่อยผ่านไปให้ Server วันนี้พี่จะพามาดูเทคนิคการตรวจฟอร์มใน Vue.js ทั้งแบบ "ลูกทุ่งเขียนเอง" และแบบ "ใช้เครื่องมือทุ่นแรง" อย่าง Plugin ยอดฮิตกันครับ!

## 3. 🧠 แก่นวิชา (Core Concepts)

การทำ Form Validation ใน Vue.js มักจะแบ่งออกเป็น 2 สายหลักๆ ครับ ขึ้นอยู่กับขนาดของโปรเจกต์:

### 🛠️ 1. แบบ Manual (เขียนมือด้วยพลังแห่ง Vue)
เหมาะสำหรับฟอร์มเล็กๆ ที่มีแค่ 2-3 ช่อง (เช่น ฟอร์ม Login) เราสามารถใช้ความสามารถพื้นฐานของ Vue มาประกอบร่างกันได้เลย:
*   **State (`ref` / `reactive`):** เก็บค่าที่ผู้ใช้พิมพ์ และเก็บข้อความ Error
*   **Event Handling (`@blur` / `@input`):** คอยดักจับว่าผู้ใช้พิมพ์เสร็จหรือยัง หรือคลิกเมาส์ออกจากช่องกรอกหรือยัง
*   **Computed Properties:** ใช้เช็กสถานะรวมของฟอร์ม (เช่น ถ้าไม่มี Error เลย ให้ปุ่ม Submit กดได้)

### 🧰 2. แบบ Plugin (ให้ผู้เชี่ยวชาญจัดการ)
เมื่อฟอร์มของคุณเริ่มมีขนาดใหญ่ (เช่น หน้าลงทะเบียนที่มี 10 ช่อง และมีเงื่อนไขซับซ้อน) การเขียน `if-else` เช็กทีละช่องจะทำให้โค้ดรกเป็นป่าดงดิบ ในโลกของ Vue จึงมี Plugin ยอดฮิตชื่อว่า **Vuelidate** (หรืออีกค่ายคือ VeeValidate)
*   **Vuelidate:** เป็น Plugin แนว Model-based validation ที่เบาหวิว มันจะให้เราเขียน "กฎ (Rules)" เช่น `required` (ห้ามว่าง), `minLength` (ความยาวขั้นต่ำ), `email` (รูปแบบอีเมล) แปะไว้คู่กับตัวแปร State ได้เลย แล้วมันจะจัดการเช็กสถานะให้เราแบบอัตโนมัติ!

{{< figure src="vue-form-validation-diagram.jpg" alt="Diagram showing the Form Validation flow in Vue comparing Manual Validation versus Plugin Validation" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code)

เรามาดูการเปรียบเทียบโค้ดทั้ง 2 แบบ ในการทำ "หน้าเข้าสู่ระบบ (Login Form)" ที่ต้องเช็กอีเมลและรหัสผ่านกันครับ

### ❌ ท่าที่ 1: Manual Validation (เขียนเองนักเลงพอ)
เราใช้ Regular Expression (Regex) ในการตรวจอีเมล และตรวจความยาวรหัสผ่าน

```vue
<template>
  <form @submit.prevent="submitForm" class="login-form">
    <!-- ช่องอีเมล -->
    <div class="form-group">
      <label>อีเมล:</label>
      <input 
        v-model.trim="email" 
        @blur="validateEmail" 
        @input="clearEmailError" 
        :class="{ 'is-error': emailError }" 
      />
      <!-- แสดง Error เมื่อตัวแปร emailError มีค่า -->
      <span v-if="emailError" class="error-text">{{ emailError }}</span>
    </div>

    <!-- ช่องรหัสผ่าน -->
    <div class="form-group">
      <label>รหัสผ่าน:</label>
      <input 
        type="password" 
        v-model="password" 
        @blur="validatePassword" 
        @input="clearPasswordError"
      />
      <span v-if="passwordError" class="error-text">{{ passwordError }}</span>
    </div>

    <!-- ปุ่ม Submit จะกดได้ก็ต่อเมื่อข้อมูลถูกหมด -->
    <button type="submit" :disabled="!isFormValid">เข้าสู่ระบบ</button>
  </form>
</template>

<script setup>
import { ref, computed } from 'vue'

const email = ref('')
const password = ref('')
const emailError = ref('')
const passwordError = ref('')

// ฟังก์ชันตรวจอีเมล
const validateEmail = () => {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
  if (!email.value) {
    emailError.value = 'กรุณากรอกอีเมล'
  } else if (!emailRegex.test(email.value)) {
    emailError.value = 'รูปแบบอีเมลไม่ถูกต้อง'
  }
}

// ฟังก์ชันตรวจรหัสผ่าน
const validatePassword = () => {
  if (!password.value) {
    passwordError.value = 'กรุณากรอกรหัสผ่าน'
  } else if (password.value.length < 8) {
    passwordError.value = 'รหัสผ่านต้องมีอย่างน้อย 8 ตัวอักษร'
  }
}

// เคลียร์ Error ทันทีที่ผู้ใช้เริ่มพิมพ์แก้ตัวใหม่
const clearEmailError = () => emailError.value = ''
const clearPasswordError = () => passwordError.value = ''

// ตรวจสอบว่าฟอร์มสมบูรณ์พร้อมส่งหรือยัง?
const isFormValid = computed(() => {
  return email.value && password.value && !emailError.value && !passwordError.value
})

const submitForm = () => {
  if (isFormValid.value) {
    console.log('ส่งข้อมูลไป Server:', { email: email.value, password: password.value })
  }
}
</script>
```
*เห็นไหมครับว่าแค่ 2 ช่อง โค้ดเราก็เริ่มยาวแล้ว เพราะต้องสร้าง State มารองรับ Error ของแต่ละฟิลด์แยกกัน*

---

### ✅ ท่าที่ 2: ใช้ Plugin (Vuelidate) ยกระดับความโปร
สมมติว่าเราติดตั้ง Vuelidate เรียบร้อยแล้ว โค้ดของเราจะสะอาดตาขึ้นมาก เพราะเราโยน "กฎ" ไปให้ Plugin จัดการแทน

```vue
<template>
  <form @submit.prevent="submitForm">
    <div class="form-group">
      <label>อีเมล:</label>
      <!-- อ้างอิงสถานะ Error ผ่าน Object $v -->
      <input 
        v-model="loginData.email" 
        @blur="$v.loginData.email.$touch()" 
      />
      <!-- $v.loginData.email.$error จะเป็น true เมื่อข้อมูลผิดกฎ -->
      <span v-if="$v.loginData.email.$error" class="error-text">
        กรุณากรอกอีเมลให้ถูกต้อง
      </span>
    </div>

    <div class="form-group">
      <label>รหัสผ่าน:</label>
      <input 
        type="password" 
        v-model="loginData.password" 
        @blur="$v.loginData.password.$touch()" 
      />
      <span v-if="$v.loginData.password.$error" class="error-text">
        รหัสผ่านต้องมีอย่างน้อย 8 ตัวอักษร
      </span>
    </div>

    <!-- $invalid เป็นสถานะรวมของทั้งฟอร์มจาก Vuelidate -->
    <button type="submit" :disabled="$v.$invalid">เข้าสู่ระบบ</button>
  </form>
</template>

<script setup>
import { reactive } from 'vue'
import { useVuelidate } from '@vuelidate/core'
// Import กฎมาตรฐานที่ Vuelidate เตรียมไว้ให้แล้ว (ไม่ต้องเขียน Regex เอง!)
import { required, email, minLength } from '@vuelidate/validators' 

const loginData = reactive({
  email: '',
  password: ''
})

// ประกาศกฎ (Rules) ให้ตรงกับชื่อตัวแปร
const rules = {
  loginData: {
    email: { required, email }, // ต้องมีข้อมูล และต้องเป็นรูปแบบอีเมล
    password: { required, minLength: minLength(8) } // ต้องมีข้อมูล และขั้นต่ำ 8 ตัว
  }
}

// ผูกกฎเข้ากับข้อมูล เกิดเป็นอ็อบเจกต์เวทมนตร์ชื่อ $v
const $v = useVuelidate(rules, loginData)

const submitForm = () => {
  $v.value.$touch() // บังคับให้ตรวจทุกช่องก่อนส่ง
  if (!$v.value.$invalid) {
    console.log('ข้อมูลถูกต้อง! ส่งไป Server ได้')
  }
}
</script>
```

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips)

**✨ 1. จังหวะการด่า... เอ้ย! จังหวะการเตือนผู้ใช้ (UX Matters)**
ถ้าสังเกตดีๆ ในโค้ดพี่จะใช้ Event `@blur` ในการสั่งให้มันเริ่มด่า (ตรวจสอบ) เสมอครับ `@blur` คือจังหวะที่ "ผู้ใช้คลิกเมาส์ออกจากช่องนั้นแล้ว" 
*จงหลีกเลี่ยงการแจ้ง Error ตั้งแต่ตอนที่เค้ากำลังพิมพ์ตัวแรก (ผ่าน `@input` ตรงๆ)* เพราะมันทำให้ผู้ใช้รู้สึกหงุดหงิดเหมือนโดนจับผิดตลอดเวลาครับ! การใช้ `$touch()` ใน Vuelidate ตอน `@blur` คือ Best Practice ที่บอกระบบว่า *"ช่องนี้ผู้ใช้ไปแตะมันและพิมพ์เสร็จแล้วนะ ตรวจเลย!"*

**🚨 2. กฎเหล็กสาย Security: "Never Trust the Client"**
คำเตือนจาก Senior Dev ครับ! แม้ว่าคุณจะทำ Form Validation ใน Vue (Client-side) ได้เทพและรัดกุมแค่ไหนก็ตาม **คุณก็ยังจำเป็นต้องให้ฝั่ง Backend (Server-side) เขียนโค้ดตรวจสอบข้อมูลซ้ำอีกรอบเสมอครับ!** 
เหตุผลก็คือ Client-side ถูกสร้างมาเพื่อ "UX ที่ดี (ให้รู้ทันทีไม่ต้องรอโหลด)" แต่มันถูกแฮ็กหรือปิดการทำงานด้วยเครื่องมือของ Hacker ได้ง่ายมาก การรักษาความปลอดภัยที่แท้จริงต้องอยู่ที่ Server เสมอครับ!

## 6. 🏁 บทสรุป (To be continued...)
**Form Validation** ไม่ใช่แค่การดักจับข้อผิดพลาด แต่มันคือศิลปะในการนำทางผู้ใช้งาน (Guiding Users) ให้มอบข้อมูลที่ถูกต้องให้กับระบบของเราอย่างละมุนละม่อม การรู้จักประยุกต์ใช้เครื่องมืออย่าง **Vuelidate** จะช่วยชีวิตน้องๆ จากการจมกองโค้ด `if-else` มหาศาลได้อย่างแน่นอนครับ

เมื่อเรากรอกข้อมูลถูกต้องแล้ว ขั้นตอนต่อไปก็ต้อง "ส่ง" ข้อมูลก้อนนี้ข้ามอินเทอร์เน็ตไปหาฐานข้อมูลใช่ไหมครับ? ในบทต่อไป เราจะมาสวมบทบาทเป็นนักสื่อสาร เรียนรู้วิธีการยิง API คุยกับ Server ด้วยเครื่องมือยอดฮิตอย่าง **Axios** เตรียมตัวออกสู่โลกกว้างกันได้เลยครับ!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Web Application หรือ Frontend Architecture ให้กับธุรกิจของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและพัฒนาซอฟต์แวร์แบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p