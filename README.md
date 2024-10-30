# ESP32-FreeRTOS-Cookbook

ตัวอย่างที่นำมาใช้คือ การทดลอง ESP32 FreeRTOS เรื่อง การสร้าง Task เบื้องต้น และการใช้งาน ฟังก์ชัน xTaskCreate() นำมาจาก Project ESP32-FreeRTOS-Intro

### ส่วนของการแก้ไข



### ขั้นตอนการทำ project

1. สร้าง project ใหม่ใน ESP-IDF เพื่อทำงานกับ FreeRTOS
   1.1 ชื่อ project-esp-FreeRTOS_Task

![image](https://github.com/user-attachments/assets/797c5eb0-fb09-4272-b90c-d1cdfb46c0e1)


   1.2 ไม่ต้องเลือก Template (ใช้เทมเพลตพื้นฐาน)
   1.3 เมื่อสร้างแล้ว ให้แก้ไข Code ดังนี้
เพิ่ม include files
ลบ code ใน app_main() แล้วเพิ่มcode ตามรูปต่อไปนี้

![image](https://github.com/user-attachments/assets/54e29f6f-6230-4066-bf4f-1e2ec4ba0bc0)


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


ผลลัพน์คือ



สรุปผล:



ลิงค์ Project ที่ทำบน VS code :
