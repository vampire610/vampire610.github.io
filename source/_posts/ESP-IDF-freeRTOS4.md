---
title: ESP-IDF与freeRTOS(四)
date: 2023-08-04 14:58:49
tags:
  - ESP-IDF
  - freeRTOS
categories:
  - 学习笔记
---

# ESP-IDF与freeRTOS(四)

## 软件定时器

FreeRTOS的软件定时器与硬件无关，没有数量限制，定时器到时间触发任务。

``` c
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_spi_flash.h"

#include "freertos/timers.h"

#include "myled.h"

void TimerCallBack(TimerHandle_t xTimer)
{
    const char *strName;
    //取得名字后可在同一个回调函数执行不同的函数过程
    strName = pcTimerGetName(xTimer);
    printf("timer name = %s!\n",strName);
    //取得ID的返回值是返回指针
    //创建定时器的时候传入ID是数字的时候可用如下方法
    //若传入的是地址，如(void*)&id1,则此处取值时获取到的是地址
    int id;
    id = (uint32_t)pvTimerGetTimerID(xTimer);
    printf("timer ID = %d!\n",id);
}

void app_main(void)
{
    TimerHandle_t xTimer1;
    TimerHandle_t xTimer2;

    // 创建timer1，1秒后执行，选择是否自动重装，ID 0，回调函数自定为Timer1CallBack
    xTimer1 = xTimerCreate("Timer1", pdMS_TO_TICKS(1000), pdTRUE, (void *)0, TimerCallBack);
    xTimer2 = xTimerCreate("Timer2", pdMS_TO_TICKS(2000), pdTRUE, (void *)1, TimerCallBack);
    // 启动定时器
    xTimerStart(xTimer1, 0);
    xTimerStart(xTimer2, 0);
    printf("启动定时器1\n");

    //改变timer定时周期
    xTimerChangePeriod(xTimer1,pdMS_TO_TICKS(4000),0);

    // vTaskDelay(pdMS_TO_TICKS(3000));
    // //停止定时器
    // xTimerStop(xTimer1,0);

    // while(1)
    // {
    //     vTaskDelay(pdMS_TO_TICKS(1000));
    //     //重启timer2，让timer2从零计时
    //     xTimerReset(xTimer2,0);
    // }


    // xTaskCreatePinnedToCore(ledTask, "ledTask", 2048, NULL, 1, NULL, 1);
}

```

到此为止，RTOS简单操作基本结束，下一步开始要学习信号量等内容，任务艰巨。
