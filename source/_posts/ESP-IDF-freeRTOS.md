---
title: ESP-IDF与freeRTOS
date: 2023-07-23 00:45:58
tags:
  - ESP-IDF
  - freeRTOS
categories:
  - 学习笔记
---

# ESP-IDF与freeRTOS

ESP-IDF是乐鑫官方推出的ESP32开发环境，个人不太喜欢arduino，所以选择学习使用IDF编程，上手发现很多地方十分陌生，在B站发现宝藏up[**Michael_ee**]([Michael_ee的个人空间_哔哩哔哩_bilibili](https://space.bilibili.com/1338335828))讲的很详细，于是跟着学习并简单记录。

## 预备工作，安装vscode和IDF插件，并配置

安装过程很多教程，也不复杂，就不再写了。

安装之后需要选择开发板，随后在界面左下点击齿轮图标进入menucofig设置，在其中设置芯片flash大小，算法，主频，flash结束是否自动运行等内容。

## Task创建与删除

首先需要写一个任务（就是一个函数）,任务如果需要持续执行，需要在任务中设置无尽循环，否则顺序执行完任务就直接退出了。

例如：

``` c
// 输入参数定义为void类型指针，可强转其他任何类型参数传入，在函数内再转换回去
void myTask(void* pvParam)
{
    while (1)
    {
        printf("Hello world! In the task!\n");
        vTaskDelay(1000 / portTICK_PERIOD_MS);
    }
}
```

### 任务创建

建立好任务函数之后，可以使用`xTaskCreate()`来在OS中创建任务。

``` c
xTaskCreate(myTask, "myTask1", 1024, NULL, 1, NULL);
```

函数参数具体描述可自己建立工程跳转查看，这里只做简介。

1. *pvTaskCode*：这里填入*myTask*，就是任务函数指针。

2. *pcName*：这里填入*”myTask1“*，代表在RTOS中的任务名称，调试等时候用。
3. *usStackDepth*：这里填入*1024*，任务栈大小，暂定1024，多任务时需要计算分配。
4. *pvParameters*：传入任务的参数，这里填入内容将会传递给*myTask*函数的参数部分，此处没有参数，填写*NULL*即可。
5. *uxPriority*：任务优先级，idle任务为0，最低，最高255。
6. *pxCreatedTask*：任务句柄，一个结构体，可配置任务相关信息，此处不使用，填*NULL*。

这样一个任务*mytask1*就创建好了，运行代码即可看到效果。

也可通过任务句柄来设置一些任务属性:

``` c
TaskHandle_t myHandle = NULL;
xTaskCreate(myTask, "myTask1", 1024, NULL, 1, &myHandle);
```

这样创建的任务就会使用句柄中定义的任务属性。

此处`myHandle`定义时为NULL，则创建任务时会将参数放入。

### 任务删除

如果使用了任务句柄，使用`*vTaskDelete(myHandle);*` 即可删除与`myHandle`相关的任务。

``` c
//定义句柄
TaskHandle_t myHandle = NULL;
//创建任务
xTaskCreate(myTask, "myTask1", 1024, NULL, 1, NULL);
//延时一段时间后删除任务
vTaskDelay(5000 / portTICK_PERIOD_MS);
//判断句柄是否为空，不为空则删除任务
if (myHandle != NULL)
{
    vTaskDelete(myHandle);
}
```

也可以不使用句柄，在任务函数中删除任务。

``` c
void myTask(void* pvParam)
{   
	printf("Hello world! In the task!\n");
    vTaskDelay(1000 / portTICK_PERIOD_MS);
    printf("Hello world! In the task!\n");
    vTaskDelay(1000 / portTICK_PERIOD_MS);
    printf("Hello world! In the task!\n");
    vTaskDelay(1000 / portTICK_PERIOD_MS);
	// vTaskDelete函数参数为空时删除当前任务
    vTaskDelete(NULL);
}
```

