---
title: ESP-IDF与freeRTOS(九) 数据流缓冲区
date: 2023-08-16 10:53:39
tags:
  - ESP-IDF
  - freeRTOS
categories:
  - 学习笔记
---

# ESP-IDF与freeRTOS(九) 数据流缓冲区

## 流数据缓冲区

### 运行结果1

由于设置触发长度为10，而每次发送长度为25，所以每次发送都能触发接收

``` bash
-----------------
Send: str_len = 25, send_byte = 25
-------------
Receive: rec_bytes = 25, data = Hello, I am vampire610 1       

-----------------
Send: str_len = 25, send_byte = 25
-------------
Receive: rec_bytes = 25, data = Hello, I am vampire610 2       

-----------------
Send: str_len = 25, send_byte = 25
-------------
Receive: rec_bytes = 25, data = Hello, I am vampire610 3 
```

### 运行结果2

此处可以看到第一次发送后接收成功了，而后续需要满100个数据才会触发接收。

这是因为当buffer中不为空时，接收函数会先接收再进入阻塞，然后等待满100触发。

可以将task1中的delay提到前面，保持buffer为空，让task2先因为不满100而阻塞，再让task1向buffer发送数据。

``` bash
-----------------
Send: str_len = 24, send_byte = 24
-------------
Receive: rec_bytes = 24, data = Hello, I am vampire610 1       
-----------------
Send: str_len = 24, send_byte = 24
-----------------
Send: str_len = 24, send_byte = 24
-----------------
Send: str_len = 24, send_byte = 24
-----------------
Send: str_len = 24, send_byte = 24
-----------------
Send: str_len = 24, send_byte = 24
-------------
Receive: rec_bytes = 50, data = Hello, I am vampire610 2Hello, I am vampire610 3He�?
-------------
Receive: rec_bytes = 50, data = llo, I am vampire610 4Hello, I am vampire610 5Hell�?
-------------
Receive: rec_bytes = 20, data = o, I am vampire610 6
-----------------
Send: str_len = 24, send_byte = 24
-------------
```

### 运行结果3

按上述方法修改delay位置后，可以看到第一次接收消失了，task2一开始就就进入阻塞，需要等待stream buffer满100个才能接收。

``` bash
-----------------
Send: str_len = 24, send_byte = 24
-----------------
Send: str_len = 24, send_byte = 24
-----------------
Send: str_len = 24, send_byte = 24
-----------------
Send: str_len = 24, send_byte = 24
-----------------
Send: str_len = 24, send_byte = 24
-------------
Receive: rec_bytes = 50, data = Hello, I am vampire610 1Hello, I am vampire610 2He�?
-------------
Receive: rec_bytes = 50, data = llo, I am vampire610 3Hello, I am vampire610 4Hell�?
-------------
Receive: rec_bytes = 20, data = o, I am vampire610 5
-----------------
```



### 示例代码

``` c
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_spi_flash.h"

#include "freertos/stream_buffer.h"
#include "string.h"
StreamBufferHandle_t StreamBufferHandle = NULL;

void Task1(void *pvParam)
{
    int i = 0;
    int buf_len = 0;
    int send_bytes = 0;
    char tx_buf[50];
    while (1)
    {
        i++;
        // 向tx_buf填值，并返回长度
        buf_len = sprintf(tx_buf, "Hello, I am vampire610 %d\n", i);
        // 发送buf，并返回成功发送长度
        send_bytes = xStreamBufferSend(StreamBufferHandle, // 句柄
                                       (void *)tx_buf,     // 发送的buf，需要强转位void指针
                                       buf_len,            // buf长度
                                       portMAX_DELAY);     // 阻塞时间

        printf("-----------------\n");
        printf("Send: str_len = %d, send_byte = %d\n", buf_len, send_bytes);

        vTaskDelay(pdMS_TO_TICKS(3000));
    }
}

void Task2(void *pvParam)
{
    char rx_buf[50];
    int rec_bytes = 0;
    while (1)
    {
        // 清空接收数组
        memset(rx_buf, 0, 50);
        // 接收streambuffer
        rec_bytes = xStreamBufferReceive(StreamBufferHandle, // 句柄
                                         (void *)rx_buf,     // 接收缓冲区，强转void指针
                                         sizeof(rx_buf),     // sizeof取得缓冲区大小
                                         portMAX_DELAY);     // 阻塞时间

        printf("-------------\n");
        printf("Receive: rec_bytes = %d, data = %s\n", rec_bytes, rx_buf);

        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}

void app_main(void)
{

    // 创建streambuffer，两个参数，buffer大小，触发大小
    StreamBufferHandle = xStreamBufferCreate(1000, 10);
    if (StreamBufferHandle != NULL)
    {
        vTaskSuspendAll();

        // 创建Task1时需要传出Task1的句柄
        xTaskCreatePinnedToCore(Task1, "Task1", 1024 * 5, NULL, 1, NULL, 0);
        xTaskCreatePinnedToCore(Task2, "Task2", 1024 * 5, NULL, 1, NULL, 0);

        // 任务创建结束恢复任务调度器
        xTaskResumeAll();
    }
    else
    {
        printf("Fail to creat stream buffer\n");
    }
}

```

## 如何确定流数据缓冲区大小

太大浪费资源，太小不能正常工作。引入task3检测缓冲区空闲空间大小变化情况。

初始设置为比较大的值，再通过检测情况进行调整，这与之前确定任务的堆栈大小思路一致。

流数据实际使用场景一般产生数据的速率不固定，所以需要稍长的时间来检测空闲空间。

### 运行示例

运行一段时间后，就可以看到当前缓冲区剩余空间大概是多少，最小剩余多少，即可根据该值修改缓冲区容量，来避免过分浪费。

``` bash
-------------
buf_space = 1000,min_space = 1000
-------------
buf_space = 1000,min_space = 1000
-----------------
Send: str_len = 24, send_byte = 24        
-------------
buf_space = 976,min_space = 976
-----------------
Send: str_len = 24, send_byte = 24        
-------------
buf_space = 952,min_space = 952
-------------
buf_space = 952,min_space = 952
-----------------
Send: str_len = 24, send_byte = 24        
-------------
buf_space = 928,min_space = 928
-----------------
Send: str_len = 24, send_byte = 24        
-------------
buf_space = 904,min_space = 904
-------------
buf_space = 904,min_space = 904
-----------------
Send: str_len = 24, send_byte = 24        
-------------
Receive: rec_bytes = 50, data = Hello, I am vampire610 1Hello, I am vampire610 2He�?
-------------
Receive: rec_bytes = 50, data = llo, I am vampire610 3Hello, I am vampire610 4Hell�?
-------------
buf_space = 980,min_space = 904
-------------
Receive: rec_bytes = 20, data = o, I am vampire610 5
-----------------
Send: str_len = 24, send_byte = 24        
-------------
Receive: rec_bytes = 24, data = Hello, I am vampire610 6
-------------
buf_space = 1000,min_space = 904
-------------
buf_space = 1000,min_space = 904
-----------------
```

### 示例代码

``` c
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_spi_flash.h"

#include "freertos/stream_buffer.h"
#include "string.h"
StreamBufferHandle_t StreamBufferHandle = NULL;

void Task1(void *pvParam)
{
    int i = 0;
    int buf_len = 0;
    int send_bytes = 0;
    char tx_buf[50];
    while (1)
    {
        vTaskDelay(pdMS_TO_TICKS(3000));
        i++;
        // 向tx_buf填值，并返回长度
        buf_len = sprintf(tx_buf, "Hello, I am vampire610 %d", i);
        // 发送buf，并返回成功发送长度
        send_bytes = xStreamBufferSend(StreamBufferHandle, // 句柄
                                       (void *)tx_buf,     // 发送的buf，需要强转位void指针
                                       buf_len,            // buf长度
                                       portMAX_DELAY);     // 阻塞时间

        printf("-----------------\n");
        printf("Send: str_len = %d, send_byte = %d\n", buf_len, send_bytes);
    }
}

void Task2(void *pvParam)
{
    char rx_buf[50];
    int rec_bytes = 0;
    while (1)
    {
        // 清空接收数组
        memset(rx_buf, 0, 50);
        // 接收streambuffer
        rec_bytes = xStreamBufferReceive(StreamBufferHandle, // 句柄
                                         (void *)rx_buf,     // 接收缓冲区，强转void指针
                                         sizeof(rx_buf),     // sizeof取得缓冲区大小
                                         portMAX_DELAY);     // 阻塞时间

        printf("-------------\n");
        printf("Receive: rec_bytes = %d, data = %s\n", rec_bytes, rx_buf);

        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}

void Task3(void *pvParam)
{
    int buf_space = 0;
    int min_space = 1000;
    while (1)
    {
        buf_space = xStreamBufferSpacesAvailable(StreamBufferHandle);
        if(buf_space<min_space)
        {
            //取得最小值
            min_space = buf_space;
        }
        printf("-------------\n");
        printf("buf_space = %d,min_space = %d\n", buf_space,min_space);
        //探查最小剩余空间时，这段代码运行频率要高于数据放入buffer的频率
        vTaskDelay(pdMS_TO_TICKS(2000));
    }
}

void app_main(void)
{

    // 创建streambuffer，两个参数，buffer大小，触发大小
    StreamBufferHandle = xStreamBufferCreate(1000, 100);
    if (StreamBufferHandle != NULL)
    {
        vTaskSuspendAll();

        // 创建Task1时需要传出Task1的句柄
        xTaskCreatePinnedToCore(Task1, "Task1", 1024 * 5, NULL, 1, NULL, 0);
        xTaskCreatePinnedToCore(Task2, "Task2", 1024 * 5, NULL, 1, NULL, 0);
        xTaskCreatePinnedToCore(Task3, "Task3", 1024 * 5, NULL, 1, NULL, 0);

        // 任务创建结束恢复任务调度器
        xTaskResumeAll();
    }
    else
    {
        printf("Fail to creat stream buffer\n");
    }
}

```

x
