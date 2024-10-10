# MQTT_TCP_7Segment

1. สร้าง project ใหม่ จาก example ที่ชื่อ mqtt->tcp

2. build 1 ครั้ง เพื่อทดสอบว่ามี error หรือไม่ ถ้ามี ให้แก้ไขให้ถูกต้อง
        
   2.1 error อาจจะเกิดจากเลือก version ของ idf ไม่ถูกต้อง หรือเลือก serial port  ไม่ถูกต้อง

   2.2 ตรวจสอบดูว่าได้ติดตั้ง driver ของอุปกรณ์ต่างๆ ไว้ครบถ้วนแล้ว

3. กดปุ่ม menuconfig (รูปเฟือง) หรือเลือกจากเมนู เพื่อแก้ไข mqtt broker และ wifi SSID./Password

4. ดึง component `LED` และ `SevenSegment` ที่เคยทำไว้แล้ว

5.       
