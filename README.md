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

10. add sevensegment_c_connector.h include file

11. add number variable and TaskHandle_t

12. add subscribe topic

13. add code to get number from mqtt topic

14. add  scan seven segment task

15. create and run seven segmant task

```c

/* MQTT (over TCP) Example

   This example code is in the Public Domain (or CC0 licensed, at your option.)

   Unless required by applicable law or agreed to in writing, this
   software is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR
   CONDITIONS OF ANY KIND, either express or implied.
*/

#include <stdio.h>
#include <stdint.h>
#include <stddef.h>
#include <string.h>
#include "esp_wifi.h"
#include "esp_system.h"
#include "nvs_flash.h"
#include "esp_event.h"
#include "esp_netif.h"
#include "protocol_examples_common.h"

#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "freertos/semphr.h"
#include "freertos/queue.h"

#include "lwip/sockets.h"
#include "lwip/dns.h"
#include "lwip/netdb.h"

#include "esp_log.h"
#include "mqtt_client.h"

// 10. add sevensegment_c_connector.h include file
#include "sevensegment_c_connector.h"

// 11. add number variable and TaskHandle_t
uint8_t number = 0;
TaskHandle_t xSevenSegmentHandle = NULL;


static const char *TAG = "mqtt_example";


static void log_error_if_nonzero(const char *message, int error_code)
{
    if (error_code != 0) {
        ESP_LOGE(TAG, "Last error %s: 0x%x", message, error_code);
    }
}

/*
 * @brief Event handler registered to receive MQTT events
 *
 *  This function is called by the MQTT client event loop.
 *
 * @param handler_args user data registered to the event.
 * @param base Event base for the handler(always MQTT Base in this example).
 * @param event_id The id for the received event.
 * @param event_data The data for the event, esp_mqtt_event_handle_t.
 */
static void mqtt_event_handler(void *handler_args, esp_event_base_t base, int32_t event_id, void *event_data)
{
    ESP_LOGD(TAG, "Event dispatched from event loop base=%s, event_id=%" PRIi32 "", base, event_id);
    esp_mqtt_event_handle_t event = event_data;
    esp_mqtt_client_handle_t client = event->client;
    int msg_id;
    switch ((esp_mqtt_event_id_t)event_id) {
    case MQTT_EVENT_CONNECTED:
        ESP_LOGI(TAG, "MQTT_EVENT_CONNECTED");
        msg_id = esp_mqtt_client_publish(client, "/topic/qos1", "data_3", 0, 1, 0);
        ESP_LOGI(TAG, "sent publish successful, msg_id=%d", msg_id);

        msg_id = esp_mqtt_client_subscribe(client, "/topic/qos0", 0);
        ESP_LOGI(TAG, "sent subscribe successful, msg_id=%d", msg_id);

        // 12. add subscribe topic
        msg_id = esp_mqtt_client_subscribe(client, "/topic/7seg", 0);
        ESP_LOGI(TAG, "sent subscribe successful, msg_id=%d", msg_id);


        msg_id = esp_mqtt_client_subscribe(client, "/topic/qos1", 1);
        ESP_LOGI(TAG, "sent subscribe successful, msg_id=%d", msg_id);

        msg_id = esp_mqtt_client_unsubscribe(client, "/topic/qos1");
        ESP_LOGI(TAG, "sent unsubscribe successful, msg_id=%d", msg_id);
        break;
    case MQTT_EVENT_DISCONNECTED:
        ESP_LOGI(TAG, "MQTT_EVENT_DISCONNECTED");
        break;

    case MQTT_EVENT_SUBSCRIBED:
        ESP_LOGI(TAG, "MQTT_EVENT_SUBSCRIBED, msg_id=%d", event->msg_id);
        msg_id = esp_mqtt_client_publish(client, "/topic/qos0", "data", 0, 0, 0);
        ESP_LOGI(TAG, "sent publish successful, msg_id=%d", msg_id);
        break;
    case MQTT_EVENT_UNSUBSCRIBED:
        ESP_LOGI(TAG, "MQTT_EVENT_UNSUBSCRIBED, msg_id=%d", event->msg_id);
        break;
    case MQTT_EVENT_PUBLISHED:
        ESP_LOGI(TAG, "MQTT_EVENT_PUBLISHED, msg_id=%d", event->msg_id);
        break;
    case MQTT_EVENT_DATA:
        ESP_LOGI(TAG, "MQTT_EVENT_DATA");
        printf("TOPIC=%.*s\r\n", event->topic_len, event->topic);
        printf("DATA=%.*s\r\n", event->data_len, event->data);

        // 13. add code to get number from mqtt topic
        if(strstr(event->topic,"/topic/7seg") != NULL)
        {
            number  = (event->data[0]-0x30)*10 + (event->data[1]-0x30);
        }
        break;
    case MQTT_EVENT_ERROR:
        ESP_LOGI(TAG, "MQTT_EVENT_ERROR");
        if (event->error_handle->error_type == MQTT_ERROR_TYPE_TCP_TRANSPORT) {
            log_error_if_nonzero("reported from esp-tls", event->error_handle->esp_tls_last_esp_err);
            log_error_if_nonzero("reported from tls stack", event->error_handle->esp_tls_stack_err);
            log_error_if_nonzero("captured as transport's socket errno",  event->error_handle->esp_transport_sock_errno);
            ESP_LOGI(TAG, "Last errno string (%s)", strerror(event->error_handle->esp_transport_sock_errno));

        }
        break;
    default:
        ESP_LOGI(TAG, "Other event id:%d", event->event_id);
        break;
    }
}

static void mqtt_app_start(void)
{
    esp_mqtt_client_config_t mqtt_cfg = {
        .broker.address.uri = CONFIG_BROKER_URL,
    };
#if CONFIG_BROKER_URL_FROM_STDIN
    char line[128];

    if (strcmp(mqtt_cfg.broker.address.uri, "FROM_STDIN") == 0) {
        int count = 0;
        printf("Please enter url of mqtt broker\n");
        while (count < 128) {
            int c = fgetc(stdin);
            if (c == '\n') {
                line[count] = '\0';
                break;
            } else if (c > 0 && c < 127) {
                line[count] = c;
                ++count;
            }
            vTaskDelay(10 / portTICK_PERIOD_MS);
        }
        mqtt_cfg.broker.address.uri = line;
        printf("Broker url: %s\n", line);
    } else {
        ESP_LOGE(TAG, "Configuration mismatch: wrong broker url");
        abort();
    }
#endif /* CONFIG_BROKER_URL_FROM_STDIN */

    esp_mqtt_client_handle_t client = esp_mqtt_client_init(&mqtt_cfg);
    /* The last argument may be used to pass data to the event handler, in this example mqtt_event_handler */
    esp_mqtt_client_register_event(client, ESP_EVENT_ANY_ID, mqtt_event_handler, NULL);
    esp_mqtt_client_start(client);
}

// 14. add  scan seven segment task

void vTaskScanSevenSegment(void *Parameters)
{
    while (1)
    {
        s1_displayNumber(number / 10);
        s1_displayOn();
        vTaskDelay(10 / portTICK_PERIOD_MS);
        s1_displayOff();

        s2_displayNumber(number % 10);
        s2_displayOn();
        vTaskDelay(10 / portTICK_PERIOD_MS);
        s2_displayOff();
    }
}


void app_main(void)
{
    ESP_LOGI(TAG, "[APP] Startup..");
    ESP_LOGI(TAG, "[APP] Free memory: %" PRIu32 " bytes", esp_get_free_heap_size());
    ESP_LOGI(TAG, "[APP] IDF version: %s", esp_get_idf_version());

    esp_log_level_set("*", ESP_LOG_INFO);
    esp_log_level_set("mqtt_client", ESP_LOG_VERBOSE);
    esp_log_level_set("mqtt_example", ESP_LOG_VERBOSE);
    esp_log_level_set("transport_base", ESP_LOG_VERBOSE);
    esp_log_level_set("esp-tls", ESP_LOG_VERBOSE);
    esp_log_level_set("transport", ESP_LOG_VERBOSE);
    esp_log_level_set("outbox", ESP_LOG_VERBOSE);

    ESP_ERROR_CHECK(nvs_flash_init());
    ESP_ERROR_CHECK(esp_netif_init());
    ESP_ERROR_CHECK(esp_event_loop_create_default());

    /* This helper function configures Wi-Fi or Ethernet, as selected in menuconfig.
     * Read "Establishing Wi-Fi or Ethernet Connection" section in
     * examples/protocols/README.md for more information about this function.
     */
    ESP_ERROR_CHECK(example_connect());

    // 15. create and run seven segmant task
    xTaskCreate(vTaskScanSevenSegment, "Seven Seg", 
        1024, NULL, 10, &xSevenSegmentHandle);
    mqtt_app_start();
}

```

### การทดสอบ

16. Flash  โปรแกรมลงใน ESP32
17. ส่งตัวเลข 2 หลักจาก mqttexplorer ผ่านทาง topic ที่กำหนดในข้อ 13
18. ตรวจสอบว่ามีเลขที่ป้อนทาง mqtt explorer ปรากฏที่ 7 segment อย่างถูกต้องหรือไม่



# งานที่ทำบน vs coden ขึ้น git  hub

https://github.com/AnchisaPhetnoi/ESP_MQTT_7segment.git
