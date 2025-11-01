# บันทึกผลการทดลอง
## Lab 1: Binary Semaphores
### 📊 การสังเกตและบันทึกผล
| ทดลอง | Events Sent | Events Received | Timer Events | Button Presses | Efficiency |
|-------|-------------|-----------------|--------------|----------------|------------|
| 1 (Normal) | 3| 3| 1| 0| 100%|
| 2 (Multiple Give) | 5| 5| 1| 0| 100%|
| 3 (Short Timeout) | 8| 8| 2| 0| 100%|


### คำถามสำหรับการทดลง
1. เมื่อ give semaphore หลายครั้งติดต่อกัน จะเกิดอะไรขึ้น?
- เนื่องจาก Binary Semaphore เป็นเพียงกลไก Signal ที่มีสถานะเพียง มี/ไม่มี (0 หรือ 1)
- การเรียก xSemaphoreGive() ซ้ำ ๆ โดยที่ Semaphore มีค่าเป็น 1 อยู่แล้ว จะ ไม่มีผล ทำให้ค่าสะสมเพิ่มขึ้น
- สัญญาณที่ส่งมาซ้ำซ้อนจะ ถูกละเลย โดยมีผลเท่ากับการส่งสัญญาณเพียงครั้งเดียวเท่านั้น

2. ISR สามารถใช้ `xSemaphoreGive` หรือต้องใช้ `xSemaphoreGiveFromISR`?
- ต้องใช้ xSemaphoreGiveFromISR() เท่านั้น
- ฟังก์ชันที่ลงท้ายด้วย FromISR ถูกออกแบบมาให้ทำงานใน Context ของ Interrupt Service Routine (ISR) ได้อย่าง ปลอดภัย
- ฟังก์ชันนี้จะจัดการการปลุก Task ที่กำลังรอ Semaphore อย่างถูกต้องโดยไม่รบกวนการทำงานของ Scheduler หรือทำให้เกิดปัญหาระบบค้าง

3. Binary Semaphore แตกต่างจาก Queue อย่างไร?
- Binary Semaphore: ใช้สำหรับ การซิงโครไนซ์เหตุการณ์ (Event Synchronization) หรือการแจ้งเตือนว่า "มีบางอย่างเกิดขึ้น" โดย ไม่มีการส่งผ่านข้อมูล
- Queue: ใช้สำหรับ การสื่อสารข้อมูล (Data Passing) ซึ่งมีบัฟเฟอร์ที่สามารถเก็บข้อมูลได้หลายรายการระหว่าง Task หรือระหว่าง ISR กับ Task

## Lab 2: Mutex and Critical Sections
### 📊 การสังเกตและบันทึกผล
| ทดลอง                | Successful | Failed | Corrupted | Success Rate            | สังเกต                                                                                         |
| -------------------- | ---------- | ------ | --------- | ----------------------- | ---------------------------------------------------------------------------------------------- |
| 1 (With Mutex)       | 16         | 0      | 0         | 100%                    | ระบบปกติ ไม่มี data corruption; Mutex ป้องกันได้สมบูรณ์                                        |
| 2 (No Mutex)         | 28         | 0      | 0         | 100%                    | ยังไม่เกิด race condition ในช่วงสั้น แต่ไม่มีการป้องกันเลย                                     |
| 3 (Changed Priority) | 42         | 0      | 3         | 100% (แต่มี corruption) | เกิด **data corruption** จาก race condition เพราะ low-priority task แย่ง access ก่อน task อื่น |


### คำถามสำหรับการทดลอง
1. เมื่อไม่ใช้ Mutex จะเกิด data corruption หรือไม่?
- มีโอกาสสูงที่จะเกิด Data Corruption
- เมื่อหลาย Task เข้าถึงและแก้ไขตัวแปรที่ใช้ร่วมกัน (Shared Resource) ในเวลาเดียวกันโดยไม่มี Mutex ป้องกัน จะเกิด Race Condition
- ข้อมูลที่ถูกแก้ไขโดย Task หนึ่งอาจถูก เขียนทับ ด้วย Task อื่น ทำให้ค่าสุดท้ายไม่ถูกต้อง หรือเกิดความเสียหายของข้อมูล (Inconsistent State)

2. Priority Inheritance ทำงานอย่างไร?
- Priority Inheritance เป็นกลไกป้องกัน Priority Inversion
- หาก Task ลำดับต่ำกำลังถือ Mutex อยู่ และ Task ลำดับสูงต้องการ Mutex นั้น ระบบ FreeRTOS จะ ยกระดับ Priority ของ Task ลำดับต่ำให้ เท่ากับ Task ลำดับสูงที่กำลังรอ
- เมื่อ Task ลำดับต่ำทำงานเสร็จและ ปล่อย Mutex แล้ว Priority ของ Task นั้นจะ กลับคืนสู่ค่าเดิม

3. Task priority มีผลต่อการเข้าถึง shared resource อย่างไร?
- โดยทั่วไป Task ที่มี Priority สูง จะมีโอกาสได้ เข้าถึง Critical Section เร็วขึ้น หาก Mutex ว่าง
- อย่างไรก็ตาม หากใช้ Mutex อย่างถูกต้อง การเข้าถึงจะถูกควบคุมโดย กลไกการล็อก เพื่อป้องกันการเข้าถึงพร้อมกัน (Mutual Exclusion)
- หากไม่มี Mutex การตั้ง Priority ที่ไม่เหมาะสมอาจทำให้ Task ลำดับต่ำ แทบไม่ได้เข้าถึง ทรัพยากรเลย (Starvation)

## Lab 3: Counting Semaphores 
### 📊 การสังเกตและบันทึกผล
| ทดลอง          | Resources | Producers |            Success Rate (%) | Avg Wait (ms) | Resource Utilization (%) | หมายเหตุ                                       |
| :------------- | --------: | --------: | --------------------------: | ------------: | -----------------------: | :--------------------------------------------- |
| **1 (3R, 5P)** |         3 |         5 |                   **100 %** |        ≈ 1690 |                   ≈ 95 % | ทรัพยากรถูกใช้เต็มเกือบตลอด ไม่มี timeout      |
| **2 (5R, 5P)** |         5 |         5 |                   **100 %** |         ≈ 400 |                ≈ 60–70 % | มีทรัพยากรว่างเกือบตลอด ใช้ไม่เต็ม pool        |
| **3 (3R, 8P)** |         3 |         8 | **≈ 100 % (บางช่วง delay)** |        ≈ 2700 |                  ≈ 100 % | มีการรอคิวนานขึ้น / ใช้ทรัพยากรเต็มแทบตลอดเวลา |


### คำถามสำหรับการทดลอง
1. เมื่อ Producers มากกว่า Resources จะเกิดอะไรขึ้น?
- จะเกิด การแย่งชิงทรัพยากร (Resource Contention)
- Producers ที่ร้องขอทรัพยากร (เรียก xSemaphoreTake()) ในขณะที่ Semaphore มีค่าเป็นศูนย์ (ทรัพยากรถูกใช้หมด) จะถูก บล็อก (Blocked)
- Task เหล่านั้นจะเข้าสู่สถานะรอจนกว่า Producer ตัวอื่นจะ คืนทรัพยากร (xSemaphoreGive()) เข้าสู่ Pool หรือจนกว่าจะเกิด Timeout

2. Load Generator มีผลต่อ Success Rate อย่างไร?
- Load Generator ถูกใช้เพื่อจำลอง ภาระงานสูงสุด (Burst Load) ในช่วงเวลาสั้น ๆ
- เมื่อเกิด Burst Load ขึ้น จำนวนการร้องขอจะ สูงเกินกว่า จำนวน Resources ที่มีชั่วคราว ทำให้ Resource Pool หมดเกลี้ยง
- ผลคือ Task บางตัวที่ร้องขอในช่วงเวลานั้นจะไม่ได้รับ Resource และอาจเกิด Timeout ทำให้ Success Rate ลดลงชั่วคราว

3. Counting Semaphore จัดการ Resource Pool อย่างไร?
- Counting Semaphore ทำหน้าที่เป็น ตัวนับ (Counter) สำหรับควบคุมการเข้าถึงชุดทรัพยากรที่มีจำนวนจำกัด
- xSemaphoreCreateCounting(max, initial): กำหนดขนาดสูงสุดของ Pool
- xSemaphoreTake(): ลดค่า Counter (จอง Resource) และทำให้ Task Blocked หาก Counter เป็นศูนย์
- xSemaphoreGive(): เพิ่มค่า Counter (คืน Resource)
- uxSemaphoreGetCount(): ใช้ตรวจสอบจำนวนทรัพยากรที่ยังเหลืออยู่ในปัจจุบันภายในระบบ
- กลไกนี้ช่วยให้มั่นใจได้ว่า จำนวน Task ที่เข้าถึง Resource จะไม่เกิน จำนวนทรัพยากรที่มีอยู่จริงในระบบ (Controlled Sharing)
