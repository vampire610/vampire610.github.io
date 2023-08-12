---
title: ESP-IDF与freeRTOS(八)
date: 2023-08-10 13:36:04
tags:
  - ESP-IDF
  - freeRTOS
categories:
  - 学习笔记
---

# ESP-IDF与freeRTOS(八)

## 事件组

所需头文件`event_groups.h`

通过宏定义`configUSE_16_BIT_TICKS`来查看事件组是几位，若宏定义为0是24位。宏定义可通过vscode搜索，我的路径在`components\freertos\esp_additions\include\freertos\FreeRTOSConfig.h`

这次主要学习`xEventGroupWaitBits()`函数，该函数不可在中断中调用

``` c
// 返回值
EventBits_t xEventGroupWaitBits(const EventGroupHandle_t xEventGroup,//事件组句柄
                                    const EventBits_t uxBitsToWaitFor,//设置需要检查哪些二进制位，此处输入十六进制
                                    const BaseType_t xClearOnExit,//如果设为pdTRUE，退出时清零对应位
                                    const BaseType_t xWaitForAllBits,//设置等待事件组中的位同时(AND)满足或任意一位(OR)满足
                                    TickType_t xTicksToWait);//设置阻塞等待时间
```





### 运行结果

可以看到运行时当task2将0或者4置为1时，任务1才会解除阻塞

``` bash
-------------
task1 begin to wait
-------------
task2 begin set bit0
-------------
In task1, BIT_0 or BIT_4 is set
-------------
task1 begin to wait
-------------
task2 begin set bit0
-------------
In task1, BIT_0 or BIT_4 is set
-------------
task1 begin to wait
-------------
task2 begin set bit0
-------------
```





### 代码

``` c
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_spi_flash.h"

#include "freertos/event_groups.h"

EventGroupHandle_t EventGroupHandle;

// 定义位,利用左移运算
#define BIT_0 (1 << 0)
#define BIT_4 (1 << 4)

void Task1(void *pvParam)
{
    while (1)
    {
        printf("-------------\n");
        printf("task1 begin to wait\n");
        /* Wait a maximum time for either bit 0 or bit 4 to be set within
            the event group. Clear the bits before exiting. */
        xEventGroupWaitBits(
            EventGroupHandle, /* The event group being tested. */
            // 0，4只要有一位满足
            // 此处可用16进制直接设置，如此处0，4位可设置0x11
            BIT_0 | BIT_4, /* The bits within the event group to wait for. */
            pdTRUE,        /* BIT_0 and BIT_4 should be cleared before returning. */
            // 此处设置为TRUE则0，4位同时置位才会接触阻塞
            pdFALSE,        /* Don't wait for both bits, either bit will do. */
            portMAX_DELAY); /* Wait a maximum time for either bit to be set. */

        printf("-------------\n");
        printf("In task1, BIT_0 or BIT_4 is set\n");

        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}

void Task2(void *pvParam)
{
    vTaskDelay(pdMS_TO_TICKS(1000));
    while (1)
    {
        printf("-------------\n");
        printf("task2 begin set bit0\n");

        xEventGroupSetBits(EventGroupHandle, BIT_0);

        vTaskDelay(pdMS_TO_TICKS(5000));

        printf("-------------\n");
        printf("task2 begin set bit4\n");

        xEventGroupSetBits(EventGroupHandle, BIT_4);

        vTaskDelay(pdMS_TO_TICKS(5000));
    }
}

void app_main(void)
{
    /* Attempt to create the event group. */
    EventGroupHandle = xEventGroupCreate();
    /* Was the event group created successfully? */
    if (EventGroupHandle == NULL)
    {
        /* The event group was not created because there was insufficient
        FreeRTOS heap available. */
        printf("Event group creat faill!\n");
    }
    else
    {
        /* The event group was created. */
        // 创建任务前挂起任务调度器
        vTaskSuspendAll();

        xTaskCreatePinnedToCore(Task1, "Task1", 1024 * 5, NULL, 1, NULL, 0);
        xTaskCreatePinnedToCore(Task2, "Task2", 1024 * 5, NULL, 1, NULL, 0);

        // 任务创建结束恢复任务调度器
        xTaskResumeAll();
    }
}

```





## 事件组同步

`xEventGroupWaitBits()`与`xEventGroupSync()`很相似，可参考如下场景

![使用xEventGroupWaitBits()](https://fastly.jsdelivr.net/gh/vampire610/PicGo/Blog-PIC/20230810160017.png)

而使用事件组同步的时候，需要同步的任务会阻塞直到最后一个需要同步的事件组置位，然后同时运行，实现同步。

``` c
EventBits_t xEventGroupSync(EventGroupHandle_t xEventGroup,//事件组句柄
                            const EventBits_t uxBitsToSet,//当前任务要设置的位
                            const EventBits_t uxBitsToWaitFor,//当前任务等待的位
                            TickType_t xTicksToWait);//等待时间
```



### 运行结果

可以看到当3个任务都运行置位之后三个任务才一起执行后面的任务。

``` bash
-------------
task0 begin
-------------
task1 begin
-------------
task2 begin
task2 sync
task0 sync
task1 sync
```

### 代码

``` c
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_spi_flash.h"

#include "freertos/event_groups.h"

EventGroupHandle_t EventGroupHandle;

/* Bits used by the three tasks. */
#define TASK_0_BIT (1 << 0)
#define TASK_1_BIT (1 << 1)
#define TASK_2_BIT (1 << 2)
#define ALL_SYNC_BITS (TASK_0_BIT | TASK_1_BIT | TASK_2_BIT)

void Task0(void *pvParam)
{
    while (1)
    {
        printf("-------------\n");
        printf("task0 begin\n");

        vTaskDelay(pdMS_TO_TICKS(1000));

        xEventGroupSync(EventGroupHandle, TASK_0_BIT, ALL_SYNC_BITS, portMAX_DELAY);
        printf("task0 sync\n");
        vTaskDelay(pdMS_TO_TICKS(10000));
    }
}

void Task1(void *pvParam)
{
    while (1)
    {
        printf("-------------\n");
        printf("task1 begin\n");

        vTaskDelay(pdMS_TO_TICKS(3000));

        xEventGroupSync(EventGroupHandle, TASK_1_BIT, ALL_SYNC_BITS, portMAX_DELAY);
        printf("task1 sync\n");
        vTaskDelay(pdMS_TO_TICKS(10000));
    }
}

void Task2(void *pvParam)
{
    while (1)
    {
        printf("-------------\n");
        printf("task2 begin\n");

        vTaskDelay(pdMS_TO_TICKS(5000));

        xEventGroupSync(EventGroupHandle, TASK_2_BIT, ALL_SYNC_BITS, portMAX_DELAY);
        printf("task2 sync\n");
        vTaskDelay(pdMS_TO_TICKS(10000));
    }
}

void app_main(void)
{
    /* Attempt to create the event group. */
    EventGroupHandle = xEventGroupCreate();
    /* Was the event group created successfully? */
    if (EventGroupHandle == NULL)
    {
        /* The event group was not created because there was insufficient
        FreeRTOS heap available. */
        printf("Event group creat faill!\n");
    }
    else
    {
        /* The event group was created. */
        // 创建任务前挂起任务调度器
        vTaskSuspendAll();

        xTaskCreatePinnedToCore(Task0, "Task0", 1024 * 5, NULL, 1, NULL, 0);
        xTaskCreatePinnedToCore(Task1, "Task1", 1024 * 5, NULL, 1, NULL, 0);
        xTaskCreatePinnedToCore(Task2, "Task2", 1024 * 5, NULL, 1, NULL, 0);

        // 任务创建结束恢复任务调度器
        xTaskResumeAll();
    }
}

```

