---
title: ESP-IDF与freeRTOS(六) 计数信号量
date: 2023-08-08 20:39:38
tags:
  - ESP-IDF
  - freeRTOS
categories:
  - 学习笔记
---

# ESP-IDF与freeRTOS(六) 计数信号量

## 计数型信号量

用于记录剩余资源的数量，当没有资源的时候将等待资源产生。

可以看作是一个停车场，计数型信号量记录剩余车位，信号量用完时将不允许再停车，当有车离开时信号量增加，将允许车辆进入。

示例代码：

``` c
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_spi_flash.h"

#include "freertos/semphr.h"

// #include "myled.h"

SemaphoreHandle_t semphrHandle;

void carInTask(void *pvParam)
{
    int emptySpace = 0;
    BaseType_t iResult;
    while (1)
    {
        // 获取计数型信号量剩余值，停车场剩余车位
        emptySpace = uxSemaphoreGetCount(semphrHandle);
        printf("\nemptySpace = %d!\n", emptySpace);
        // 取得、消耗信号量，车进入停车场
        iResult = xSemaphoreTake(semphrHandle, 0);

        if (iResult == pdPASS)
        {
            // 成功取得
            printf("One car in\n");
        }
        else
        {
            // 取得失败
            printf("No space\n");
        }
        vTaskDelay(pdMS_TO_TICKS(100));
    }
}

void carOutTask(void *pvParam)
{
    while (1)
    {
        vTaskDelay(pdMS_TO_TICKS(600));
        // give信号，车开出停车场
        xSemaphoreGive(semphrHandle);
        printf("\nOne car out\n");
    }
}

void app_main(void)
{
    // 创建计数信号量,需要参数最大值和初始值，创建时已经指定剩余为5，不需要开始就give
    semphrHandle = xSemaphoreCreateCounting(5, 5);

    xTaskCreatePinnedToCore(carInTask, "carInTask", 1024 * 5, NULL, 1, NULL, 1);
    xTaskCreatePinnedToCore(carOutTask, "carOutTask", 1024 * 5, NULL, 1, NULL, 1);
}

```



运行结果：

每100ms进入一辆车，每600ms开出一辆车

``` bash
...
emptySpace = 5!
One car in

emptySpace = 4!
One car in

emptySpace = 3!
One car in

emptySpace = 2!
One car in

emptySpace = 1!
One car in

emptySpace = 0!
No space

One car out

emptySpace = 1!
One car in

emptySpace = 0!
No space

emptySpace = 0!
No space

emptySpace = 0!
No space

emptySpace = 0!
No space

emptySpace = 0!
No space

One car out

emptySpace = 1!
One car in

emptySpace = 0!
No space

emptySpace = 0!
No space

emptySpace = 0!
No space

emptySpace = 0!
No space

emptySpace = 0!
No space

One car out

emptySpace = 1!
One car in
...
```

