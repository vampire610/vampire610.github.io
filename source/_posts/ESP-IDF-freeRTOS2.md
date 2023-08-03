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


## 队列传递数据

RTOS队列需要添加头文件`freertos/queue.h`

常用函数：
``` c
//Creates a new queue and returns a handle by which the queue can be referenced.
//创建队列
QueueHandle_t xQueueCreate( UBaseType_t uxQueueLength,UBaseType_t uxItemSize );

//创建队列集合
QueueSetHandle_t xQueueCreateSet( const UBaseType_t uxEventQueueLength );

//创建静态队列
QueueHandle_t xQueueCreateStatic(UBaseType_t uxQueueLength,
                                 UBaseType_t uxItemSize,
                                 uint8_t *pucQueueStorageBuffer,
                                 StaticQueue_t *pxQueueBuffer);

//删除队列
void vQueueDelete( TaskHandle_t pxQueueToDelete );
```

更多请查阅FreeRTOS参考手册。

### 整型队列示例

``` c
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_spi_flash.h"

#include "freertos/queue.h"

// 输入参数定义为void类型指针，可强转其他任何类型参数传入，在函数内再转换回去
void sendTask1(void *pvParam)
{
    //创建队列句柄，将传入void类型队列指针转为队列句柄类型指针
    QueueHandle_t QHandle;
    QHandle = (QueueHandle_t)pvParam;

    BaseType_t xStatus;

    int i = 0;
    while (1)
    {
        //发送i的值到队列QHandle，超时等待设置为0
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
        i++;
        if (i == 8)
        {
            i = 0;
        }
        //设置延时1s发送一次
        vTaskDelay(1000 / portTICK_PERIOD_MS);
    }
}

void recTask(void *pvParam)
{
    QueueHandle_t QHandle;
    QHandle = (QueueHandle_t)pvParam;

    BaseType_t xStatus;
    int x = 0;
    while (1)
    {
        //判断队列是否为空
        if (uxQueueMessagesWaiting(QHandle) != 0)
        {
            // 队列非空
            //从QHandle接收数据到x，超时等待为0
            xStatus = xQueueReceive(QHandle, &x, 0);
            if (xStatus != pdPASS)
            {
                // 发送失败
                printf("rec fail!\n");
            }
            else
            {
                printf("rec x = %d!\n", x);
            }
        }
        else
        {
            printf("No data!\n");
        }
		//设置延时200ms读取一次
        vTaskDelay(200 / portTICK_PERIOD_MS);
    }
}

void app_main(void)
{

    TaskHandle_t pxTask1 = NULL;
    TaskHandle_t pxTask2 = NULL;

    // 创建队列句柄
    QueueHandle_t QHandle;
    // 创建队列，参数为队列长度、Item大小
    // 此处设置队列存储int型变量，用sizeof取得大小
    QHandle = xQueueCreate(5, sizeof(int));
    // 判断是否创建成功
    if (QHandle != NULL)
    {
        printf("Creat queue successfully!\n");
        xTaskCreatePinnedToCore(sendTask1, "sendTask1", 1024 * 5, (void *)QHandle, 1, NULL, 0);
        xTaskCreatePinnedToCore(recTask, "recTask", 1024 * 5, (void *)QHandle, 1, NULL, 0);
    }
    else
    {
        printf("Can't creat queue successfully!\n");
    }
}

```

示例输出

``` bash
...
No data!
No data!
No data!
No data!
send done!
rec x = 0!
No data!
No data!
No data!
No data!
send done!
rec x = 1!
No data!
No data!
No data!
No data!
send done!
rec x = 2!
No data!
No data!
No data!
No data!
send done!
rec x = 3!
No data!
No data!
No data!
No data!
send done!
rec x = 4!
No data!
No data!
No data!
...
```

### 结构体队列

示例：

``` c
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_spi_flash.h"

#include "freertos/queue.h"

typedef struct A_STRUCT
{
    char id;
    char data;
}xStruct;

// 输入参数定义为void类型指针，可强转其他任何类型参数传入，在函数内再转换回去
void sendTask1(void *pvParam)
{
    QueueHandle_t QHandle;
    QHandle = (QueueHandle_t)pvParam;

    BaseType_t xStatus;

    //定义结构体并赋初值
    xStruct xUSB = {1,55};
    while (1)
    {
        xStatus = xQueueSend(QHandle, &xUSB, 0);
        if (xStatus != pdPASS)
        {
            // 发送失败
            printf("send fail!\n");
        }
        else
        {
            printf("send done!\n");
        }
        xUSB.id++;
        if (xUSB.id == 8)
        {
            xUSB.id = 0;
        }
        vTaskDelay(1000 / portTICK_PERIOD_MS);
    }
}

void recTask(void *pvParam)
{
    QueueHandle_t QHandle;
    QHandle = (QueueHandle_t)pvParam;

    BaseType_t xStatus;
    
    xStruct xUSB;
    while (1)
    {
        // 判断队列是否为空
        if (uxQueueMessagesWaiting(QHandle) != 0)
        {
            // 队列非空
            xStatus = xQueueReceive(QHandle, &xUSB, 0);
            if (xStatus != pdPASS)
            {
                // 发送失败
                printf("rec fail!\n");
            }
            else
            {
                printf("rec id = %d, data = %d!\n", xUSB.id,xUSB.data);
            }
        }
        else
        {
            printf("No data!\n");
        }

        vTaskDelay(500 / portTICK_PERIOD_MS);
    }
}

void app_main(void)
{
    xTaskCreatePinnedToCore(ledTask, "ledTask", 1024, NULL, 1, NULL, 1);
    // 创建队列句柄
    QueueHandle_t QHandle;
    // 创建队列，参数为队列长度、Item大小
    // 此处设置队列存储结构体变量，用sizeof取得大小
    QHandle = xQueueCreate(5, sizeof(xStruct));
    // 判断是否创建成功
    if (QHandle != NULL)
    {
        printf("Creat queue successfully!\n");
        xTaskCreatePinnedToCore(sendTask1, "sendTask1", 1024 * 5, (void *)QHandle, 1, NULL, 0);
        xTaskCreatePinnedToCore(recTask, "recTask", 1024 * 5, (void *)QHandle, 1, NULL, 0);
    }
    else
    {
        printf("Can't creat queue successfully!\n");
    }
}

```

运行结果：

``` bash
...
No data!
send done!
rec id = 0, data = 55!
No data!
send done!
rec id = 1, data = 55!
No data!
send done!
rec id = 2, data = 55!
No data!
send done!
rec id = 3, data = 55!
No data!
send done!
rec id = 4, data = 55!
No data!
send done!
rec id = 5, data = 55!
...
```

### 指针传递（需要特别小心）

注意创建与释放，指针越界

需要注意，示例中队列传递为指针类型，代码中创建了指针来储存字符串，即为字符串数组，在传递时对数组名取地址，即为字符串的指针。

示例

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
    char *pStrToSend;
    // char型可不用sizeof，直接填50，此处保持好习惯，避免写其他类型时分配错误
    pStrToSend = (char *)malloc(sizeof(char) * 50);
    int i = 0;

    QueueHandle_t QHandle;
    QHandle = (QueueHandle_t)pvParam;

    BaseType_t xStatus;

    while (1)
    {
        //循环里分配内存，每一个数一个内存空间，需要在使用完后及时释放
        // char型可不用sizeof，直接填50，此处保持好习惯，避免写其他类型时分配错误
        pStrToSend = (char *)malloc(sizeof(char) * 50);
        // sprintf不会限制长度，可能会导致越界
        // snprintf格式化输出到buf，并限制长度
        // 如此处限制长度50，则最多输出49个，最后一个会置为'\0'
        snprintf(pStrToSend, 50, "Today is a good %d day!\r\n", i);
        i++;
        // 此处传递为字符串名的地址，字符串指针的地址
        xStatus = xQueueSend(QHandle, &pStrToSend, 0);
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
    QueueHandle_t QHandle;
    QHandle = (QueueHandle_t)pvParam;

    char *pStrToRec;
    // pStrToRec = (char *)malloc(sizeof(char)*50);

    BaseType_t xStatus;

    while (1)
    {
        // 判断队列是否为空
        if (uxQueueMessagesWaiting(QHandle) != 0)
        {
            // 队列非空
            // 此处传递为字符串名的地址，字符串指针的地址
            xStatus = xQueueReceive(QHandle, &pStrToRec, 0);
            if (xStatus != pdPASS)
            {
                // 发送失败
                printf("rec fail!\n");
            }
            else
            {
                printf("rec: %s\r\n", pStrToRec);
            }
            //使用完传递的数据，将内存及时释放
            //由于传递数据为指针
            //实际上发送函数和接收函数指针指向为同一个内存空间
            //释放接收函数指向内存即可
            free(pStrToRec);
        }
        else
        {
            printf("No data!\n");
        }

        vTaskDelay(500 / portTICK_PERIOD_MS);
    }
}

void app_main(void)
{
    led_init();

    xTaskCreatePinnedToCore(ledTask, "ledTask", 1024, NULL, 1, NULL, 1);

    // 创建队列句柄
    QueueHandle_t QHandle;
    // 创建队列，参数为队列长度、Item大小
    // 此处设置队列存储指针类型变量，用sizeof取得指针宽度大小
    QHandle = xQueueCreate(5, sizeof(char *));
    // 判断是否创建成功
    if (QHandle != NULL)
    {
        printf("Creat queue successfully!\n");
        xTaskCreatePinnedToCore(sendTask1, "sendTask1", 1024 * 5, (void *)QHandle, 1, NULL, 0);
        xTaskCreatePinnedToCore(recTask, "recTask", 1024 * 5, (void *)QHandle, 1, NULL, 0);
    }
    else
    {
        printf("Can't creat queue successfully!\n");
    }
}

```

运行结果

``` bash
...
Creat queue successfully!
send done!
rec: Today is a good 0 day!

No data!
send done!
rec: Today is a good 1 day!

No data!
send done!
rec: Today is a good 2 day!

No data!
send done!
rec: Today is a good 3 day!

No data!
send done!
rec: Today is a good 4 day!

No data!
send done!
rec: Today is a good 5 day!
...
```





