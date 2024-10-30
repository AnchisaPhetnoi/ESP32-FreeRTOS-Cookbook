# ESP32-FreeRTOS-Cookbook

ตัวอย่างที่นำมาใช้คือ การทดลอง ESP32 FreeRTOS เรื่อง การสร้าง Task เบื้องต้น และการใช้งาน ฟังก์ชัน xTaskCreate() นำมาจาก Project ESP32-FreeRTOS-Intro

### ส่วนของการแก้ไข

แก้ไขในโค้ด main.cpp โดยการแก้ไขเป็นแบบโค้ดแบบใช้ Task เบื้องต้น กับ โค้ดแบบใช้ ฟังก์ชัน xTaskCreate() ให้แสดงผลลัพน์ที่ไม่เหมือนในตัวอย่าง การสร้าง Task เบื้องต้น จะแก้ไขโค้ดให้แสดงผล การนับถอยหลังที่แสดงผลบนคอนโซลทุกวินาที โดยวนไปที่ตัวเลข 10 ถึง 0
และในส่วนของ การใช้งาน ฟังก์ชัน xTaskCreate() จะเป็นการสร้างโค้ดใหม่ใน main.cpp โดยต้องการให้ ProducerTask ส่งข้อมูลเข้า Queue และ ConsumerTask อ่านข้อมูลจาก Queue และให้  MonitorTask แจ้งเตือนเมื่อ Queue เต็ม

### ขั้นตอนการทำ project

1. สร้าง project ใหม่ใน ESP-IDF เพื่อทำงานกับ FreeRTOS
   1.1 ชื่อ project-esp-FreeRTOS_Task

![image](https://github.com/user-attachments/assets/797c5eb0-fb09-4272-b90c-d1cdfb46c0e1)


   1.2 ไม่ต้องเลือก Template (ใช้เทมเพลตพื้นฐาน)

   
   1.3 เมื่อสร้างแล้ว ให้แก้ไข Code ดังนี้
   
เพิ่ม include files

ลบ code ใน app_main() แล้วเพิ่มcode ตามรูปต่อไปนี้

![image](https://github.com/user-attachments/assets/54e29f6f-6230-4066-bf4f-1e2ec4ba0bc0)

## การสร้าง Task เบื้องต้น

``` cpp
#include <stdio.h>
#include <stdbool.h>
#include <unistd.h>

#include "freertos/FreeRTOS.h"
#include "freertos/task.h"

TaskHandle_t MyFirstTaskHandle = NULL;

void My_First_Task(void* arg)
{
    int i = 10; // เริ่มนับจาก 10
    while(1)
    {
        printf("Countdown: %d\n", i);
        vTaskDelay(1000 / portTICK_PERIOD_MS);
        i--;
        if (i < 0) i = 10; // ถ้าถึง 0 แล้วก็เริ่มใหม่ที่ 10
    }
}

```

Build และทดสอบบนบอร์ด ESP32

![image](https://github.com/user-attachments/assets/bd4ab7f1-7ea9-4c7c-9c18-b1ae283c8c63)

![image](https://github.com/user-attachments/assets/2cd1af41-2929-4798-b790-92c7a39a018a)


จะเห็นผลลัพน์ที่ได้ คือ  การนับถอยหลังที่แสดงผลบนคอนโซลทุกวินาที โดยวนไปที่ตัวเลข 10 ถึง 0

## ฟังก์ชัน xTaskCreate()

1.4 ฟังก์ชัน xTaskCreate() ลบ code ใน app_main() แล้วเพิ่มcode ตามรูปต่อไปนี้

![image](https://github.com/user-attachments/assets/1c6899b7-9503-4fdb-b024-b1615d835b68)

![image](https://github.com/user-attachments/assets/61f7c95d-7ca6-4422-ad96-e8ffcef91ca6)


![image](https://github.com/user-attachments/assets/d18b82e4-7474-4a61-ba3c-f5ce28cb201d)

``` cpp
#include <stdio.h>
#include <stdint.h>

#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "freertos/queue.h"
#include "freertos/event_groups.h"

QueueHandle_t dataQueue;
EventGroupHandle_t eventGroup;

#define QUEUE_FULL_EVENT (1 << 0)

// Task สำหรับส่งข้อมูลไปยัง Queue
void ProducerTask(void *arg)
{
    uint16_t i = 0;
    while (1)
    {
        printf("ProducerTask: Sending %d to queue\n", i);
        xQueueSend(dataQueue, &i, portMAX_DELAY);
        i++;
        
        // เมื่อ Queue เต็ม ส่งสัญญาณ Event
        if (uxQueueMessagesWaiting(dataQueue) == 5)
        {
            xEventGroupSetBits(eventGroup, QUEUE_FULL_EVENT);
        }
        vTaskDelay(500 / portTICK_PERIOD_MS); // หน่วง 0.5 วินาที
    }
}

// Task สำหรับรับข้อมูลจาก Queue
void ConsumerTask(void *arg)
{
    uint16_t receivedValue;
    while (1)
    {
        if (xQueueReceive(dataQueue, &receivedValue, portMAX_DELAY))
        {
            printf("ConsumerTask: Received %d from queue\n", receivedValue);
        }
    }
}

// Task ที่จะทำงานเมื่อ Queue เต็ม
void MonitorTask(void *arg)
{
    while (1)
    {
        xEventGroupWaitBits(eventGroup, QUEUE_FULL_EVENT, pdTRUE, pdFALSE, portMAX_DELAY);
        printf("MonitorTask: Queue is full, processing data...\n");
    }
}

void app_main(void)
{
    // สร้าง Queue และ Event Group
    dataQueue = xQueueCreate(5, sizeof(uint16_t));
    eventGroup = xEventGroupCreate();

    // สร้าง Task สำหรับ Producer, Consumer และ Monitor
    xTaskCreate(ProducerTask, "ProducerTask", 2048, NULL, 10, NULL);
    xTaskCreate(ConsumerTask, "ConsumerTask", 2048, NULL, 10, NULL);
    xTaskCreate(MonitorTask, "MonitorTask", 2048, NULL, 10, NULL);
}
```

รันโปรแกรมและอธิบายผลที่ได้

![image](https://github.com/user-attachments/assets/e59245ee-3f72-4dca-86a9-0335f837ab76)

![image](https://github.com/user-attachments/assets/2ac96a36-b762-486b-8c12-905ec7c3d4d1)

จะเห็นผลลัพธ์ที่ ProducerTask ส่งข้อมูลเข้า Queue และ ConsumerTask อ่านข้อมูลจาก Queue นอกจากนี้จะเห็น MonitorTask แจ้งเตือนเมื่อ Queue เต็ม


สรุปผล:  การสร้าง Task เบื้องต้น นั้น ในผลลัพน์นี้จะเป็น การแสดงค่าที่ลดลงในลูปไม่มีที่สิ้นสุด ส่วน การสร้าง xTask โดยใช้ ฟังก์ชัน xTaskCreate() นั้นเป็นระบบที่ซับซ้อนกว่า โดยมีการสื่อสารระหว่างงานหลายงานผ่าน Queue และ Event Group ซึ่งทำให้สามารถจัดการการส่งและรับข้อมูลได้อย่างมีประสิทธิภาพ โดยการแสดงผลทั้งหมดนั้นเป็นการแสดงผลทดสอบการใช้งาน Task แบบเบื้องต้น และแบบ xTaskCreate() ของ ESP32 FreeRTOS



ลิงค์ Project ที่ทำบน VS code : https://github.com/AnchisaPhetnoi/project-esp-FreeRTOS_Task.git
