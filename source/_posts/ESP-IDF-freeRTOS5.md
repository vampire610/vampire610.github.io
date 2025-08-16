---
title: ESP-IDF与freeRTOS(五) 二值信号量
date: 2023-08-06 15:34:14
tags:
  - ESP-IDF
  - freeRTOS
categories:
  - 学习笔记
---

# ESP-IDF与freeRTOS(五) 二值信号量

## 二值信号量

> 2023年8月8日 更正：二值信号量与互斥信号量mutex不同，mutex专门用于互斥访问，而二值信号量不是，具体区别：(来自FreeRTOS官网)  
> 二进制信号量和互斥锁极为相似，但存在一些细微差别：互斥锁包括优先继承机制，而二进制信号量则不然 。这使得二进制信号器成为实现同步（任务之间或任务与中断之间） 的更好选择，而互斥锁则是实现简单互斥的更好选择。 关于互斥锁如何被用作互斥机制的描述 同样适用于二进制信号量。
>
信号量是操作系统中用于进程同步和临界区互斥访问的关键，二值信号量多用于临界区的互斥访问或者"任务通知"

在下面的示例代码中，在两个任务的for循环中对`iCont`变量进行访问，这个for循环即相当于临界区，如果两个进程同时访问临界区，则可能导致临界区数据混乱。于是在task1临界区前Take信号量（给临界区上锁），则其他进程无法再进行Take，等到task1释放give信号量（临界区解锁）后，其他任务才能访问临界区。


示例代码：

``` c
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_spi_flash.h"

#include "freertos/semphr.h"


SemaphoreHandle_t semphrHandle;
int iCont = 0;

void myTask1(void *pvParam)
{
    while (1)
    {
        // 没有获得信号量无限等待
        xSemaphoreTake(semphrHandle, portMAX_DELAY);
        //中间即为临界区，保护iCont的互斥访问
        for (int i = 0; i < 10; i++)
        {
            iCont++;
            printf("myTask1 iCont = %d\n", iCont);
            vTaskDelay(pdMS_TO_TICKS(1000));
        }
        xSemaphoreGive(semphrHandle);
        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}

void myTask2(void *pvParam)
{
    while (1)
    {
        // 没有获得信号量无限等待
        xSemaphoreTake(semphrHandle, portMAX_DELAY);
        for (int i = 0; i < 10; i++)
        {
            iCont++;
            printf("myTask2 iCont = %d\n", iCont);
            vTaskDelay(pdMS_TO_TICKS(1000));
        }
        xSemaphoreGive(semphrHandle);
        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}

void app_main(void)
{
    // 创建二值信号量,信号量创建默认为空
    semphrHandle = xSemaphoreCreateBinary();
    // 创建完成的时候释放信号量
    xSemaphoreGive(semphrHandle);

    xTaskCreatePinnedToCore(myTask1, "myTask1", 1024*5, NULL, 1, NULL, 1);
    xTaskCreatePinnedToCore(myTask2, "myTask2", 1024*5, NULL, 1, NULL, 1);
}

```

