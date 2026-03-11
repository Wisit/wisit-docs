---
title: "เจาะลึก Pinia ระดับสูง: การสื่อสารระหว่าง Stores และการทำ Persistence"
weight: 265
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Vue3", "Pinia", "State Management", "LocalStorage", "VueUse", "Frontend Architecture"]
---
{{< figure src="vue-pinia-advanced-persistence-cover.jpg" alt="Aesthetic study notes about Advanced Pinia State Management on an iPad screen" >}}

## 1. 🎯 ตอนที่ 36: เจาะลึก Pinia ระดับสูง การสื่อสารข้ามโกดังและเคล็ดวิชาจำฝังใจ (Persistence)

## 2. 📖 เปิดฉาก (The Hook)
น้องๆ ครับ หลังจากที่เราเปลี่ยนจาก Vuex มาใช้ Pinia ชีวิตเราก็ง่ายขึ้นเยอะเลยใช่ไหมครับ? แต่เมื่อโปรเจกต์ของเราใหญ่ขึ้นระดับ Enterprise เราจะเริ่มเจอโจทย์ที่ท้าทายขึ้น 

ลองจินตนาการว่าเรากำลังทำระบบ E-commerce โกดังที่ 1 (`UserStore`) เก็บข้อมูลว่าลูกค้าคนนี้เป็นสมาชิกระดับ VIP หรือไม่ โกดังที่ 2 (`CartStore`) เก็บสินค้าในตะกร้า คำถามคือ... **ถ้า `CartStore` อยากจะคำนวณส่วนลดโดยต้องเช็กสถานะ VIP จาก `UserStore` มันจะข้ามไปคุยกันยังไง?** ในยุค Vuex เราต้องมานั่งปวดหัวกับ `rootState` และ Modules ที่ซ้อนกันวุ่นวายไปหมด

อีกปัญหาที่คลาสสิกสุดๆ คือ **"กด F5 แล้วตะกร้าสินค้าหายวับไปกับตา!"** เพราะ State ใน Pinia มันเก็บอยู่ใน Memory ของเบราว์เซอร์ พอรีเฟรชหน้าเว็บ ทุกอย่างก็ถูกล้างใหม่หมด เราจะทำยังไงให้ Pinia มัน "จำฝังใจ" ดึงข้อมูลกลับมาได้อัตโนมัติ? 

วันนี้พี่จะพามาจิบกาแฟ และปลดล็อกวิชาขั้นสูงของ Pinia 2 กระบวนท่า นั่นคือ **Cross-Store Communication** และการทำ **State Persistence** รับรองว่าโค้ดของน้องๆ จะทรงพลังขึ้นอีกระดับครับ!

## 3. 🧠 แก่นวิชา (Core Concepts)

### 🤝 1. การขอยืมของข้ามโกดัง (Cross-Store Communication)
ความสวยงามของ Pinia คือมันถูกออกแบบมาให้ **"ไม่มี Modules ซ้อนกัน (No Nested Modules)"** ทุก Store มีความอิสระเท่าเทียมกัน (Flat architecture) ดังนั้นถ้า Store A อยากได้ข้อมูลจาก Store B เราก็แค่ `import` Store B เข้ามาใช้งานข้างใน Getter หรือ Action ของ Store A ได้ตรงๆ เลยครับ! เรียบง่ายและตรงไปตรงมาสุดๆ

### 💾 2. วิชาจำฝังใจด้วย LocalStorage (State Persistence)
การจะทำให้ State อยู่ทนทานแม้ User จะปิดเบราว์เซอร์หรือรีเฟรชหน้าเว็บ เราต้องเก็บข้อมูลลงใน `localStorage` ของเบราว์เซอร์ครับ 
แทนที่เราจะต้องมานั่งเขียน `localStorage.setItem` และ `getItem` เองให้เมื่อยมือ ในโลกของ Vue 3 เรามีตัวช่วยระดับเทพที่ชื่อว่าไลบรารี **VueUse** (ชุดรวม Composables ยอดฮิต) ซึ่งมีฟังก์ชัน `useLocalStorage()` ที่สามารถเสกให้ตัวแปร Reactive ของเรา ซิงก์ข้อมูลกับ LocalStorage ได้แบบอัตโนมัติ!

{{< figure src="vue-pinia-advanced-persistence-diagram.jpg" alt="Diagram showing Advanced Pinia architecture with Cross-Store Communication and LocalStorage Persistence" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code)

เรามาดูการประกอบร่างทั้ง 2 วิชาเข้าด้วยกันครับ! พี่จะใช้รูปแบบการเขียน Pinia แบบ **Setup Store** (คล้าย Composition API) เพราะมันใช้ร่วมกับ VueUse ได้สวยงามที่สุด

**Step 1: ติดตั้ง VueUse (ถ้ายังไม่มี)**
```bash
npm install @vueuse/core
```

**Step 2: สร้างโกดังเก็บผู้ใช้งาน (`src/stores/user.js`)**
โกดังนี้ทำหน้าที่ง่ายๆ คือเก็บข้อมูลผู้ใช้ปัจจุบัน
```javascript
import { defineStore } from 'pinia'
import { ref } from 'vue'

export const useUserStore = defineStore('user', () => {
  // สร้าง State เก็บชื่อและสถานะ VIP
  const currentUser = ref({
    name: 'น้องสมชาย',
    isVip: true
  })

  return { currentUser }
})
```

**Step 3: สร้างโกดังตะกร้าสินค้า (`src/stores/cart.js`) 🌟 พระเอกของเรา**
ที่นี่เราจะดึง `useUserStore` มาใช้คำนวณส่วนลด และใช้ `useLocalStorage` เพื่อจำสินค้าในตะกร้า!

```javascript
import { defineStore } from 'pinia'
import { computed } from 'vue'
import { useLocalStorage } from '@vueuse/core'
import { useUserStore } from './user' // 1. นำเข้าโกดังเพื่อนบ้าน

export const useCartStore = defineStore('cart', () => {
  // 2. 💾 State Persistence: ใช้ useLocalStorage แทน ref() ปกติ
  // อาร์กิวเมนต์แรกคือ 'Key' ใน LocalStorage, อาร์กิวเมนต์สองคือ 'ค่าเริ่มต้น'
  // เมื่อค่า items เปลี่ยน มันจะ Save ลง LocalStorage อัตโนมัติ!
  const items = useLocalStorage('my-shop-cart', []) 

  // 3. 🤝 Cross-Store Communication: ดึงโกดัง User มาใช้ใน Getter
  const cartTotalWithDiscount = computed(() => {
    // คำนวณราคารวมปกติ
    const total = items.value.reduce((sum, item) => sum + item.price, 0)
    
    // เรียกใช้ User Store
    const userStore = useUserStore()
    
    // ถ้าเป็น VIP ลดให้ 20% ไปเลย!
    if (userStore.currentUser.isVip) {
      return total * 0.8 
    }
    return total
  })

  // Action สำหรับเพิ่มสินค้า
  const addItem = (product) => {
    items.value.push(product)
  }

  return { items, cartTotalWithDiscount, addItem }
})
```

**Step 4: เรียกใช้ใน Component ปกติเลย!**
```vue
<template>
  <div class="checkout-box">
    <h2>ตะกร้าสินค้า ({{ cartStore.items.length }} ชิ้น)</h2>
    <!-- ราคานี้จะถูกคำนวณส่วนลด VIP มาแล้วเรียบร้อย! -->
    <h3>ยอดชำระสุทธิ: {{ cartStore.cartTotalWithDiscount }} บาท</h3>
    <button @click="addToCart">เพิ่มสินค้าตัวอย่าง</button>
  </div>
</template>

<script setup>
import { useCartStore } from '@/stores/cart'

const cartStore = useCartStore()

const addToCart = () => {
  cartStore.addItem({ id: 1, name: 'คีย์บอร์ด', price: 1000 })
}
</script>
```
*ลองรันโค้ดแล้วกดเพิ่มสินค้าดูครับ จากนั้นกดรีเฟรชหน้าเว็บ (F5) จะเห็นว่าของในตะกร้ายังอยู่ครบ! และราคาลด 20% ก็ทำงานได้อย่างถูกต้อง แม้จะแยก Store กันก็ตาม!*

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips)

**🚨 1. ระวังวงจรอุบาทว์ (Circular Dependencies)**
การเรียก Store ข้ามกันเป็นเรื่องดีครับ แต่ต้องระวังอย่าให้ **Store A เรียก Store B** และในขณะเดียวกัน **Store B ก็เรียก Store A** ในระดับ Global Scope เด็ดขาด! เพราะระบบจะไม่รู้ว่าต้องสร้างใครก่อนและพังในที่สุด (Infinite Loop)
*💡 Pro-Tip:* ถ้าจำเป็นต้องเรียกกันเองจริงๆ ให้หลีกเลี่ยงการเรียกใช้ตอนประกาศ State แต่ให้ไปเรียก `const storeA = useStoreA()` **ซ่อนไว้ข้างใน** Getter หรือ Action แทนครับ เพื่อให้มันเรียกใช้เมื่อถูกกระทำเท่านั้น (Lazy Instantiation)

**🔒 2. ความปลอดภัยของ LocalStorage**
การใช้ `useLocalStorage` สะดวกก็จริง แต่พึงระลึกไว้เสมอว่า ข้อมูลใน LocalStorage สามารถถูกอ่านและแก้ไขได้โดยผู้ใช้หรือ Script อื่น (ถ้าเว็บน้องโดนโจมตีแบบ XSS) **ห้าม!** เก็บข้อมูลที่เป็นความลับเด็ดขาด เช่น รหัสผ่านแบบเพียวๆ (Plain-text) หรือ Token สำคัญๆ หากไม่ได้เข้ารหัสไว้ครับ

**🛠️ 3. ปลั๊กอิน Pinia Plugin Persistedstate**
ถ้าโปรเจกต์ของน้องเขียน Pinia แบบ Options API (แบบเก่า) หรือต้องการความสามารถระดับสูง (เช่น อยากเลือกว่าจะเซฟลง `sessionStorage` แทน หรืออยากเซฟแค่ State บางตัว) พี่แนะนำให้ติดตั้งปลั๊กอินยอดฮิตตัวนี้ครับ: `pinia-plugin-persistedstate` มันจะช่วยให้เราตั้งค่า Persistence ผ่าน Property `persist: true` ในตอนประกาศ Store ได้เลย ง่ายสุดๆ!

## 6. 🏁 บทสรุป (To be continued...)
ด้วยสถาปัตยกรรมที่แบนราบ (Flat) ของ Pinia ทำให้การ **Cross-Store Communication** เป็นเรื่องง่ายเหมือนปอกกล้วยเข้าปาก เราไม่ต้องมานั่งปวดหัวเรื่อง Scope หรือ Namespaces อีกต่อไป และเมื่อรวมพลังกับไลบรารีอย่าง **VueUse** เราก็สามารถทำ **State Persistence** ให้แอปพลิเคชันของเราจำข้อมูลผู้ใช้ได้เพียงการเปลี่ยนโค้ดแค่บรรทัดเดียว!

เมื่อ State Management ของเราแข็งแกร่งขนาดนี้แล้ว ในบทความหน้า เราจะกลับมาดูเรื่องของการจัดการ API กันบ้างครับ ว่าถ้าเราต้องยิง API เยอะๆ เราจะใช้สถาปัตยกรรมแบบไหนเพื่อแยก Logic เหล่านี้ออกจาก UI Component ให้โค้ดเราคลีนที่สุด เตรียมตัวพบกับ **Repository Pattern** ได้เลยครับ!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Web Application หรือ Frontend Architecture ให้กับธุรกิจของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและพัฒนาซอฟต์แวร์แบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p