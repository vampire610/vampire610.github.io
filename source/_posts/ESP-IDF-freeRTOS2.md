---
title: ESP-IDF与freeRTOS(二)
date: 2023-07-25 21:14:34
tags:
  - ESP-IDF
  - freeRTOS
categories:
  - 学习笔记
---

# ESP-IDF与freeRTOS(二)

书接上回


## Task 看门狗

ESP32中，看门狗分为中断看门狗和Task看门狗，可参考乐鑫官网[Watchdogs - ESP32 - — ESP-IDF 编程指南 latest 文档 (espressif.com)](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32/api-reference/system/wdts.html)。

看门狗相关设置可在menuconfig中修改。

测试代码如下：

``` c
void myTask1(void *pvParam)
{
    while (1)
    {
        // printf("task1\n");
        // vTaskDelay(1000 / portTICK_PERIOD_MS);
        ;
    }
}
```

串口输出信息：

``` bash
E (10291) task_wdt: Task watchdog got triggered. The following tasks/users did not reset the watchdog in time:
E (10291) task_wdt:  - IDLE (CPU 0) //看门狗监控IDLE(CPU0)
E (10291) task_wdt: Tasks currently running:
E (10291) task_wdt: CPU 0: myTask1
E (10291) task_wdt: CPU 1: IDLE
E (10291) task_wdt: Print CPU 0 (current core) backtrace
```

由以上信息可见，Task看门狗被触发，触发时CPU0在运行myTask1，CPU1在运行IDLE，看门狗监控IDLE（CPU0）。

myTask1中没有阻塞函数，类似`vTaskDelay`等，会导致任务一直占用CPU0运行，且优先级高于CPU0的IDLE任务，导致IDLE饥饿，触发看门狗。

解决办法：

>  task1中加入阻塞操作函数，让低优先级任务得以运行；
>
> 将task1优先级降值0，与IDLE相同，此时同优先级任务将都被分配时间片运行。

### 使用看门狗监控自己的任务

- 添加头文件 `esp_task_wdt.h`

``` c
void myTask2(void * pvParam)
{
    //添加任务到看门狗监控
    esp_task_wdt_add(NULL);
    while(1)
    {
        printf("task2 running\n");
        //喂狗，在menuconfig中设置任务看门狗时间
        esp_task_wdt_reset();
        vTaskDelay(1000 / portTICK_PERIOD_MS);
    }
}
```

如果不在规定时间内喂狗，将触发看门狗。