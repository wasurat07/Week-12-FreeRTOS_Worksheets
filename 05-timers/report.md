# บันทึกผลการทดลอง
## Lab 2: Timer Applications - Watchdog & LED Patterns
### 📋 Post-Lab Questions
1. **Watchdog Design**: เหตุใดต้องใช้ separate timer สำหรับ feeding watchdog?
- การใช้ Timer ที่แยกต่างหาก สำหรับการ "ให้อาหาร" Watchdog (Feed Timer) มีวัตถุประสงค์เพื่อรักษา ความเป็นอิสระในการตรวจสอบ
- Watchdog Timer จะทำหน้าที่เพียง นับถอยหลัง เพื่อหาความผิดปกติ
- ถ้า Feed Timer ถูกรันตามปกติ แสดงว่า Scheduler และ Task หลัก ยังคงทำงานอยู่และสามารถเรียกฟังก์ชัน Feed ได้ ซึ่งเป็นการยืนยันความมีชีวิต
- หากระบบ เกิดการหยุดชะงัก (Hang) Feed Timer จะถูกหยุดไปด้วย ทำให้ Watchdog ไม่ได้รับการรีเซ็ต (reset) และเกิด Timeout ในที่สุด การแยก Timer จึงป้องกันไม่ให้ Task ที่ค้างอยู่หลอก Watchdog ได้

2. **Pattern Timing**: อธิบายการเลือก Timer Period สำหรับแต่ละ pattern
- Timer Period ถูกกำหนดขึ้นอยู่กับ ลักษณะการรับรู้ทางสายตา และ จุดประสงค์ในการสื่อสาร ของแต่ละ Pattern
- Period ยาว (เช่น 1000 ms): ใช้สำหรับสถานะ ไม่เร่งด่วน หรือ Idle (เช่น SLOW_BLINK, OFF เพื่อประหยัดพลังงาน)
- Period สั้น (เช่น 100–300 ms): ใช้สำหรับ การเตือน หรือการแสดง กิจกรรมที่ต่อเนื่อง (เช่น FAST_BLINK, RAINBOW) เพื่อดึงดูดความสนใจและสื่อถึงความถี่ของ Event
- Period ผสม (เช่น SOS): ต้องใช้ Period ที่สัมพันธ์กันเพื่อสร้างจังหวะ รหัสมอร์ส ที่มนุษย์สามารถถอดรหัสได้

3. **Sensor Adaptation**: ประโยชน์ของ Adaptive Sampling Rate คืออะไร?
- Adaptive Sampling Rate คือการ ปรับเปลี่ยนความถี่ในการเก็บข้อมูล ตามเงื่อนไขของข้อมูลที่กำลังถูกวัด
- ประโยชน์หลักคือ:
  - การประหยัดทรัพยากร: ลดภาระ CPU และการใช้พลังงาน เมื่อค่าเซนเซอร์ คงที่ (Sampling ช้าลง)
  - การตอบสนองที่รวดเร็วขึ้น: เพิ่มความถี่ในการเก็บข้อมูลทันทีเมื่อตรวจพบ การเปลี่ยนแปลงอย่างรวดเร็ว เพื่อให้ได้ข้อมูลที่ละเอียดในสถานการณ์วิกฤต

4. **System Health**: metrics ใดบ้างที่ควรติดตามในระบบจริง?
- การติดตามสุขภาพระบบควรมีหลายมิติเพื่อวินิจฉัยปัญหาเชิงลึก:
  - ความเสถียรของระบบ (System Stability): ติดตาม Uptime, จำนวนการรีสตาร์ท (Restart Count), และเหตุผลการรีสตาร์ทล่าสุด (เช่น Watchdog reset)
  - การใช้ทรัพยากร (Resource Usage): ตรวจสอบ หน่วยความจำ Heap ที่ว่าง (Free Heap), โหลด CPU (CPU Load), และ ระดับ Stack ที่เหลือ (Stack High Water Mark) เพื่อตรวจหา Memory Leak หรือ Stack Overflow
  - การตอบสนองของ Timer (Timing Integrity): ติดตามจำนวน Timer Timeout และความถี่ที่ Watchdog ถูก Feed เพื่อยืนยันว่า Scheduling ยังทำงานตามปกติ

# Lab 3: Advanced Timer Management & Performance 
### 📋 Advanced Analysis Questions
1. Service Task Priority: ผลกระทบของ Priority ต่อ Timer Accuracy?
-  Timer Service Task ที่มี Priority สูง จะส่งผลให้ ความแม่นยำของ Timer สูง (Low Jitter) เนื่องจาก Task นี้จะได้รับ CPU ทันทีที่ Tick Interrupt เกิดขึ้น ทำให้ Callback Function ทำงานใกล้เคียงกับเวลาที่กำหนด
- หาก Priority ต่ำ Callback อาจถูก Task อื่นที่มี Priority สูงกว่า Preempt ทำให้เกิด ความคลาดเคลื่อนสะสม (Accumulated Drift) ในช่วงเวลา (Phase Shift) และ ความแม่นยำของเวลาลดลง

2. Callback Performance: วิธีการเพิ่มประสิทธิภาพ Callback Functions?
- หลีกเลี่ยงการบล็อก (Avoid Blocking): Callback ควรเป็นโค้ดที่ สั้นที่สุด ห้ามใช้ vTaskDelay() หรือวนลูปนาน
- จำกัดภาระงาน: ให้ Callback มีหน้าที่เพียง ส่งสัญญาณ (Notify) หรือ ส่ง Event ผ่าน Queue/Semaphore ไปยัง Worker Task ที่มีเวลาทำงานของตัวเอง
- การจัดการหน่วยความจำ: ควร หลีกเลี่ยงการจัดสรรหน่วยความจำแบบพลวัต (Dynamic Allocation) เช่น malloc() ภายใน Callback เพื่อลดความเสี่ยงของ Heap Fragmentation

3. Memory Management: กลยุทธ์การจัดการ Memory สำหรับ Dynamic Timers?
- ควรจัดสรร Timer Objects ใน Static Pool (อาร์เรย์คงที่) ก่อน เพื่อลดการพึ่งพา Heap
- หากจำเป็นต้องใช้ Dynamic Timer ควรมีการ จำกัดจำนวนสูงสุด และใช้กลไก ตรวจสอบและนำวัตถุกลับมาใช้ใหม่ (Reuse Object) แทนการสร้างใหม่เสมอเพื่อลด Fragmentation และการรั่วไหลของหน่วยความจำ (Memory Leak)

4. Error Recovery: วิธีการ Handle Timer System Failures?
- การบูรณาการ Watchdog: ให้ Watchdog ตรวจสอบ Task ที่รับผิดชอบ Timer Service โดยเฉพาะ หาก Task นี้ Hang ให้สั่ง รีเซ็ตระบบ ทันที
- การจับสถานะ (Status Checking): ควรมีการ ตรวจสอบสถานะ ของ Timer เป็นระยะ (เช่น xTimerIsTimerActive()) และ เริ่มต้น Timer ใหม่ (Auto-Recreate) หากพบว่า Timer ที่สำคัญหยุดทำงานอย่างไม่คาดคิด
- Redundancy (ความซ้ำซ้อน): ในระบบวิกฤต อาจใช้ Timer สำรอง (Backup Timer) เพื่อทำหน้าที่แทนเมื่อ Timer หลักล้มเหลว

5. Production Deployment: การปรับแต่งสำหรับ Production Environment?
- การปรับความถี่ Tick Rate: ปรับ configTICK_RATE_HZ ให้เหมาะสม โดยใช้ค่าสูง (เช่น 1000 Hz) หากต้องการความแม่นยำระดับ 1 ms และใช้ค่าต่ำ (เช่น 100–250 Hz) เพื่อ ประหยัดพลังงาน
- การปักหมุด Task (Pinning): ในชิป Dual-Core (เช่น ESP32) ควรใช้ xTaskCreatePinnedToCore() เพื่อกำหนดให้ Timer/Service Task รันบน Core ใด Core หนึ่งโดยเฉพาะ เพื่อ ลด Jitter ที่เกิดจากการขัดจังหวะ
- Adaptive Control: เปิดใช้งานระบบควบคุมอัตโนมัติ (เช่น การลด Sampling Rate เมื่อโหลดสูง) เพื่อให้ระบบ ปรับตัว ตามสภาพแวดล้อมจริงและรักษาเสถียรภาพไว้