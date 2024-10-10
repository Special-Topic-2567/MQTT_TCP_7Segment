# MQTT_TCP_7Segment

### สร้าง project จากตัวอย่าง
1. สร้าง project ใหม่ จาก example ที่ชื่อ mqtt->tcp

2. build 1 ครั้ง เพื่อทดสอบว่ามี error หรือไม่ ถ้ามี ให้แก้ไขให้ถูกต้อง
        
   2.1 error อาจจะเกิดจากเลือก version ของ idf ไม่ถูกต้อง หรือเลือก serial port  ไม่ถูกต้อง

   2.2 ตรวจสอบดูว่าได้ติดตั้ง driver ของอุปกรณ์ต่างๆ ไว้ครบถ้วนแล้ว

3. กดปุ่ม menuconfig (รูปเฟือง) หรือเลือกจากเมนู เพื่อแก้ไข mqtt broker และ wifi SSID./Password

### ดึง component ที่ได้สร้างไว้ มา reuse

4. ดึง component `LED` และ `SevenSegment` ที่เคยทำไว้แล้ว โดยแก้ไขไฟล์ idf_component.yml ให้มีเนื้อหาดังตัวอย่างต่อไปนี

``` yml
dependencies:
  protocol_examples_common:
    path: ${IDF_PATH}/examples/common_components/protocol_examples_common
  espressif/esp_wifi_remote:
    version: ">=0.1.12"
    rules:
      - if: "target in [esp32p4, esp32h2]"
  LED:
    git: https://github.com/koson/LED_Test.git
    path: components/LED
  SEVENSEGMENT:
    git: https://github.com/koson/sevenseg_component.git
    path: components/SevenSegment 
    version: 57dca1419ad633cefb7d1f9ef2871cc222f5c3ed
```
ส่วนที่ต้องแก้คือ git url ใน  LED และ SEVENSEGMENT

5. build  1 ครั้ง จะได้โฟลเดอร์ managecomponent และมี 2 โฟลเดอร์ย่อยอยู่ด้านใน คือ LED และ SEVENSEGMENT

### เชื่อมโยง code ที่ได้จากตัวอย่าง (ภาษา C) และ code จาก component (ภาษา C++)
 
6. เพิ่ม code สำหรับการเชื่อมโยงระหว่างภาษา C และ C++ ไว้ใน main รวมทั้งสิ้น 4 ไฟล์  
 
6.1 ไฟล์ led_c_connector.cpp

``` cpp
#include "led_c_connector.h"
#include "LED.h"

#ifdef __cplusplus
extern "C" {
#endif

// Inside this "extern C" block, I can implement functions in C++, which will externally 
//   appear as C functions (which means that the function IDs will be their names, unlike
//   the regular C++ behavior, which allows defining multiple functions with the same name
//   (overloading) and hence uses function signature hashing to enforce unique IDs),


LED led1(16); 

void LED_ON() 
{
    led1.ON();
}

void LED_OFF() 
{
    led1.OFF();
}

#ifdef __cplusplus
}
#endif
```

6.2 ไฟล์ led_c_connector.h

``` cpp
#ifndef LED_C_CONNECTOR_H
#define LED_C_CONNECTOR_H

#ifdef __cplusplus
extern "C"
{
#endif
    void LED_ON();
    void LED_OFF();

#ifdef __cplusplus
}
#endif

#endif
```
      

6.3 ไฟล์ sevensegment_c_connector.cpp

```cpp
#include "sevensegment_c_connector.h"
 #include "SevenSegment.h"

#ifdef __cplusplus
extern "C" {
#endif

SevenSegment s1(0);
SevenSegment s2(4);

void s1_displayOn()
{
    s1.DisplayOn();    
}

void s1_displayOff()
{
    s1.DisplayOff();
}

void s2_displayOn()
{
    s2.DisplayOn();    
}

void s2_displayOff()
{
    s2.DisplayOff();
}

void s1_displayNumber(int number)
{
    s1.DisplayNumber(number);
}

void s2_displayNumber(int number)
{
    s2.DisplayNumber(number);
}

#ifdef __cplusplus
}
#endif
```
6.4 ไฟล์ sevensegment_c_connector.h
```cpp
#ifndef SEVENSEGMENT_C_CONNECTOR_H
#define SEVENSEGMENT_C_CONNECTOR_H

#ifdef __cplusplus
extern "C"
{
#endif
    void s1_displayOn();
    void s1_displayOff();
    void s2_displayOn();
    void s2_displayOff();
    void s1_displayNumber(int number);
    void s2_displayNumber(int number);

#ifdef __cplusplus
}
#endif

#endif
```

7. ตรวจสอบไฟล์ CMakeLists.txt ใน main ว่ามีการเพิ่มไฟล์ connector แล้วหรือไม่
    - ถ้ายังไม่เพิ่ม ให้เพิ่มด้วย
    - เมื่อเพิ่มแล้ว ไฟล์ CMakeLists.txt จะมีเนื้อหาตามตัวอย่าง
  
```CMake
idf_component_register(
          SRCS "sevensegment_c_connector.cpp"
                "led_c_connector.cpp"
                "app_main.c"
          INCLUDE_DIRS ".")
```      



9.   build ดูว่ามี error หรือไม่

### แก้ไขไฟล์ main ของโปรเจคตัวอย่าง
