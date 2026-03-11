---
title: "เชื่อมต่อโลกภายนอกด้วย Axios: วิธีดึงข้อมูล API มาแสดงบนเว็บ"
weight: 230
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Vue3", "Axios", "API", "Async/Await", "Frontend"]
---
{{< figure src="vue-axios-api-integration-cover.jpg" alt="Aesthetic study notes about Vue.js Axios API integration on an iPad screen" >}}

## 1. 🎯 ตอนที่ 23: เปิดประตูสู่โลกภายนอกด้วย Axios ดึงและดันข้อมูล API ดั่งใจนึก

## 2. 📖 เปิดฉาก (The Hook)
น้องๆ ครับ ลองจินตนาการว่าเราสร้าง Component หน้าตาเว็บขึ้นมาสวยงามมาก มีแอนิเมชันวูบวาบ มีฟอร์มที่ทำ Validation อย่างดี แต่... ข้อมูลทั้งหมดดันถูกฮาร์ดโค้ด (Hardcoded) ไว้ในไฟล์เฉยๆ แอปพลิเคชันของเราก็เปรียบเสมือน "คฤหาสน์ที่สวยงามแต่ไม่มีประตูหน้าต่าง" ครับ มันถูกตัดขาดจากโลกภายนอก ไม่สามารถอัปเดตข้อมูลใหม่ๆ หรือบันทึกข้อมูลผู้ใช้ลงฐานข้อมูลได้เลย

ในโลกของ Frontend Development เราจำเป็นต้องให้เว็บของเราสื่อสารกับ Backend Server เพื่อขอข้อมูลหรือส่งข้อมูลไปเก็บกระบวนการนี้เราเรียกว่าการยิง API (Application Programming Interface) ครับ

แม้ว่า JavaScript จะมีคำสั่ง `fetch()` มาให้ใช้ตั้งแต่เกิด แต่มันก็ยังมีข้อจุกจิกกวนใจ เช่น ต้องมานั่งแปลงข้อมูลเป็น JSON เอง หรือจัดการ Error ได้ไม่ค่อยเนียนเท่าไหร่ นี่จึงเป็นเหตุผลที่พระเอกของเราอย่าง **Axios** ถือกำเนิดขึ้นมา! วันนี้พี่จะพาน้องๆ จิบกาแฟ แล้วมาเรียนรู้วิธีการใช้ Axios เพื่อเชื่อมต่อเว็บของเราเข้ากับโลกภายนอกกันครับ

## 3. 🧠 แก่นวิชา (Core Concepts)

### 📦 1. Axios คืออะไร?
**Axios** คือ HTTP Client ยอดฮิตที่ทำงานบนพื้นฐานของ Promise (คำมั่นสัญญา) หน้าที่ของมันคือการเป็น "บุรุษไปรษณีย์" ที่คอยวิ่งไปส่งจดหมาย (Request) ที่ Server และรอรับพัสดุ (Response) กลับมาให้เราที่หน้าเว็บครับ 

**วิธีติดตั้งในโปรเจกต์ Vue:**
เปิด Terminal แล้วพิมพ์คำสั่งนี้ได้เลย:
```bash
npm install axios
```

### 🛣️ 2. HTTP Methods (รูปแบบการส่งจดหมาย)
เวลาเราจะคุยกับ API เราต้องเลือกวิธีคุยให้ถูกต้องครับ โดย Axios เตรียมฟังก์ชันที่ตรงกับมาตรฐาน HTTP ไว้ให้เราครบถ้วน:
*   **`axios.get(url)`:** ใช้สำหรับ "ขอข้อมูล" (เช่น ขอรายชื่อสินค้า, ขอข้อมูลโปรไฟล์)
*   **`axios.post(url, data)`:** ใช้สำหรับ "ส่งข้อมูลใหม่" ไปบันทึก (เช่น ส่งข้อมูลฟอร์มสมัครสมาชิก)
*   **`axios.put(url, data)` / `axios.patch(url, data)`:** ใช้สำหรับ "อัปเดตข้อมูลเดิม"
*   **`axios.delete(url)`:** ใช้สำหรับ "ลบข้อมูล"

### ⏳ 3. จังหวะเวลา (Async / Await)
การวิ่งไปขอข้อมูลจาก Server "ต้องใช้เวลา" ครับ (อาจจะ 1 วินาที หรือ 5 วินาที ขึ้นอยู่กับอินเทอร์เน็ต) ถ้าเราไม่รอให้ข้อมูลมาถึงก่อนแล้วชิงแสดงผล หน้าเว็บเราก็จะพัง (Error: Undefined) 

เราจึงต้องใช้คู่หูคีย์เวิร์ด **`async`** (บอกว่าฟังก์ชันนี้มีการทำงานแบบไม่ซิงโครนัส) และ **`await`** (สั่งให้โค้ด "หยุดรอ" ตรงบรรทัดนี้จนกว่า Axios จะดึงข้อมูลเสร็จ แล้วค่อยทำบรรทัดต่อไป)

{{< figure src="vue-axios-api-integration-diagram.jpg" alt="Diagram showing the Vue.js Component making HTTP GET and POST requests using Axios to a Backend API Server" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code)

เรามาดูตัวอย่างการสร้าง Component "รายชื่อพนักงาน" ที่มีการดึงข้อมูลตอนเปิดหน้าเว็บ (GET) และมีฟอร์มสำหรับเพิ่มพนักงานใหม่ (POST) โดยใช้ **Composition API** ร่วมกับ **Lifecycle Hook (`onMounted`)** ครับ

```vue
<template>
  <div class="user-dashboard">
    <h2>รายชื่อพนักงานจาก API 🌐</h2>

    <!-- ส่วนที่ 1: ฟอร์มเพิ่มพนักงาน (POST) -->
    <form @submit.prevent="submitUser" class="add-form">
      <input v-model="newUser" placeholder="กรอกชื่อพนักงานใหม่..." required />
      <!-- ปิดปุ่มตอนกำลังโหลด ป้องกันการกดเบิ้ล -->
      <button type="submit" :disabled="isSubmitting">
        {{ isSubmitting ? 'กำลังบันทึก...' : 'เพิ่มพนักงาน' }}
      </button>
    </form>

    <hr />

    <!-- ส่วนที่ 2: แสดงสถานะ Loading, Error และ Data (GET) -->
    <div v-if="isLoading" class="loading">กำลังโหลดข้อมูล กรุณารอสักครู่... ⏳</div>
    
    <div v-else-if="errorMessage" class="error">
      ❌ เกิดข้อผิดพลาด: {{ errorMessage }}
    </div>

    <ul v-else class="user-list">
      <li v-for="user in users" :key="user.id">
        {{ user.name }} (ID: {{ user.id }})
      </li>
    </ul>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios' // อัญเชิญ Axios เข้ามาในไฟล์

// --- State (ตัวแปรเก็บข้อมูล) ---
const users = ref([])
const newUser = ref('')
const isLoading = ref(false)
const isSubmitting = ref(false)
const errorMessage = ref(null)

// URL ของ API ปลอมสำหรับทดสอบ
const API_URL = 'https://jsonplaceholder.typicode.com/users'

// --- ฟังก์ชันดึงข้อมูล (GET) ---
const fetchUsers = async () => {
  isLoading.value = true // 1. เปิดสถานะ Loading
  errorMessage.value = null

  try {
    // 2. สั่ง await เพื่อรอรับข้อมูลจาก API
    const response = await axios.get(API_URL)
    // 3. ข้อมูลที่ตอบกลับมาจะซ่อนอยู่ใน response.data เสมอ!
    users.value = response.data 
  } catch (error) {
    // 4. ดักจับ Error (เช่น เน็ตหลุด, Server พัง)
    errorMessage.value = 'ไม่สามารถดึงข้อมูลได้ โปรดลองอีกครั้ง'
    console.error('Fetch Error:', error)
  } finally {
    // 5. ไม่ว่าจะสำเร็จหรือพัง ก็ต้องปิด Loading เสมอ
    isLoading.value = false 
  }
}

// --- ฟังก์ชันเพิ่มข้อมูล (POST) ---
const submitUser = async () => {
  isSubmitting.value = true
  
  try {
    // ส่งข้อมูลเป็น Object ไปพร้อมกับ URL
    const response = await axios.post(API_URL, {
      name: newUser.value
    })
    
    // เมื่อ Server ตอบกลับมาว่าสร้างสำเร็จ (มักจะได้ Data พร้อม ID ใหม่กลับมา)
    // เราก็เอามาดันเข้า Array เพื่อให้อัปเดตหน้าจอทันทีโดยไม่ต้อง Refresh หน้าเว็บ
    users.value.unshift(response.data) 
    newUser.value = '' // เคลียร์ช่องกรอกข้อมูล
    alert('เพิ่มพนักงานสำเร็จ!')
  } catch (error) {
    alert('เกิดข้อผิดพลาดในการบันทึกข้อมูล')
    console.error('Post Error:', error)
  } finally {
    isSubmitting.value = false
  }
}

// --- เรียกใช้ฟังก์ชันตอนที่ Component เกิดขึ้นมาบนหน้าจอ (Lifecycle Hook) ---
onMounted(() => {
  fetchUsers()
})
</script>

<style scoped>
.error { color: red; }
.loading { color: blue; font-style: italic; }
.add-form { margin-bottom: 20px; }
</style>
```

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips)

เมื่อโปรเจกต์ของน้องๆ ใหญ่ขึ้น การมานั่งพิมพ์ URL เต็มๆ หรือเขียน Try/Catch ซ้ำๆ ทุกไฟล์คงไม่ใช่เรื่องสนุก นี่คือ "ท่าไม้ตายระดับ Senior" ในการจัดการ Axios ครับ:

**💡 Pro-Tip 1: สร้าง Axios Instance (โรงงานผลิตจดหมาย)**
แทนที่เราจะ `import axios from 'axios'` ตรงๆ ในทุกไฟล์ เราควรสร้างไฟล์ `src/api/index.js` เพื่อตั้งค่า "ที่อยู่ตั้งต้น (Base URL)" ครับ:
```javascript
// src/api/index.js
import axios from 'axios';

const api = axios.create({
  baseURL: 'https://api.mycompany.com/v1', // เปลี่ยน URL แค่ที่เดียวตรงนี้!
  timeout: 5000 // ถ้าโหลดเกิน 5 วินาทีให้ตัดจบเลย (ป้องกันเว็บค้าง)
});

export default api;
```
เวลาเอาไปใช้ใน Component ก็แค่ `import api from '@/api'` แล้วเรียก `api.get('/users')` โค้ดจะสั้นลงและดูแลรักษาง่ายขึ้นมากครับ!

**🚨 Pro-Tip 2: Interceptors (รปภ. ประจำ API)**
หลายครั้งที่ API ของเราต้องมีการแนบ "กุญแจยืนยันตัวตน (Auth Token)" ไปกับทุกๆ Request ด้วย Axios เราสามารถตั้งด่านตรวจ (Interceptor) ให้มันแอบยัด Token ใส่จดหมายให้เราอัตโนมัติก่อนส่งออกไปได้เลยครับ:
```javascript
// แอบแนบ Token ไปกับทุก Request
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});
```

## 6. 🏁 บทสรุป (To be continued...)
**Axios** ผสมผสานกับ **Async/Await** คือกระบวนท่ามาตรฐานในการเชื่อมต่อ Frontend เข้ากับ Backend ยุคใหม่ครับ สิ่งสำคัญที่พี่อยากย้ำคือ อย่าลืมจัดการ "สภาวะอารมณ์" ของหน้าเว็บ (Loading, Error, Success) ให้ดีเสมอ เพื่อให้ผู้ใช้งาน (User Experience) รู้สึกว่าแอปพลิเคชันของเราตอบสนองอยู่ตลอดเวลา

ตอนนี้เราสามารถดึงข้อมูลจาก Server มาได้แล้ว แต่ถ้าข้อมูลก้อนนี้มันต้องถูกแชร์ไปให้หน้าอื่นๆ ใช้ด้วยล่ะ? จะให้ยิง API ใหม่ทุกรอบ หรือจะส่ง Props ข้ามไปข้ามมาทีละ 5 ชั้นก็คงไม่ไหว! 

ในบทต่อไป เราจะมากางคัมภีร์วิชา **State Management** ทำความรู้จักกับ **Pinia** (คลังเสบียงส่วนกลางของ Vue) ที่จะมาช่วยเราเก็บข้อมูลจาก API ไว้แชร์ใช้ร่วมกันได้ทั้งแอปพลิเคชัน เตรียมตัวก้าวขึ้นสู่ความเป็น Senior กันได้เลยครับ!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Web Application หรือ Frontend Architecture ให้กับธุรกิจของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและพัฒนาซอฟต์แวร์แบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p