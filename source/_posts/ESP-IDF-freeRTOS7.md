---
title: ESP-IDF与freeRTOS(七) 互斥信号量
date: 2023-08-10 10:28:06
tags:
  - ESP-IDF
  - freeRTOS
categories:
  - 学习笔记
---

# ESP-IDF与freeRTOS(七) 互斥信号量

## mutex互斥量

与二值信号量十分相似，但mutex信号量包含优先级继承机制，详情可以看`FreeRTOS`官方文档，文章末尾我也会将原文放上。

个人理解如下：

如有信号量被任务1持有，此时有高优先级的任务2想获取该信号量，任务1的优先级就会提高到任务2的优先级，直到任务1归还信号量，归还后任务1的优先级自动恢复。

互斥量测试代码通常创建3个任务。

### 分析一下任务：

1. 若不使用mutex信号量，即没有优先级继承时：

   任务1取得信号量之后，任务1为信号量的所有者，当时间片到，调度器会进行任务切换，切换后先执行task3的时间片，task3执行完时间片，会切换到优先级较低的任务2，而任务2中没有阻塞自己，之后就只有高优先级的任务3能够剥夺任务2的执行，从而一直在运行任务2，3，导致任务1一直不能够执行，信号量也将一直得不到归还。任务3也将一直阻塞自己，得不到信号量。并且IDLE任务也会因为得不到执行而触发看门狗。

2. 有优先级继承时：

   

### 运行结果：

1. 不使用mutex信号量,可以看到任务3获取不到信号量，而且任务2一直运行导致IDLE触发看门狗。

   ``` bash
   task3 begin!
   task2 begin!
   task1 begin!
   E (12271) task_wdt: Task watchdog got triggered. The following tasks/users did not reset the watchdog in time:
   E (12271) task_wdt:  - IDLE (CPU 0)
   E (12271) task_wdt: Tasks currently running:
   E (12271) task_wdt: CPU 0: Task2
   E (12271) task_wdt: CPU 1: IDLE
   E (12271) task_wdt: Print CPU 0 (current core) backtrace
   
   E (17271) task_wdt: Task watchdog got triggered. The following tasks/users did not reset the watchdog in time:
   E (17271) task_wdt:  - IDLE (CPU 0)
   E (17271) task_wdt: Tasks currently running:
   E (17271) task_wdt: CPU 0: Task2
   E (17271) task_wdt: CPU 1: IDLE
   E (17271) task_wdt: Print CPU 0 (current core) backtrace
   
   task3 didn't take!
   E (22271) task_wdt: Task watchdog got triggered. The following tasks/users did not reset the watchdog in time:
   E (22271) task_wdt:  - IDLE (CPU 0)
   E (22271) task_wdt: Tasks currently running:
   E (22271) task_wdt: CPU 0: Task2
   E (22271) task_wdt: CPU 1: IDLE
   E (22271) task_wdt: Print CPU 0 (current core) backtrace
   
   ```

2. 使用mutex信号量，可以看到任务1取得信号量后会保持信号量且一直有cpu使用权，期间任务3无法取得信号量，且任务2也在持续运行，导致看门狗触发。当任务1执行结束归还信号量后，任务3才取得信号量且执行任务，而任务1由于优先级低，将和IDLE一样由于任务2不阻塞自身而得不到执行。

   ``` bash
   I (0) cpu_start: Starting scheduler on APP CPU.        
   task3 begin!
   task2 begin!
   task1 begin!
   task1 take!
   task1 i=0
   task1 i=1
   task1 i=2
   task1 i=3
   task1 i=4
   task1 i=5
   E (12271) task_wdt: Task watchdog got triggered. The following tasks/users did not reset the watchdog in time:
   E (12271) task_wdt:  - IDLE (CPU 0)
   E (12271) task_wdt: Tasks currently running:
   E (12271) task_wdt: CPU 0: Task2
   E (12271) task_wdt: CPU 1: IDLE
   E (12271) task_wdt: Print CPU 0 (current core) backtrace
   
   task1 i=6
   task1 i=7
   task1 i=8
   task1 i=9
   task1 i=10
   E (17271) task_wdt: Task watchdog got triggered. The following tasks/users did not reset the watchdog in time:
   E (17271) task_wdt:  - IDLE (CPU 0)
   E (17271) task_wdt: Tasks currently running:
   E (17271) task_wdt: CPU 0: Task2
   E (17271) task_wdt: CPU 1: IDLE
   E (17271) task_wdt: Print CPU 0 (current core) backtrace
   
   task3 didn't take!
   task1 i=11
   task1 i=12
   task1 i=13
   task1 i=14
   E (22271) task_wdt: Task watchdog got triggered. The following tasks/users did not reset the watchdog in time:
   E (22271) task_wdt:  - IDLE (CPU 0)
   E (22271) task_wdt: Tasks currently running:
   E (22271) task_wdt: CPU 0: Task2
   E (22271) task_wdt: CPU 1: IDLE
   E (22271) task_wdt: Print CPU 0 (current core) backtrace
   
   task1 i=15
   task1 i=16
   task1 i=17
   task1 i=18
   task1 i=19
   E (27271) task_wdt: Task watchdog got triggered. The following tasks/users did not reset the watchdog in time:
   E (27271) task_wdt:  - IDLE (CPU 0)
   E (27271) task_wdt: Tasks currently running:
   E (27271) task_wdt: CPU 0: Task2
   E (27271) task_wdt: CPU 1: IDLE
   E (27271) task_wdt: Print CPU 0 (current core) backtrace
   
   task3 take!
   task3 i = 0
   task3 i = 1
   task3 i = 2
   task3 i = 3
   task3 i = 4
   E (32271) task_wdt: Task watchdog got triggered. The following tasks/users did not reset the watchdog in time:
   E (32271) task_wdt:  - IDLE (CPU 0)
   E (32271) task_wdt: Tasks currently running:
   E (32271) task_wdt: CPU 0: Task2
   E (32271) task_wdt: CPU 1: IDLE
   E (32271) task_wdt: Print CPU 0 (current core) backtrace
   
   task3 i = 5
   task3 i = 6
   task3 i = 7
   task3 i = 8
   task3 i = 9
   E (37271) task_wdt: Task watchdog got triggered. The following tasks/users did not reset the watchdog in time:
   E (37271) task_wdt:  - IDLE (CPU 0)
   E (37271) task_wdt: Tasks currently running:
   E (37271) task_wdt: CPU 0: Task2
   E (37271) task_wdt: CPU 1: IDLE
   E (37271) task_wdt: Print CPU 0 (current core) backtrace
   
   task3 give!
   E (42271) task_wdt: Task watchdog got triggered. The following tasks/users did not reset the watchdog in time:
   E (42271) task_wdt:  - IDLE (CPU 0)
   E (42271) task_wdt: Tasks currently running:
   E (42271) task_wdt: CPU 0: Task2
   E (42271) task_wdt: CPU 1: IDLE
   E (42271) task_wdt: Print CPU 0 (current core) backtrace
   
   task3 take!
   task3 i = 0
   task3 i = 1
   task3 i = 2
   task3 i = 3
   task3 i = 4
   E (47271) task_wdt: Task watchdog got triggered. The following tasks/users did not reset the watchdog in time:
   E (47271) task_wdt:  - IDLE (CPU 0)
   E (47271) task_wdt: Tasks currently running:
   E (47271) task_wdt: CPU 0: Task2
   E (47271) task_wdt: CPU 1: IDLE
   E (47271) task_wdt: Print CPU 0 (current core) backtrace
   
   task3 i = 5
   task3 i = 6
   task3 i = 7
   task3 i = 8
   task3 i = 9
   E (52271) task_wdt: Task watchdog got triggered. The following tasks/users did not reset the watchdog in time:
   E (52271) task_wdt:  - IDLE (CPU 0)
   E (52271) task_wdt: Tasks currently running:
   E (52271) task_wdt: CPU 0: Task2
   E (52271) task_wdt: CPU 1: IDLE
   E (52271) task_wdt: Print CPU 0 (current core) backtrace
   
   ...
   ```

   

### 代码：

``` c
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_spi_flash.h"

#include "freertos/semphr.h"

SemaphoreHandle_t mutexHandle;

void Task3(void *pvParam)
{
    BaseType_t iRet;
    printf("task3 begin!\n");
    vTaskDelay(pdMS_TO_TICKS(1000));

    while (1)
    {
        // 试图取得信号量
        iRet = xSemaphoreTake(mutexHandle, 1000);
        if (iRet == pdPASS)
        {
            // 取得成功，打印信息
            printf("task3 take!\n");
            for (int i = 0; i < 10; i++)
            {
                printf("task3 i = %d\n", i);
                vTaskDelay(pdMS_TO_TICKS(1000));
            }
            // 执行完毕，归还信号量
            xSemaphoreGive(mutexHandle);
            printf("task3 give!\n");
            vTaskDelay(pdMS_TO_TICKS(5000));
        }
        else
        {
            // 取得失败，输出信息，循环尝试取得信号量
            printf("task3 didn't take!\n");
            vTaskDelay(pdMS_TO_TICKS(1000));
        }
    }
}

void Task2(void *pvParam)
{
    printf("task2 begin!\n");
    // 阻塞1秒让task1有机会执行
    vTaskDelay(pdMS_TO_TICKS(1000));
    while (1)
    {
        // 进入task2之后，由于task2没有阻塞自身
        // 只有task3可以剥夺task2的执行
        // 而task1优先级更低，不能执行
        ;
    }
}

void Task1(void *pvParam)
{
    BaseType_t iRet;
    while (1)
    {
        // 任务1取得信号量之后，任务1为mutex的所有者
        printf("task1 begin!\n");
        iRet = xSemaphoreTake(mutexHandle, 1000);
        if (iRet == pdPASS)
        {
            printf("task1 take!\n");
            for (int i = 0; i < 20; i++)
            {
                printf("task1 i=%d\n", i);
                vTaskDelay(pdMS_TO_TICKS(1000));
            }
            // 执行完毕，归还信号量
            xSemaphoreGive(mutexHandle);
            printf("task1 give!\n");
            vTaskDelay(pdMS_TO_TICKS(5000));
        }
        else
        {
            printf("task1 didn't take\n");
            vTaskDelay(pdMS_TO_TICKS(1000));
        }
    }
}

void app_main(void)
{
    // 创建mutax
    mutexHandle = xSemaphoreCreateMutex();
    // 创建二进制信号量
    // mutexHandle = xSemaphoreCreateBinary();

    // 创建任务前挂起任务调度器
    vTaskSuspendAll();
    xTaskCreatePinnedToCore(Task1, "Task1", 1024 * 5, NULL, 1, NULL, 0);
    xTaskCreatePinnedToCore(Task2, "Task2", 1024 * 5, NULL, 2, NULL, 0);
    xTaskCreatePinnedToCore(Task3, "Task3", 1024 * 5, NULL, 3, NULL, 0);
    // 任务创建结束恢复任务调度器
    xTaskResumeAll();

}

```



## 递归互斥信号量



![图片来自B站Michael_ee老师的视频](https://fastly.jsdelivr.net/gh/vampire610/PicGo/Blog-PIC/20230810134747.png)

如图中所示，任务1需要互斥访问资源A，在资源A中又要互斥访问资源B，若不用递归互斥，就需要手动给A上锁解锁，手动给B上锁解锁，每个互斥锁需要一个互斥信号量，若有更多这种关系，就需要更多的信号量来实现。

而递归互斥量会递归的给后面的资源上锁。

### 运行结果

``` bash
-------------------
task1 begin!
task1 take A!
task1 i=0 for A
-------------------
task2 begin!
task1 i=1 for A
task1 i=2 for A
task1 i=3 for A
task1 i=4 for A
task1 i=5 for A
task1 i=6 for A
task1 i=7 for A
task1 i=8 for A
task1 i=9 for A
task1 take B!
task1 i=0 for B
task1 i=1 for B
task1 i=2 for B
task1 i=3 for B
task1 i=4 for B
task1 i=5 for B
task1 i=6 for B
task1 i=7 for B
task1 i=8 for B
task1 i=9 for B
task1 give B   此处可以看到释放B的锁的时候，任务2还是不能获取信号量
task1 give A   当任务1也释放A的锁之后，任务2可取得信号量
task2 take
task2 i=0
task2 i=1
task2 i=2
```



### 示例代码：

``` c
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_spi_flash.h"

#include "freertos/semphr.h"

SemaphoreHandle_t mutexHandle;

void Task1(void *pvParam)
{
    while (1)
    {
        printf("-------------------\n");
        printf("task1 begin!\n");
        // 取得A的锁
        xSemaphoreTakeRecursive(mutexHandle, portMAX_DELAY);

        printf("task1 take A!\n");

        for (int i = 0; i < 10; i++)
        {
            printf("task1 i=%d for A\n", i);
            vTaskDelay(pdMS_TO_TICKS(1000));
        }

        // 取得资源B 再次加锁
        xSemaphoreTakeRecursive(mutexHandle, portMAX_DELAY);

        printf("task1 take B!\n");

        for (int i = 0; i < 10; i++)
        {
            printf("task1 i=%d for B\n", i);
            vTaskDelay(pdMS_TO_TICKS(1000));
        }

        printf("task1 give B\n");
        // 释放B的锁
        xSemaphoreGiveRecursive(mutexHandle);
        vTaskDelay(pdMS_TO_TICKS(3000));

        printf("task1 give A\n");
        // 释放A的锁
        xSemaphoreGiveRecursive(mutexHandle);
        vTaskDelay(pdMS_TO_TICKS(3000));
    }
}

void Task2(void *pvParam)
{
    // 尝试获得递归互斥量
    vTaskDelay(pdMS_TO_TICKS(1000));

    while (1)
    {
        printf("-------------------\n");
        printf("task2 begin!\n");
        xSemaphoreTakeRecursive(mutexHandle, portMAX_DELAY);
        printf("task2 take\n");
        for (int i = 0; i < 10; i++)
        {
            printf("task2 i=%d\n", i);
            vTaskDelay(pdMS_TO_TICKS(1000));
        }
        printf("task2 give\n");
        xSemaphoreGiveRecursive(mutexHandle);
        vTaskDelay(pdMS_TO_TICKS(3000));
    }
}

void app_main(void)
{
    // 创建mutax
    mutexHandle = xSemaphoreCreateRecursiveMutex();

    // 创建任务前挂起任务调度器
    vTaskSuspendAll();

    xTaskCreatePinnedToCore(Task1, "Task1", 1024 * 5, NULL, 1, NULL, 0);
    xTaskCreatePinnedToCore(Task2, "Task2", 1024 * 5, NULL, 1, NULL, 0);

    // 任务创建结束恢复任务调度器
    xTaskResumeAll();

}

```
