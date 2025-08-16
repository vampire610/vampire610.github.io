---
title: ESP-IDF与freeRTOS(三) 队列扩展
date: 2023-08-03 09:14:41
tags:
  - ESP-IDF
  - freeRTOS
categories:
  - 学习笔记
---

# ESP-IDF与freeRTOS(三) 队列扩展

书接上回

## 队列多进单出模型

与单进单出基本没区别，主要注意任务优先级分配，读取时可以设置如下

``` c
xStatus = xQueueReceive(QHandle, &x, portMAX_DELAY);
```

其中最后一个参数会使任务阻塞，等待队列中数据到来

## 队列集合

`QueueSetHandle_t xQueueCreateSet( const UBaseType_t uxEventQueueLength );`

代码示例

``` c
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_spi_flash.h"

#include "freertos/queue.h"

#include "myled.h"

typedef struct A_STRUCT
{
    char id;
    char data;
} xStruct;

// 输入参数定义为void类型指针，可强转其他任何类型参数传入，在函数内再转换回去
void sendTask1(void *pvParam)
{
    int i = 111;

    QueueHandle_t QHandle;
    QHandle = (QueueHandle_t)pvParam;

    BaseType_t xStatus;

    while (1)
    {
        xStatus = xQueueSend(QHandle, &i, 0);
        if (xStatus != pdPASS)
        {
            // 发送失败
            printf("send fail!\n");
        }
        else
        {
            printf("send done!\n");
        }
        vTaskDelay(1000 / portTICK_PERIOD_MS);
    }
}

void sendTask2(void *pvParam)
{
    int i = 222;

    QueueHandle_t QHandle;
    QHandle = (QueueHandle_t)pvParam;

    BaseType_t xStatus;

    while (1)
    {
        xStatus = xQueueSend(QHandle, &i, 0);
        if (xStatus != pdPASS)
        {
            // 发送失败
            printf("send fail!\n");
        }
        else
        {
            printf("send done!\n");
        }
        vTaskDelay(1000 / portTICK_PERIOD_MS);
    }
}

void recTask(void *pvParam)
{
    // 创建QueueSet句柄，将入口参数转为此类型
    QueueSetHandle_t QueueSet;
    QueueSet = (QueueSetHandle_t)pvParam;

    QueueSetMemberHandle_t QueueData;
    BaseType_t xStatus;
    int x = 0;
    while (1)
    {
        // 取出有数据的队列
        QueueData = xQueueSelectFromSet(QueueSet, portMAX_DELAY);
        // 从有数据的队列取出值
        xStatus = xQueueReceive(QueueData, &x, portMAX_DELAY);
        if (xStatus != pdPASS)
        {
            // 发送失败
            printf("rec fail!\n");
        }
        else
        {
            printf("rec: %d\r\n", x);
        }
    }
}

void app_main(void)
{
    led_init();

    xTaskCreatePinnedToCore(ledTask, "ledTask", 1024, NULL, 1, NULL, 1);

    // 创建队列句柄
    QueueHandle_t QHandle1;
    QueueHandle_t QHandle2;
    // 创建队列，参数为队列长度、Item大小
    QHandle1 = xQueueCreate(5, sizeof(int));
    QHandle2 = xQueueCreate(5, sizeof(int));

    QueueSetHandle_t QueueSet;

    // 创建QueueSet，长度为要接入集合的队列的长度和
    QueueSet = xQueueCreateSet(10);

    // 将队列加入集合
    xQueueAddToSet(QHandle1, QueueSet);
    xQueueAddToSet(QHandle2, QueueSet);

    // 判断是否创建成功
    if ((QHandle1 != NULL) && (QHandle2 != NULL) && (QueueSet != NULL))
    {

        // 接收任务优先级更高时，一旦有数据就会被接收
        printf("Creat queue successfully!\n");
        xTaskCreatePinnedToCore(sendTask1, "sendTask1", 1024 * 5, (void *)QHandle1, 1, NULL, 0);
        xTaskCreatePinnedToCore(sendTask2, "sendTask2", 1024 * 5, (void *)QHandle2, 1, NULL, 0);
        xTaskCreatePinnedToCore(recTask, "recTask", 1024 * 5, (void *)QueueSet, 2, NULL, 0);
    }
    else
    {
        printf("Can't creat queue successfully!\n");
    }
}

```

## 队列邮箱(单发多收)

队列和邮箱都用于任务、中断等传递数据，但队列传递时取出即删除数据，数据将不会保持。邮箱传递数据时数据会被保持，取出数据不会删除数据，直到数据被覆盖。（以下文本来自FreeRTOS官方文档）

> - A queue is used to send data from one task to another task, or from an interrupt
>   service routine to a task. The sender places an item in the queue, and the receiver
>   removes the item from the queue. The data passes through the queue from the sender
>   to the receiver.
> - A mailbox is used to hold data that can be read by any task, or any interrupt service
>   routine. The data does not pass through the mailbox, but instead remains in the
>   mailbox until it is overwritten. The sender overwrites the value in the mailbox. The
>   receiver reads the value from the mailbox, but does not remove the value from the
>   mailbox.

示例代码:

``` c
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_spi_flash.h"

#include "freertos/queue.h"

#include "myled.h"

typedef struct A_STRUCT
{
    char id;
    char data;
} xStruct;

// 输入参数定义为void类型指针，可强转其他任何类型参数传入，在函数内再转换回去
void writeTask1(void *pvParam)
{
    int i = 0;

    QueueHandle_t Mailbox;
    Mailbox = (QueueHandle_t)pvParam;

    BaseType_t xStatus;

    while (1)
    {
        // mailbox写入使用overwrite
        xStatus = xQueueOverwrite(Mailbox, &i);
        if (xStatus != pdPASS)
        {
            // 发送失败
            printf("send fail!\n");
        }
        else
        {
            printf("send done!\n");
        }
        i++;
        vTaskDelay(6000 / portTICK_PERIOD_MS);
    }
}

void readTask(void *pvParam)
{
    QueueHandle_t Mailbox;
    Mailbox = (QueueHandle_t)pvParam;

    BaseType_t xStatus;
    int i = 0;
    while (1)
    {
        // 从队列读出值，但不删除，无数据则等待
        // 但Peek读数不会删除队列数据，会导致读进程一直运行不会停止
        xStatus = xQueuePeek(Mailbox, &i, portMAX_DELAY);
        if (xStatus != pdPASS)
        {
            // 发送失败
            printf("read fail!\n");
        }
        else
        {
            printf("read i = %d done!\n", i);
        }
        vTaskDelay(1000 / portTICK_PERIOD_MS);
    }
}

void app_main(void)
{
    led_init();

    xTaskCreatePinnedToCore(ledTask, "ledTask", 1024, NULL, 1, NULL, 1);

    // 创建队列句柄
    QueueHandle_t Mailbox;
    Mailbox = xQueueCreate(1, sizeof(int));

    // 判断是否创建成功
    if (Mailbox != NULL)
    {
        // 接收任务优先级更高时，一旦有数据就会被接收
        printf("Creat queue successfully!\n");
        // 此处可用一个主体函数创建3个readTask
        xTaskCreatePinnedToCore(writeTask1, "writeTask1", 1024 * 5, (void *)Mailbox, 1, NULL, 0);
        xTaskCreatePinnedToCore(readTask, "readTask1", 1024 * 5, (void *)Mailbox, 2, NULL, 0);
        xTaskCreatePinnedToCore(readTask, "readTask2", 1024 * 5, (void *)Mailbox, 2, NULL, 0);
        xTaskCreatePinnedToCore(readTask, "readTask3", 1024 * 5, (void *)Mailbox, 2, NULL, 0);
    }
    else
    {
        printf("Can't creat queue successfully!\n");
    }
}

```

