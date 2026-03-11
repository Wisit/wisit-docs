---
title: "จัดการสถานะแอปด้วย Pinia: State Management ยุคใหม่ที่ง่ายกว่า Vuex"
weight: 260
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Vue3", "Pinia", "State Management", "Vuex", "Frontend Architecture"]
---
{{< figure src="vue-pinia-state-management-cover.jpg" alt="Aesthetic study notes about Pinia State Management in Vue 3 on an iPad screen" >}}

## 1. 🎯 ตอนที่ 26: จัดการสถานะแอปด้วย Pinia คลังเสบียงส่วนกลางแห่งยุคใหม่

## 2. 📖 เปิดฉาก (The Hook)
น้องๆ เคยเจอสภาวะ "โค้ดพันกันเป็นสายหูฟัง" ไหมครับ? ลองจินตนาการว่าเรากำลังทำเว็บ E-commerce ที่มีหน้าตระกร้าสินค้า (Cart) อยู่ที่มุมขวาบนของแถบนำทาง (Navbar) แล้วในหน้าแสดงสินค้า เรามีปุ่ม "เพิ่มลงตะกร้า" 

คำถามคือ... พอผู้ใช้กดปุ่มปุ๊บ ตัวเลขใน Navbar จะอัปเดตตามได้อย่างไร? ถ้าเราใช้แค่ Props กับ Emits (ส่งข้อมูลจากแม่ไปลูก ลูกฟ้องแม่) โค้ดของเราจะต้องส่งข้อมูลข้าม Component กระโดดไปมาเป็นสิบๆ ชั้น (เราเรียกปัญหานี้ว่า Prop Drilling) 

ในอดีตเราแก้ปัญหานี้ด้วย **Vuex** ซึ่งเป็นไลบรารีจัดการสถานะส่วนกลางที่ทรงพลังมาก แต่มันก็แลกมากับความน่าปวดหัวในการเขียนโค้ดที่ยืดยาว (Verbosity) ต้องมานั่งเขียน Mutations วุ่นวายไปหมด 

จนกระทั่งในเดือนกุมภาพันธ์ ปี 2022 ทีมพัฒนา Vue ได้ประกาศเปิดตัว **Pinia** (สัญลักษณ์รูปสับปะรด) ให้เป็น State Management ตัวหลักอย่างเป็นทางการแทนที่ Vuex (Evan You ผู้สร้าง Vue ถึงกับเรียกมันว่า "de facto Vuex 5" เลยทีเดียว) วันนี้พี่จะพามาจิบกาแฟ และทำความรู้จักกับคลังเสบียงตัวใหม่นี้กันครับ รับรองว่าเขียนง่ายและสนุกกว่าเดิมเยอะ!

## 3. 🧠 แก่นวิชา (Core Concepts)

**Pinia** คือ State Management Library ที่ทำหน้าที่เป็น "คลังเสบียงส่วนกลาง (Centralized Store)" ให้กับแอปพลิเคชันของเรา Component ไหนอยากได้ข้อมูล ก็แค่เดินมาเบิกที่โกดังนี้ ไม่ต้องฝากของข้ามชั้นกันอีกต่อไป

### 🍍 ทำไม Pinia ถึงครองใจนักพัฒนา (Pinia vs Vuex)?
*   **บอกลา Mutations (No Mutations):** ใน Vuex เราต้องอัปเดต State ผ่าน Mutations เท่านั้น ซึ่งน่ารำคาญมาก แต่ใน Pinia เราสามารถให้ Actions ดัดแปลง State ได้โดยตรงเลย! โค้ดสั้นลงมหาศาล
*   **TypeScript Support ขั้นเทพ:** Pinia เกิดมาเพื่อ TS อย่างแท้จริง มี Autocomplete เดาโค้ดให้เราแม่นยำสุดๆ
*   **ไม่ต้องมี Nested Modules:** ไม่ต้องมาปวดหัวกับการจัดโครงสร้าง Module ซ้อน Module แบบ Vuex อีกแล้ว ใน Pinia ทุก Store เป็นอิสระแยกจากกัน (แต่สามารถเรียกใช้ข้ามกันได้)
*   **รองรับ Composition API เต็มรูปแบบ:** เขียนด้วย `<script setup>` ได้อย่างสวยงาม

### 🏛️ โครงสร้างของ Store (The 3 Pillars)
Pinia ถูกออกแบบมาให้เหมือนกับ Component เป๊ะๆ โดยประกอบไปด้วย 3 เสาหลัก:
1.  📦 **State (โกดังเก็บของ):** เทียบได้กับ `data` หรือ `ref()` ใน Component มีหน้าที่เก็บข้อมูลดิบ
2.  🧮 **Getters (นักบัญชี):** เทียบได้กับ `computed` มีหน้าที่นำ State มาคำนวณหรือกรอง แล้วส่งผลลัพธ์ออกไป (เช่น นับจำนวนของในตะกร้า)
3.  🛠️ **Actions (ผู้จัดการโกดัง):** เทียบได้กับ `methods` เป็นฟังก์ชันสำหรับแก้ไข State และสามารถทำงานแบบ Asynchronous (เช่น ยิง API ไปหลังบ้าน) ได้ด้วย

{{< figure src="vue-pinia-state-management-diagram.jpg" alt="Diagram showing the Pinia state management architecture with Actions, State, and Getters" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code)

เรามาลองสร้างระบบบล็อก (Blog) ง่ายๆ ที่ดึงรายชื่อบทความจาก API กันครับ 

**1. การสร้าง Store (`src/stores/post.js`)**
เราใช้ฟังก์ชัน `defineStore` ในการตั้งชื่อและกำหนดโครงสร้างของโกดังนี้ครับ
```javascript
import { defineStore } from 'pinia'

// ขึ้นต้นชื่อด้วย use... เสมอเป็นธรรมเนียมปฏิบัติ
export const usePostStore = defineStore({
  id: 'post', // ชื่อ ID ต้องไม่ซ้ำกันในแอป
  
  // 1. State: คืนค่า Object ที่เก็บข้อมูลเริ่มต้น
  state: () => ({
    posts: [],
    loading: false,
    error: null
  }),

  // 2. Getters: รับ State เข้ามาเพื่อคำนวณและส่งค่าออกไป
  getters: {
    // ดึงบทความเฉพาะของนักเขียนคนนั้นๆ (สร้างเป็น Higher-order function)
    getPostsByAuthor: (state) => {
      return (authorId) => state.posts.filter((post) => post.userId === authorId)
    }
  },

  // 3. Actions: ทำงานกับ API และเปลี่ยนแปลง State ได้โดยตรง (ใช้ this)
  actions: {
    async fetchPosts() {
      this.posts = []
      this.loading = true
      try {
        const response = await fetch('https://jsonplaceholder.typicode.com/posts')
        this.posts = await response.json() // เปลี่ยนแปลง State โดยตรงได้เลย!
      } catch (error) {
        this.error = error
      } finally {
        this.loading = false
      }
    }
  }
})
```

**2. การเรียกใช้ Store ใน Component (`src/views/PostsView.vue`)**
เราจะนำโกดังมาใช้ใน Component ผ่าน Composition API ครับ
```vue
<template>
  <main class="posts-container">
    <h2>บทความล่าสุด 📝</h2>
    
    <p v-if="loading" class="loading">กำลังโหลดบทความ...</p>
    <p v-if="error" class="error">{{ error.message }}</p>
    
    <ul v-if="posts.length > 0">
      <li v-for="post in posts" :key="post.id">
        <h3>{{ post.title }}</h3>
        <p>{{ post.body }}</p>
      </li>
    </ul>
  </main>
</template>

<script setup>
import { storeToRefs } from 'pinia'
import { usePostStore } from '../stores/post'

// 1. สร้าง Instance ของ Store
const postStore = usePostStore()

// 2. 🚨 ดึง State และ Getters ออกมาใช้งาน (ต้องใช้ storeToRefs เท่านั้น!)
const { posts, loading, error } = storeToRefs(postStore)

// 3. ดึง Actions ออกมาใช้งาน (Destructure ดึงฟังก์ชันมาใช้ตรงๆ ได้เลย)
const { fetchPosts } = postStore

// 4. สั่งผู้จัดการโกดังให้ไปดึงข้อมูลมา!
fetchPosts()
</script>

<style scoped>
.loading { color: blue; }
.error { color: red; }
</style>
```

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips)

มีจุดหลุมพรางที่โปรแกรมเมอร์หลายคนตกม้าตายตอนเปลี่ยนมาใช้ Pinia ครับ พี่ขอเตือนไว้ตรงนี้เลย:

**🚨 กฎเหล็กแห่ง Reactivity: ห้าม Destructure State ตรงๆ เด็ดขาด!**
ใน JavaScript ปกติ เราชอบดึงข้อมูลออกจาก Object ด้วยการเขียนแบบนี้ใช่ไหมครับ:
`const { posts } = usePostStore()` ❌
**อย่าทำแบบนี้กับ State เด็ดขาดครับ!** เพราะทันทีที่คุณแกะมันออกมาแบบดิบๆ ข้อมูลนั้นจะหลุดจากการเป็น Reactive ทันที (หน้าจอจะไม่อัปเดตเมื่อข้อมูลเปลี่ยน) 

**✅ ทางแก้:** ให้หุ้มมันด้วยฟังก์ชัน **`storeToRefs()`** เสมอเมื่อต้องการดึง State หรือ Getters ออกมา มันจะช่วยแปลงข้อมูลให้กลายเป็น `ref()` และรักษาความสามารถในการอัปเดตหน้าจอเอาไว้ครับ *(หมายเหตุ: ยกเว้น Actions สามารถดึงออกมาตรงๆ ได้เลย ไม่ต้องใช้ storeToRefs เพราะฟังก์ชันไม่ต้องทำ Reactivity)*

**🤝 การขอยืมของข้ามโกดัง (Cross-Store Usage)**
ถ้าเรามี Store จัดการ "ความคิดเห็น (Comments)" แล้วเราอยากรู้ว่าตอนนี้มี "บทความ (Post)" ไหนกำลังเปิดอยู่ Pinia อนุญาตให้เรา Import Store ตัวอื่นเข้ามาเรียกใช้ใน Getters หรือ Actions ของอีก Store ได้อย่างอิสระเลยครับ ไม่ต้องตั้งค่าเชื่อม Module ให้วุ่นวายเหมือนสมัย Vuex แล้ว!

## 6. 🏁 บทสรุป (To be continued...)
**Pinia** คือคำตอบที่สวยงามที่สุดสำหรับปัญหา State Management ในปัจจุบันครับ ด้วยโครงสร้างที่เบาหวิว โค้ดที่อ่านง่าย และการตัด Mutations ทิ้งไป ทำให้เราสามารถจัดการข้อมูลส่วนกลางของแอปพลิเคชันสเกลใหญ่ได้อย่างสนุกสนานและเป็นระเบียบ

ตอนนี้แอปพลิเคชันของเรามีระบบจัดการข้อมูลที่แข็งแกร่งแล้ว! ในบทความหน้า เราจะเจาะลึกเข้าไปในโลกของ **Architecture และ Best Practices** ว่าเมื่อโปรเจกต์ของเราใหญ่ระดับ Enterprise เราควรจะจัดเรียงโฟลเดอร์ หรือแยก Component อย่างไรให้ทีมทำงานร่วมกันได้โดยไม่ตีกัน เตรียมตัวก้าวขึ้นสู่ระดับ Senior กันเต็มตัวได้เลยครับ!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Web Application หรือ Frontend Architecture ให้กับธุรกิจของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและพัฒนาซอฟต์แวร์แบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p