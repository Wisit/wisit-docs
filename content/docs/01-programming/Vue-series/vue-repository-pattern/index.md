---
title: "สถาปัตยกรรมระดับสูง: การใช้ Repository Pattern ใน Vue.js"
weight: 340
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Vue3", "Architecture", "Repository Pattern", "Design Pattern", "Enterprise"]
---
{{< figure src="vue-repository-pattern-cover.jpg" alt="Aesthetic study notes about Repository Pattern in Vue.js on an iPad screen" >}}

## 1. 🎯 ตอนที่ 34: สถาปัตยกรรมระดับสูง: การใช้ Repository Pattern เพื่อแยกส่วน API ออกจาก UI

## 2. 📖 เปิดฉาก (The Hook)
น้องๆ เคยเจอสถานการณ์แบบนี้ไหมครับ? เรามี Component หรือ Vuex/Pinia Action ที่ทำหน้าที่ดึงข้อมูลผู้ใช้งาน เราก็เลยเขียนโค้ด `axios.get('/api/users')` แปะลงไปตรงๆ เลย 

ตอนแรกมันก็ดูง่ายดีครับ แต่พอโปรเจกต์เรามีขนาดใหญ่ขึ้นระดับ Enterprise มีสัก 50 Components ที่เรียกใช้ API ยิบย่อยเต็มไปหมด วันดีคืนดี ทีม Backend เดินมาบอกว่า *"น้อง... พี่ขอเปลี่ยน URL จาก `/api/users` เป็น `/api/v2/members` นะ"* หรือแย่กว่านั้น หัวหน้าทีมสั่งว่า *"เราจะเลิกใช้ Axios แล้ว เปลี่ยนไปใช้ Fetch API แทนนะ"*... ช็อกสิครับ! เราต้องมานั่งไล่ค้นหา (Find and Replace) โค้ดทั่วทั้งโปรเจกต์ เสี่ยงต่อการพัง (Regression bugs) แบบสุดๆ

นี่คือสาเหตุที่หนังสือ **"Architecting Vue.js 3"** ของคุณ Solomon Eseme แนะนำให้เราก้าวข้ามการเขียนโค้ดแบบเดิมๆ แล้วหันมาใช้สถาปัตยกรรมที่เรียกว่า **"Repository Pattern"** ครับ วันนี้พี่จะพามาจิบกาแฟ แล้วดูวิธีสร้าง "เกราะป้องกัน" ไม่ให้โค้ดส่วน UI ของเราต้องมานั่งปวดหัวกับการเปลี่ยนแปลงของ API หลังบ้านกันครับ!

## 3. 🧠 แก่นวิชา (Core Concepts)

### 📦 Repository Pattern คืออะไร?
**Repository Pattern** คือรูปแบบการออกแบบซอฟต์แวร์ (Design Pattern) ที่ทำหน้าที่เป็น "คนกลาง" ระหว่าง **Business Logic (หรือ UI Component)** กับ **Data Source (เช่น API หรือ Database)** 

ลองเปรียบเทียบง่ายๆ ครับ:
*   **UI Component** คือ "ลูกค้า" ที่มาสั่งอาหาร
*   **Repository** คือ "ผู้จัดการร้าน" ที่รับออเดอร์
*   **Axios/Fetch** คือ "พ่อครัว" ที่ลงมือทำอาหาร

ลูกค้า (UI) ไม่จำเป็นต้องรู้เลยว่าพ่อครัว (Axios) หั่นผักยังไง หรือใช้กระทะยี่ห้ออะไร ลูกค้าแค่บอกผู้จัดการร้าน (Repository) ว่า "ขอรายชื่อผู้ใช้หน่อย" แล้วเดี๋ยวผู้จัดการไปจัดการเอามาให้

### ✨ ข้อดีของการทำแบบนี้ (อ้างอิงจากคัมภีร์ Architecting Vue.js 3)
1. **Reusable (นำไปใช้ซ้ำได้ทุกที่):** โค้ดส่วนดึงข้อมูลถูกรวมไว้ที่เดียว Component ไหนอยากได้ก็แค่เรียกใช้
2. **Decoupling (แยกส่วนการทำงาน):** UI Component จะไม่ผูกติด (Tight coupling) กับ Axios อีกต่อไป
3. **Testable (ทดสอบง่าย):** เราสามารถจำลอง (Mock) ข้อมูลจาก Repository เพื่อทำ Unit Test ให้กับ Component ได้ง่ายมาก โดยไม่ต้องยิง API จริงๆ

{{< figure src="vue-repository-pattern-diagram.jpg" alt="Diagram showing Vue.js Repository Pattern architecture" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code)

ในโปรเจกต์ขนาดใหญ่ เราจะแบ่งโครงสร้างโฟลเดอร์ออกมาเป็น `src/repositories` และนี่คือขั้นตอนการประกอบร่างครับ

**Step 1: สร้างตัวจัดการ HTTP Client พื้นฐาน (`src/repositories/Clients/AxiosClient.js`)**
เราจะสร้างไฟล์นี้เพื่อตั้งค่า Axios (เช่น Base URL หรือ Headers) ไว้ที่เดียว
```javascript
import axios from 'axios';

const baseDomain = 'https://api.mycompany.com';
const baseURL = `${baseDomain}/v1`;

export default axios.create({
  baseURL,
  headers: {
    'Accept': 'application/json',
    'Content-Type': 'application/json'
  }
});
```

**Step 2: สร้าง Repository ประจำแต่ละฟีเจอร์ (`src/repositories/UserRepository.js`)**
ไฟล์นี้คือ "ผู้จัดการร้าน" ที่ดูแลเรื่อง User โดยเฉพาะ จะเก็บแค่ฟังก์ชัน (Endpoints) ที่เกี่ยวกับ User ครับ
```javascript
import Client from './Clients/AxiosClient';

// กำหนด URL ก้อนท้ายสุด
const resource = '/users';

export default {
  // ฟังก์ชันขอข้อมูลทั้งหมด (GET)
  get() {
    return Client.get(`${resource}`);
  },
  // ฟังก์ชันขอข้อมูลรายบุคคล (GET by ID)
  getUser(id) {
    return Client.get(`${resource}/${id}`);
  },
  // ฟังก์ชันสร้างผู้ใช้ใหม่ (POST)
  create(payload) {
    return Client.post(`${resource}`, payload);
  }
};
```

**Step 3: สร้างศูนย์รวม Repository (The Factory) (`src/repositories/RepositoryFactory.js`)**
นี่คือจุดที่ทำให้โค้ดเราดูโปรสุดๆ ครับ เราจะสร้าง Factory Pattern ขึ้นมาเพื่อแจกจ่าย Repository ให้กับระบบ
```javascript
import UserRepository from './UserRepository';
import PhotoRepository from './PhotoRepository';

// รวบรวมเมนูทั้งหมดไว้ในร้าน
const repositories = {
  users: UserRepository,
  photos: PhotoRepository
  // ถ้ามีสินค้า (products) ก็เอามาใส่ตรงนี้ได้เลย
};

// ส่งออกฟังก์ชันสำหรับ "หยิบ" repository ไปใช้งาน
export const RepositoryFactory = {
  get: name => repositories[name]
};
```

**Step 4: เรียกใช้งานใน Component อย่างหล่อๆ (`src/views/UserList.vue`)**
ตอนนี้ Component ของเราไม่ต้องมีคำว่า `axios` โผล่มาให้เห็นแม้แต่คำเดียวครับ!
```vue
<template>
  <div class="user-list">
    <h2 v-if="isLoading">กำลังค้นหาข้อมูล...</h2>
    <ul v-else>
      <li v-for="user in users" :key="user.id">{{ user.name }}</li>
    </ul>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { RepositoryFactory } from '@/repositories/RepositoryFactory'

// 1. สั่งให้ Factory หยิบ "ผู้จัดการร้านแผนก Users" มาให้เรา
const UserRepository = RepositoryFactory.get('users')

const users = ref([])
const isLoading = ref(false)

const fetchUsers = async () => {
  isLoading.value = true
  try {
    // 2. เรียกใช้ฟังก์ชันดึงข้อมูลได้เลย หน้าตาโค้ดอ่านง่ายเหมือนภาษาคน (Declarative)
    const { data } = await UserRepository.get()
    users.value = data
  } catch (error) {
    console.error('Error fetching users:', error)
  } finally {
    isLoading.value = false
  }
}

onMounted(() => {
  fetchUsers()
})
</script>
```

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips)

**💡 1. ศาสตร์แห่ง Dependency Injection (DI)**
การออกแบบผ่าน Repository Factory ช่วยให้โปรเจกต์รองรับแนวคิด **Dependency Injection** ได้ครับ สมมติว่าตอนพัฒนาระบบ Backend ยังทำ API ไม่เสร็จ เราสามารถสลับไปสร้าง `MockUserRepository.js` (ที่ Return ค่าเป็น JSON ปลอมๆ) แล้วเอาไปสียบใน Factory แทน `UserRepository` ตัวจริงได้เลย ทำให้ Frontend ทำงานต่อไปได้โดยไม่ต้องรอ Backend!

**✨ 2. จุดเปลี่ยนผ่านแห่งอนาคต (Future-Proofing)**
ลองจินตนาการว่าในอนาคตมีไลบรารียิง API ตัวใหม่ที่เจ๋งกว่า Axios ออกมา ถ้าเราเขียนแบบเดิม เราต้องแก้โค้ดทุกไฟล์ แต่ด้วย Repository Pattern เราแค่ไปแก้ไขที่ไฟล์ `AxiosClient.js` ไฟล์เดียว หรือสร้าง `NewClient.js` มาแทนที่ โค้ดฝั่ง Vue Component ของเราจะไม่สะเทือนเลยแม้แต่นิดเดียวครับ (นี่แหละพลังของ Separation of Concerns)

## 6. 🏁 บทสรุป (To be continued...)
การใช้ **Repository Pattern** อาจจะดูเหมือนเป็นการเพิ่มความซับซ้อนในช่วงเริ่มต้นโปรเจกต์ (Over-engineering สำหรับแอปเล็กๆ) แต่สำหรับแอปพลิเคชันระดับ Enterprise นี่คือ "รากฐาน (Foundation)" ที่ขาดไม่ได้เลยครับ มันทำให้โค้ดของเราสะอาด (Clean Code) แบ่งแยกความรับผิดชอบได้ชัดเจน และดูแลรักษาง่ายในระยะยาว

เมื่อเราจัดระเบียบชั้นข้อมูล (Data Layer) ได้อย่างสมบูรณ์แบบแล้ว ในบทความถัดไป เราจะขยับไปดูการปรับปรุงสถาปัตยกรรมด้านความเร็วในการโหลดกันบ้าง กับเทคนิค **Lazy Loading และ Code Splitting** เพื่อรีดประสิทธิภาพ Vue.js 3 ให้เร็วทะลุขีดจำกัด เตรียมตัวให้พร้อมครับ!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Web Application หรือ Frontend Architecture ให้กับธุรกิจของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและพัฒนาซอฟต์แวร์แบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p