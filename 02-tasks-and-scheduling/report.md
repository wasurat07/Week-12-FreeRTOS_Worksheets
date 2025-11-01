# บันทึกผลการทดลอง
## Lab 1: Task Priority และ Scheduling
### คำถามสำหรับวิเคราะห์
1. Priority ไหนทำงานมากที่สุด? เพราะอะไร?
- Task ที่มีลำดับความสำคัญสูงสุด (Priority 5) ได้รับการประมวลผลมากที่สุด
- เพราะ FreeRTOS ใช้ Priority-based Preemptive Scheduling ซึ่งหมายความว่า Task ที่มี Priority สูง จะได้รับ CPU ก่อนเสมอ และสามารถ ขัดจังหวะ (preempt) การทำงานของ Task ที่มี Priority ต่ำกว่าได้ทันทีเมื่อพร้อมรัน

2. เกิด Priority Inversion หรือไม่? จะแก้ไขได้อย่างไร?
- อาจเกิด Priority Inversion ได้ เมื่อ Task priority ต่ำครอบครอง resource (เช่น mutex) แล้ว task priority สูงต้องการใช้ resource เดียวกัน แต่ถูก block รอขณะที่ task medium priority แทรกเข้ามาทำงาน ทำให้ task high ต้องรอนาน
- วิธีแก้ไข:ใช้ Priority Inheritance Mechanism — เมื่อ task priority ต่ำถือ mutex อยู่ แล้ว task priority สูงรอใช้ ระบบจะ “ยก priority” ของ task ต่ำชั่วคราวให้เท่ากับของ task สูง เพื่อให้มันคืน resource ได้เร็วขึ้น

3. Tasks ที่มี priority เดียวกันทำงานอย่างไร?
- Task ที่มี Priority เท่ากันจะถูกจัดการด้วยกลไก Round-Robin Scheduling
- Scheduler จะจัดสรร Time Slice (ตาม Tick) ให้แต่ละ Task ได้รับ CPU สลับกันไปตามลำดับ อย่างยุติธรรม

4. การเปลี่ยน Priority แบบ dynamic ส่งผลอย่างไร?
- การปรับเปลี่ยน Priority ขณะ Task กำลังทำงาน (runtime) เป็นการ ปรับเปลี่ยนลำดับการเข้าถึง CPU ชั่วคราว
- สิ่งนี้ช่วยให้ Task ที่เดิมมี Priority ต่ำสามารถถูก "ส่งเสริม" (boost) ให้รันได้ก่อนในช่วงเวลาวิกฤต ซึ่งเป็นประโยชน์ในการจัดการงานที่มีความสำคัญเร่งด่วนเฉพาะกิจ

5. CPU utilization ของแต่ละ priority เป็นอย่างไร?
- การใช้ CPU จะ แปรผันตรง กับลำดับความสำคัญ
- High Priority: มี utilization สูงที่สุด เพราะได้ CPU ก่อน และถูก preempt น้อยมาก
- Low Priority: มี utilization ต่ำที่สุด เพราะต้องรอให้ Task Priority สูงกว่าอยู่ในสถานะ Blocked หรือเสร็จสิ้นก่อนจึงจะมีโอกาสรัน

## Lab 2: Task States Demonstration
### คำถามสำหรับวิเคราะห์
1. Task อยู่ใน Running state เมื่อไหร่บ้าง?
- Task จะอยู่ใน Running state ก็ต่อเมื่อ Scheduler ได้เลือกให้ Task นั้นทำงาน และกำลังประมวลผลคำสั่งจริง ๆ บน CPU
- Task จะอยู่ในสถานะนี้ตราบเท่าที่ไม่มี Task Priority สูงกว่ามาแย่ง หรือจนกว่า Task นั้นจะเรียกฟังก์ชันที่ทำให้เข้าสู่สถานะ Blocked

2. ความแตกต่างระหว่าง Ready และ Blocked state คืออะไร?
- Ready State: Task มี ความพร้อมสมบูรณ์ ที่จะรัน แต่กำลังรอ คิว เพื่อให้ Scheduler จัดสรร CPU ให้ (เนื่องจากมี Task Priority เท่ากันหรือสูงกว่ากำลังรันอยู่)
- Blocked State: Task หยุดการทำงาน ชั่วคราว เนื่องจากกำลัง รอ Event ภายนอก (เช่น การหน่วงเวลาด้วย vTaskDelay(), การรอข้อมูลจากคิว, หรือการรอ Semaphore)

3. การใช้ vTaskDelay() ทำให้ task อยู่ใน state ใด?
- เมื่อ Task เรียกใช้ vTaskDelay() Task นั้นจะเปลี่ยนสถานะเป็น Blocked state
- Scheduler จะนำ Task ออกจากคิวที่พร้อมรัน และจะนำกลับมายัง Ready state อีกครั้งเมื่อเวลาหน่วงที่กำหนดโดย FreeRTOS Tick Interrupt ได้หมดลง

4. การ Suspend task ต่างจาก Block อย่างไร?
- Suspended: เป็นการ หยุดการทำงานโดยเจตนา (Manual) โดยใช้ API เช่น vTaskSuspend() Task จะ ไม่กลับมาทำงาน เองแม้จะเกิด Event หรือหมดเวลารอ
- Blocked: เป็นการหยุดการทำงานแบบ มีเงื่อนไข (Conditional) Task จะกลับมาที่ Ready state โดย อัตโนมัติ เมื่อเงื่อนไขการรอ (เช่น เวลาหมด, ได้รับ Semaphore) เป็นจริง

5. Task ที่ถูก Delete จะกลับมาได้หรือไม่?
- ไม่สามารถกลับมาได้ เมื่อ Task ถูกลบด้วย vTaskDelete() ระบบจะ คืนหน่วยความจำ (Stack และ TCB) ที่จัดสรรไว้ให้กับ Task นั้น
- หากต้องการให้ Task เดิมกลับมาทำงาน จะต้อง สร้างขึ้นมาใหม่ ด้วยการเรียกฟังก์ชัน xTaskCreate() อีกครั้งเท่านั้น

## Lab 3: Stack Monitoring และ Debugging
### คำถามสำหรับวิเคราะห์
1. Task ไหนใช้ stack มากที่สุด? เพราะอะไร?
- Task ที่ใช้ stack มากที่สุดคือ Heavy Task
- เพราะภายใน task มีการประกาศ local arrays ขนาดใหญ่ เช่น char large_buffer[1024], int large_numbers[200], และ char another_buffer[512]
- ซึ่งทั้งหมดถูกเก็บใน stack memory ของ task นั้นโดยตรง จึงทำให้เกิดการใช้ stack สูงและมีความเสี่ยงต่อ stack overflow มากที่สุด

2. การใช้ heap แทน stack มีข้อดีอย่างไร?
- ลดความเสี่ยง Stack Overflow: ข้อมูลขนาดใหญ่ไม่ไปกินพื้นที่ Stack ที่จำกัด
- การจัดการหน่วยความจำแบบพลวัต (Dynamic Allocation): สามารถจัดสรรหน่วยความจำที่มีขนาดไม่แน่นอนหรือปรับเปลี่ยนระหว่าง Runtime ได้อย่างยืดหยุ่นด้วย malloc() หรือ pvPortMalloc()
- ประหยัด Stack: ทำให้สามารถกำหนดขนาด Stack ของ Task ได้เล็กลง

3. Stack overflow เกิดขึ้นเมื่อไหร่และทำอย่างไรป้องกัน?
- เกิดขึ้นเมื่อ: Task พยายามใช้หน่วยความจำ Stack เกินกว่าขนาดที่กำหนดไว้ตอนสร้าง Task (เช่น จากการใช้ Local Variable ขนาดใหญ่, การเรียกฟังก์ชัน Recursive ที่ลึกเกินไป)
- วิธีป้องกัน:
  1. กำหนด stack size ให้เหมาะสมตามลักษณะของ task
  2. ใช้ uxTaskGetStackHighWaterMark() เพื่อตรวจสอบปริมาณ stack ที่เหลือ
  3. เปิดใช้งาน CONFIG_FREERTOS_CHECK_STACKOVERFLOW=2 เพื่อให้ระบบตรวจจับอัตโนมัติ
  4. ใช้ heap สำหรับข้อมูลขนาดใหญ่
  5. หลีกเลี่ยง recursion ที่ไม่จำเป็น

4. การตั้งค่า stack size ควรพิจารณาจากอะไร?
- จำนวนและขนาดของ local variables ที่ใช้ใน task
- ความลึกสูงสุด ของการเรียกฟังก์ชัน (function call depth)
- ความซับซ้อนของโค้ด เช่น recursion, snprintf, sprintf, log ที่ใช้ buffer
- ควรเพิ่ม Margin เผื่อไว้ ($30-50\%$) จากค่าการใช้งานสูงสุดที่วัดได้จริงด้วยเครื่องมือมอนิเตอร์
- ใช้ข้อมูลจริงจาก uxTaskGetStackHighWaterMark() เพื่อปรับปรุงภายหลัง

5. Recursion ส่งผลต่อ stack usage อย่างไร?
- Recursion (การเรียกตัวเองของฟังก์ชัน) เพิ่ม Stack Usage อย่างรวดเร็ว ตามความลึกของการเรียกซ้ำ
- ในแต่ละรอบของการเรียกซ้ำ ระบบต้อง จัดเก็บตัวแปรท้องถิ่น และ ที่อยู่ส่งกลับ (Return Address) ไว้ใน Stack ซึ่งทำให้เกิดการใช้ Stack ในลักษณะสะสม และมีโอกาสเกิด Stack Overflow สูงหากความลึกไม่จำกัด
