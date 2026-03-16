# **โครงสร้างของระบบเกมเพลย์และระบบเซฟ (Game Architecture & Structure Design)**

**รายงานฉบับนี้สรุปภาพรวมในมุมมองของ วิศวกรรมซอฟต์แวร์ (Software Engineering) โดยเจาะลึกไปที่หลักการทางการเขียนโค้ด (Programming Principles), การออกแบบเชิงวัตถุ (OOP), โครงสร้างข้อมูล (Data Structures) และ Design Patterns ที่ถูกนำมาใช้ในระบบ Gameplay Loop และ Save/Load System**  
---

## **1\. Design Patterns ที่ใช้ในการวางโครงสร้าง**

### 1.1 Singleton Pattern

**ระบบหลักของเกมที่ต้องทำงานควบคู่กันไปตลอดและข้าม Scene (Persistent Systems) ถูกออกแบบโดยใช้รูปแบบ Singleton**

* **คลาสเป้าหมาย:**    
* **GameProgressManager และ**   
* **ExpeditionManager**  
* **การนำไปใช้งาน: ระบุตัวแปร public static Instance ควบคู่กับการเรียกคำสั่ง DontDestroyOnLoad(gameObject) ในจังหวะ**   
* **Awake()**  
* **ข้อดี: รับประกันว่าจะมี Manager เหล่านี้เพียง 1 ตัวในระบบตลอดเวลา ช่วยให้คลาสอื่นๆ อ้างอิงได้ง่าย (เช่น GameProgressManager.Instance.SaveGame()) โดยไม่ต้องใช้ FindObjectOfType ที่เปลืองทรัพยากร**

### 1.2 Observer Pattern (Event-Driven Architecture)

**การออกแบบให้ระบบต่างๆ ตอบสนองกริยาของระบบอื่น โดยไม่ต้องผูกมัดโค้ดกันตัวต่อตัว (Decoupling)**

* **ตัวอย่าง 1: UnityEvent ใน**   
* **ExpeditionManager (เช่น onDayStart, onNightBegin) ทำให้ตัวจับเวลาไม่จำเป็นต้องรู้จักคลาส DangerousMobSpawner หรือ DayNightLighting โดยตรง แต่เป็นการเปิดช่องให้คลาสเหล่านั้น "มาดักจับ (Listen)" อีเวนต์เอาเองใน Start**  
* **ตัวอย่าง 2: กดยกเลิกและลงทะเบียนฟังก์ชันเข้ากับ C\# Action / Delegate ของ Unity อย่าง SceneManager.sceneLoaded \+= OnSceneLoaded 	เพื่อรอฟังสัญญาณการโหลดจบทันที**

---

## **2\. Object-Oriented Programming (OOP) Principles**

### 2.1 Encapsulation (การแคปซูลข้อมูล)

**การซ่อนข้อมูลที่อ่อนไหวและเปิดเผยเฉพาะฟังก์ชันที่จำเป็น เพื่อป้องกันการเข้าถึงจากภายนอกแบบผิดวิธี**

* **ตัวอย่าง: ใน**   
* **GameProgressManager ออบเจกต์ \_pendingLoadData ถูกตั้งเป็น private ไม่ยอมให้สคริปต์ภายนอกเข้ามาเสกของยัดมือผู้เล่นได้โดยตรง แต่จะต้องถูกโปรเซสและลำเลียงเข้า Inventory ภายในระบบ (internal method)**  
* **การจัดกลุ่ม Helper Method: การแยก Method เป็นสัดส่วน เช่น**   
* **GetInventoryData(),**   
* **ReadSaveData(), ขอบเขตการเข้าถึงถูกแยกชัดเจน**

### 2.2 Abstraction & Component-Based Architecture

**สืบทอดจาก MonoBehaviour ของ Unity โดยตัวระบบแยกหน้าที่ย่อยเป็น Component ให้มากที่สุด**

* **ตัวอย่าง: แบ่งหน้าที่ของเวลาให้**   
* **ExpeditionManager ตัดสินใจ แต่เวลาโหลดของหรือผู้เล่นตายให้**   
* **GameProgressManager รับผิดชอบ ต่างคนต่างทำงานในหน้าที่ย่อยชัดเจน (Single Responsibility Principle \- ส่วนหนึ่งของ SOLID)**

---

## **3\. Data Structures (โครงสร้างข้อมูล)**

**รูปแบบการเลือกใช้ Data Structure ถูกพิจารณาจาก Big-O Notation และลักษณะการนำไปใช้งาน**

### 3.1 HashSet\<string\> (การเข้าถึงแบบ O(1))

* **การใช้งาน: นำมาเก็บ defeatedBosses (บอสที่ถูกฆ่า) และ openedBoxes (กล่องสมบัติที่โดนเปิดไปแล้ว)**  
* **เหตุผลทางการออกแบบ: ในเกมเพลย์ ทุกๆ ครั้งที่โหลดฉากใหม่ หรือเดินผ่านกล่อง จะต้องมีการเช็คว่า "กล่องนี้เคยเปิดรึยัง?" (.Contains()) บ่อยมาก**  
  * **การใช้ List\<string\> จะทำให้ Big-O คอ O(n) ซึ่งช้าลงเรื่อยๆ ตามจำนวนกล่องที่เปิด แต่การใช้ HashSet เป็นโครงสร้างแบบ Hash Table รับประกันความเร็วที่คงที่ระดับ O(1) ป้องกันไอเทมซ้ำซ้อนโดยธรรมชาติ (Unique constraints)**

### 3.2 List\<T\> (คอลเลกชันแบบ Dynamic Array)

* **การใช้งาน: activeTraits, runsInScenes และใช้รองรับการ Serialize ลง JSON**  
* **เหตุผลทางการออกแบบ: เหมาะกับข้อมูลเรื่องของลำดับ ไอเทมในช่องเก็บของ (InventoryData.itemNames / itemCounts) ที่ลำดับใน Index สอดคล้องกับตำแหน่งหน้าต่าง UI**

### 3.3 Data Class Models (Serialization)

* **การใช้งาน: สร้างโครงสร้างคลาสจำลองเพื่อห่อหุ้มข้อมูลก่อนนำไปเซฟ (เช่น คลาส**   
* **SaveData,**   
* **PlayerStatsData,**   
* **InventoryData) โดยกำกับด้วย Attribute \[Serializable\]**  
* **เหตุผลทางการออกแบบ: Unity อบเจกต์ (เช่น**   
* **PlayerStats ที่สืบทอด MonoBehaviour) ไม่สามารถเซฟทับลงไฟล์ JSON โดยตรงได้ จึงต้องสร้าง Data Class ร่างจำลองมาถ่ายโอนค่า (Data Transfer Object) ก่อนแปลงเป็น String JSON ด้วย JsonUtility**

---

## **4\. กลไกเชิงเทคนิคในการแก้ปัญหาเฉพาะทาง (Technical Mechanisms)**

### 4.1 Asynchronous Execution ด้วย Coroutines

* **ปัญหา: เมื่อสั่ง LoadScene คลาสทุกคลาสใน Scene ใหม่จะถูกกระชากขึ้นมา ซึ่ง Object.Start() บางตัวอาจจะทำงานก่อนหลังไม่เท่ากัน ทำให้การเซ็ตค่าทับเกิด Race Condition**  
* **การแก้ไข: โค้ดมีการเรียก StartCoroutine(ApplyPendingDataNextFrame()) และ yield return null; เพื่อบังคับให้เธรดเกม รอผ่านไป 1 เฟรม เพื่อให้องค์ประกอบลอจิกทุกอย่างใน State ใหม่สร้างตัวเองเสร็จหมดก่อน ค่อยทำการสาด Data จาก Save ทับเข้าไป ได้ผลลัพธ์ที่นิ่งและมั่นคง ไม่ขัดจังหวะ Frame rate**

### 4.2 Error Fallback Handling (Try/Catch)

* **การแก้ไข: ในลูปการ Load มีการครอบกระบวนการย่อยด้วย try-catch บล็อก (เช่น โหลดสเตตัส, โหลดคลังเก็บของ, รีเซ็ตตาย) การทำแบบนี้คือการทำให้มั่นใจว่า ถึงแม้จะมีระบบใดระบบหนึ่งอัปเดตใหม่แล้ว Data Model พังพินาศ มันก็จะไม่ขัดขวางให้ระบบส่วนที่เหลืออีก 3 อันหยุดทำางานไปด้วย เกมจะไม่จอค้างจากการโหลดล้มเหลวเพียง 1 จุด**

# **โครงสร้างระบบตัวละคร (Character Systems Architecture)**

**รายงานฉบับนี้อธิบายถึงโครงสร้างเชิงสถาปัตยกรรม (Architecture Design) และหลักการวิศวกรรมซอฟต์แวร์ (Software Engineering) ที่ใช้ในการสร้าง ตัวละครผู้เล่น (Player), ศัตรูทั่วไป (Enemy) และ บอส (Boss) ในเกมนี้**  
**การออกแบบทั้งหมดอิงหลักการ OOP (Object-Oriented Programming) อย่างเต็มรูปแบบ รวมไปถึงการใช้ Design Patterns เพื่อให้ระบบยืดหยุ่น ลดการผูกมัดโค้ด (Decoupling) และง่ายต่อการปรับขนาด (Scalability)**

---

## **1\. แกนหลักของทุกตัวละคร (Core Architecture)**

**ระบบตัวละครของเกมนี้ไม่ได้ยัดลอจิกทั้งหมดไว้ในคลาสเดียว แต่ใช้หลักการ Composition over Inheritance ผสมกับ Component-Based Architecture**  
**แทนที่จะมีคลาส Character ตัวใหญ่ตัวเดียว โค้ดถูกแบ่งเป็นชิ้นส่วนย่อย (Components) ที่ทำหน้าที่เดี่ยวๆ ตามหลัก Single Responsibility Principle (SRP) อาทิ:**

* **Control/AI: สำหรับคุมการเคลื่อนไหว พฤติกรรม**  
* **Health: สำหรับจดจำพลังชีวิตและระบบรับดาเมจ**  
* **Stats: สำหรับคำนวณค่าพลัง (STR, AGI, ฯลฯ)**  
* **Attack: สำหรับจัดการการตีและการชน (Hitbox)**

---

## **2\. ตัวละครผู้เล่น (Player System)**

### 2.1 Inheritance & Polymorphism (การสืบทอดและการพ้องรูป)

* PlayerHealth สืบทอดมาจาก **CharacterHealth: เกมมีการสร้าง Base Class CharacterHealth เอาไว้ให้ทุกตัวละครในเกมใช้งานร่วมกัน (มีฟังก์ชันลดเลือด เลือดตาย) แต่**   
* **PlayerHealth ใช้หลักการ Polymorphism (ผ่านคีย์เวิร์ด override) เพื่อเขียนทับฟังก์ชัน**   
* **TakeDamage() และ**   
* **Die()**  
  * ***เหตุผล:*** **เพื่อแทรกระบบ Parry (ปัดป้อง) และ Block (ป้องกัน) เข้าไปเฉพาะตัวละครผู้เล่นเท่านั้น โดยไม่ต้องไปกวนระบบรับดาเมจของศัตรู**

### 2.2 Coroutines สำหรับ Time-based Actions

* **ใน**   
* **PlayerControl การทำสกิล Dash (พุ่ง) ถูกเขียนบน IEnumerator ควบคู่กับการประยุกต์ใช้ Animation Events (**  
* **Anim\_DashIFrameStart())**  
  * ***เหตุผล:*** **การใช้ Coroutine ทำให้สามารถสั่งแคปความเร็วคงที่เป็นระยะเวลาเล็กน้อย (เช่น 0.15 วิ) ได้โดยไม่ต้องใช้ตัวแปร Timer มารกในโค้ด**   
  * **Update()**

---

## **3\. ศัตรูทั่วไป (Enemy AI System)**

**ศัตรูใช้เทคนิคที่ซับซ้อนขึ้นเพื่อให้มันคิดและตัดสินใจได้**

### 3.1 State Pattern (Finite State Machine)

**ปัญหาของการเขียน AI ศัตรูคือพฤติกรรมมันเยอะ (ยืน, เดินลาดตระเวน, ไล่ล่า, โจมตี, ตาย) หากใช้ if-else เช็คสถานะ โค้ดใน Update จะยาวเป็นพันบรรทัด เกมนี้แก้ปัญหาโดยใช้ State Pattern.**

* **โครงสร้างคลาส: สร้าง Abstract class ชื่อ**   
* **EnemyState (มีฟังก์ชัน Enter, Update, Exit) แล้วสร้างซับคลาสแยกเฉพาะ เช่น EnemyIdleState, EnemyChaseState, EnemyAttackState**  
* **การทำงานใน**   
* **EnemyAI: เมื่อศัตรูเจอผู้เล่น มันจะเรียกคำสั่ง**   
* **ChangeState(new EnemyChaseState(this))**  
* ***ข้อดีทางหลักการ (Open/Closed Principle)*****: หากในอนาคตต้องการเพิ่มแมงมุมที่กระโดดได้ ก็แค่เขียนคลาส EnemyJumpState เพิ่ม โดยไม่ต้องไปกดแก้โค้ดในตัว**   
* **EnemyAI สักตัวอักษรเดียว**

### 3.2 Event-Driven Design (Observer Pattern)

* **คลาส**   
* **EnemyAI ทำการสมัครสมาชิก (Subscribe) ไปที่ Hitbox.OnHit (C\# Delegate)**  
  * ***เหตุผล:*** **เมื่อกล่อง Hitbox ฟาดโดนผู้เล่น Hitbox จะตะโกนบอกทุกโค้ดที่แอบฟังอยู่ว่า "ชั้นตีโดนแล้วนะ\!" ตัว EnemyAI ก็จะรับรู้แล้วสั่งทำ HitStop (ฟรีซเฟรม) ทันที โค้ด Hitbox จึงไม่ต้องรู้จักว่าใครถือมันอยู่**

---

## **4\. บอส (Boss System)**

**บอสเกมนี้ (ผ่าน** 

**BossAI.cs) คือวิวัฒนาการขั้นสูงสุดของศัตรูทั่วไป โดยยกทัพ Patterns มาใช้อย่างครบครันเพื่อให้ดูอลังการ**

### 4.1 State Pattern ขั้นกว่า

**บอสมี FSM (Finite State Machine) ของตัวเองชื่อ** 

**BossState (คล้าย EnemyState แต่ซับซ้อนกว่าเรื่องการคุม Action). จุดเด่นคือ State ของการป้องกัน (BossDodgeState, BossBlockState) เมื่อบอสโดนตีสวน มันสามารถหักเหขัดจังหวะ State ปัจจุบันเปลี่ยนแผนไปหลบได้ทันที**

### 4.2 Strategy Pattern (ระบบ Skills)

**บอสแต่ละตัวมีสกิลไม่เหมือนกันและรูปแบบเปลี่ยนตามเฟสเลือด การนำสคริปต์สกิลไปยัดใน BossAI คือหายนะ โค้ดจึงใช้ Strategy Pattern.**

* **กลไก: สร้างคลาสจำพวก BossSkill (เช่น โจมตีประชิด, พ่นลูกโป่ง, ยิง Homing) แล้วแยกเป็นคลาสเดี่ยว**  
* **ใน**   
* **BossAI มีระบบ Dictionary (Dictionary\<BossSkill, float\>) เอาไว้ใช้จดจำสูตรสกิลเหล่านั้นพร้อมจับเวลา Cooldown เฉพาะสกิล**  
* **เวลาบอสจะตี มันจะตัดสินใจเลือกสูตรสกิลที่มี โยนเข้าฟังก์ชัน skill.Execute(this) ทำให้สามารถเอาสูตรตีท่าแปลกๆ ไปแปะบอสตัวไหนก็ได้**

### 4.3 Data-Driven Design (ระบบ Phases)

* **ตัวแปร BossPhaseData\[\] phases เปิดกว้างให้ดีไซเนอร์เข้าไปจูนใน Unity Inspector ได้ว่า:**  
  * **Phase 1: เลือด 100-50% / เดินช้า / ใช้สคริปต์สกิล ทุบ**  
  * **Phase 2: เลือด 50-0% / เดินเร็ว 1.5 เท่า / ใช้สคริปต์สกิล ยิงเวทย์**  
* **ทำให้โปรแกรมเมอร์ไม่ต้อง Hardcode ตัวเลข 50% หรือเงื่อนไขในโค้ด ปล่อยให้ระบบมันเปลี่ยนร่าง (Transition) เติมเกราะอมตะ (Hyper Armor) ตาม Data ที่ไหลเข้ามาเลย**

# **โครงสร้างระบบช่องเก็บของและไอเทม (Inventory & Item Systems Architecture)**

**รายงานฉบับนี้อธิบายถึงโครงสร้างและแพทเทิร์นเชิงเทคนิคที่ใช้ในการพัฒนาระบบคลังเก็บของ (Inventory), ระบบอาวุธชุดเกราะ (Equipment), และระบบไอเทม (Items) โดยเน้นไปที่สถาปัตยกรรม Data-Driven และ UI Event Handling**

---

## **1\. โครงสร้างข้อมูลไอเทม (Data-Driven Design)**

**ระบบไอเทมในเกมนี้ไม่ได้ถูกเขียนฝัง (Hardcode) ไว้ในสคริปต์ แต่ใช้สถาปัตยกรรม Data-Driven Design ผ่านฟีเจอร์ ScriptableObject ของ Unity**

### 1.1 คลาสฐาน (Base Class: 

### Item)

* **การประยุกต์ใช้: สร้างไฟล์คลาส Item : ScriptableObject ซึ่งเก็บเฉพาะ "ข้อมูลดิบ" เช่น ชื่อ (itemName), รูปภาพ (image), ประเภทย่อย (type), และตั้งค่าการซ้อนทับ (stackable)**  
* **ข้อดี:**  
  * **ประหยัดหน่วยความจำ: ไม่ว่าผู้เล่นจะมีขวดเลือด (Potion) 100 ขวดในกระเป๋า ข้อมูลภาพพื้นฐานจะถูกจดจำไว้ใน RAM เพียง 1 ก้อน (Flyweight Pattern Concept)**  
  * **เอื้อต่อ Game Designer: สามารถคลิกขวา Create \> Scriptable object/Item ใน Unity เพื่อสร้างไอเทมใหม่ได้ทันทีโดยไม่ต้องเขียนโค้ด**

### 1.2 การสืบทอดไอเทม (Inheritance)

**คลาส** 

**Item ถูกออกแบบไว้สำหรับการสืบทอด (Inheritance) สำหรับไอเทมที่มีฟังก์ชันเฉพาะ**

* **EquipmentItem : Item: สิ่งที่เพิ่มเข้ามาคือชุดตัวเลขค่าสถานะ StatBonus และรูปภาพตอนใส่สวม equippedSprite**  
* **UseableItem : Item: (พบผ่านการอ้างอิงใน**   
* **InventoryManager) คือไอเทมที่กดใช้ได้ มีฟังก์ชัน**   
* **Use(player) แบบ Polymorphism (พ้องรูป) เพื่อลดเลือด เพิ่มเลือด ต่างกันไปในแต่ละคลาสลูก**

---

## **2\. โครงสร้างการจัดการช่องเก็บของ (InventoryManager & EquipmentManager)**

**หัวใจของระบบคลังคือ** 

**InventoryManager และ** 

**EquipmentManager ที่ทำงานควบคู่กันด้วยแพทเทิร์น Singleton**

### 2.1 ระบบจัดเรียง Data (Data Structure)

* **กระเป๋าและ Hotbar: การเก็บของใช้ Array ในการจำลองช่องเก็บของ (InventorySlot\[\] inventorySlots, UseableItemSlot\[\] hotbarSlot)**  
* **Hotbar Indexing (validHotbarIndices): แทนที่จะวนลูปหาของใน Hotbar ถัดไปโง่ๆ ทุกครั้งที่กดเลื่อนเมาส์ เกมนี้เลือกใช้ List\<int\> เก็บอ้างอิงของ "ช่องที่ไม่ได้ว่างเปล่า" ไว้ล่วงหน้า (Caching) ทำให้ฟังก์ชันเลื่อนเมาส์ดึงค่าใช้งานได้รวดเร็วระดับ O(1)**

### 2.2 โค้ดที่สะอาดและครอบคลุมความผิดพลาด (Defensive Programming)

* **การ Add ไอเทม ถูกออกแบบค่อนข้างรัดกุม**  
  * **สเต็ป 1: ถ้ายัดไอเทมเข้ามา เช็คก่อนว่ามันซ้อนกันได้ไหม (Stackable)**  
  * **สเต็ป 2: ถ้าซ้อนได้ ให้วิ่งไปหาช่องที่มีของแบบเดียวกันอยู่แล้ว เติมให้เต็มหลอด (stackCapacoty \= 50)**  
  * **สเต็ป 3: ถ้าล้นหลอด ค่อยไปสร้างกองใหม่ การออกแบบตรรกะแบบนี้ทำให้กระเป๋าผู้เล่นเป็นระเบียบ ไม่เกิดบั๊กไอเทมกองเดียวกันกระจัดกระจาย**

---

## **3\. UI และการโต้ตอบด้วยเมาส์ (Event-Driven UI)**

**การลากวางไอเทม (Drag & Drop) ในกระเป๋าใช้ระบบพี่น้องของ Observer Pattern คือ Events และ Interfaces**

### 3.1 การสืบทอดบนช่องสวมใส่ (Polymorphism on Slots)

* **มีคลาสแม่ชื่อ**   
* **InventorySlot รองรับคำสั่งเวลาปล่อยเมาส์ลาก**   
* **OnDrop(PointerEventData)**  
* **เมื่อเราสร้างช่องใส่ชุดเกราะ เราไม่สร้างใหม่หมด แต่เขียน EquipmentSlot : InventorySlot**  
  * **การทำเช่นนี้ทำให้**   
  * **EquipmentSlot เขียนทับฟังก์ชันเช็คเงื่อนไข**   
  * **CanPlaceItem() แทรกลอจิกเข้ามาว่า *"รับเฉพาะ***   
  * ***EquipmentItem เท่านั้นนะ ไอเทมอื่นห้ามวาง\!"***

### 3.2 ระบบ Observer (UI Events)

* **การอัปเดตค่าพลังหรือเปลี่ยนภาพกระเป๋ามีการใช้ InventoryEvents.OnItemMoved และ InventoryEvents.OnBeginDragItem ซึ่งเป็นระดับ Static Delegate**  
* **ทันทีที่คุณคลิกลากดาบ ตัว**   
* **EquipmentSlot ที่แอบฟัง OnBeginDragItem อยู่ ก็จะเช็กเงื่อนไขดาบ แล้วสั่งเปลี่ยนสีช่องของตัวเองให้ "เป็นสีเขียว" (วางได้) หรือ "สีแดง" (วางไม่ได้) เพื่อนำทางสายตาผู้เล่นโดยอัตโนมัติ**  
* 

## **4\. ระบบสวมใส่อุปกรณ์และคำนวณสเตตัส (Equipment & Stats System)**

**ระบบอุปกรณ์และการคำนวณค่าสถานะ (Stats) ออกแบบโดยเน้นหลักการ Operator Overloading และการคำนวณแบบ Event-Driven Recalculation**

### 4.1 ข้อมูลแบบ Struct & Operator Overloading

* **ค่าสถานะ (HP, ATK, DEF, ฯลฯ) ถูกมัดรวมกันเป็น struct StatBonus แทนที่จะเป็น Class เพื่อลดภาระของ Garbage Collector (GC)**  
* **Operator Overloading: สคริปต์มีการเขียนทับเครื่องหมายบวก ( public static StatBonus operator \+(StatBonus a, StatBonus b) ) ทำให้เวลาผู้เล่นใส่ชุดเกราะ 5 ชิ้น โปรแกรมเมอร์แค่เขียน total \+= equip.GetBonus(); สเตตัสทุกสายก็จะบวกต่อกันเป็นทอดๆ อย่างสวยงาม โค้ดอ่านง่ายและลดโอกาสบวกเลขผิดพลาด**

### 4.2 โครงสร้างค่าพลังผู้เล่น (Inheritance on Stats)

* **CharacterStats: เป็น Base Class หน้าที่แค่เก็บค่าพลังดิบ (BaseHP, BaseATK) ที่ทั้งมอนสเตอร์และผู้เล่นมีเหมือนกัน**  
* **PlayerStats : CharacterStats: ฝั่งผู้เล่นจับไปสืบทอด แล้วตีบวกฟังก์ชันเฉพาะตัวเพิ่มเข้าไป เช่น BaseMana, heatGuage (ระบบเกจสกิล) โดยมีฟังก์ชัน**   
* **RecalculateStats() ที่จะประกบค่า Base เข้ากับ armorBonus**

### 4.3 ควบคุมการคำนวณใหม่ (Event-Driven Recalculation)

* **EquipmentManager จะไม่คำนวณสถานะพร่ำเพรื่อใน**   
* **Update() (ซึ่งเปลืองทรัพยากร)**  
* **โค้ดถูกเขียนให้ดักรอ InventoryEvents.OnItemMoved แทน เมื่อผู้เล่นลากชุดเกราะใส่ช่อง ระบบ Event จะตะโกนบอก**   
* **EquipmentManager ให้รันลูปรวมแต้ม**   
* **StatBonus จากชุดเกราะทุกชิ้น ส่งให้**   
* **PlayerStats และไปเคาะประตูสั่งให้ UI อัปเดตตัวเลขบนหน้าจอ เป็นอันเสร็จสิ้น**

