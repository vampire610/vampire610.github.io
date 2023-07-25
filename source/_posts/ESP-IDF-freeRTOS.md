---
title: ESP-IDF与freeRTOS
date: 2023-07-23 00:45:58
tags:
  - ESP-IDF
  - freeRTOS
categories:
  - 学习笔记
---

# ESP-IDF与freeRTOS(一)

ESP-IDF是乐鑫官方推出的ESP32开发环境，个人不太喜欢arduino，所以选择学习使用IDF编程，上手发现很多地方十分陌生，在B站发现宝藏up [Michael_ee(点击跳转up主页)](https://space.bilibili.com/1338335828)讲的很详细，于是跟着学习并简单记录。

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

## Task四种参数传入

整体不难，基本c知识，简单记录

### 1.变量

``` c
// 输入参数定义为void类型指针，可强转其他任何类型参数传入，在函数内再转换回去
void myTask(void *pvParam)
{
    //创建int指针，指向参数，同时将参数转为int型
    int *pInt;
    pInt = (int *)pvParam;

    printf("I got Num = %d\n", *pInt);


    vTaskDelay(1000 / portTICK_PERIOD_MS);

    // delet函数为空时删除当前任务
    vTaskDelete(NULL);
}

//定义全局变量
int testNum = 5;
void app_main(void)
{   
    //传入时需取参数地址，并转成void指针
    xTaskCreate(myTask, "myTask1", 2048, (void *)&testNum, 1, NULL);
}
```

### 2.数组

``` c
void myTask(void *pvParam)
{
    // 创建int指针，指向参数，同时将参数转为int型
    int *pArrayAddr;
    pArrayAddr = (int *)pvParam;

    printf("I got Num1 = %d\n", *pArrayAddr);
    printf("I got Num2 = %d\n", *pArrayAddr + 1);
    printf("I got Num3 = %d\n", *pArrayAddr + 2);

    vTaskDelay(1000 / portTICK_PERIOD_MS);

    // delet函数为空时删除当前任务
    vTaskDelete(NULL);
}

// 定义全局变量
int testNum = {5, 6, 7};
void app_main(void)
{
    // 传入时需取参数地址，并转成void指针,数组名本身为地址，无需取地址
    xTaskCreate(myTask, "myTask1", 2048, (void *)testNum, 1, NULL);
}
```

### 3.结构体

``` c
typedef struct A_STRUCT
{
    int iMem1;
    int iMem2;
} xStruct;

xStruct xStrTest = {6, 9};

// 输入参数定义为void类型指针，可强转其他任何类型参数传入，在函数内再转换回去
void myTask(void *pvParam)
{
    // 定义结构体指针,转换类型
    xStruct *pStrTest;
    pStrTest = (xStruct *)pvParam;

    printf("I got iMem1 = %d\n", pStrTest->iMem1);
    printf("I got iMem2 = %d\n", pStrTest->iMem2);

    vTaskDelay(1000 / portTICK_PERIOD_MS);

    // delet函数为空时删除当前任务
    vTaskDelete(NULL);
}

// 定义全局变量
int testNum = {5, 6, 7};
void app_main(void)
{
    // 传入时需取参数地址，并转成void指针,数组名本身为地址，无需取地址
    xTaskCreate(myTask, "myTask1", 2048, (void *)&xStrTest, 1, NULL);
}
```

### 4.字符串

``` c
void myTask(void *pvParam)
{
    char *pcTxtInTask;
    pcTxtInTask = pvParam;

    printf("I got message = %s\n", pcTxtInTask);


    vTaskDelay(1000 / portTICK_PERIOD_MS);

    // delet函数为空时删除当前任务
    vTaskDelete(NULL);
}

// 定义全局变量
int testNum = {5, 6, 7};
//字符串
static const char *pcTxt = "I am 字符串";
void app_main(void)
{
    // 传入时需取参数地址，并转成void指针,数组名本身为地址，无需取地址
    xTaskCreate(myTask, "myTask1", 2048, (void *)pcTxt, 1, NULL);
}

```

## 优先级

优先级定义在`FreeRTOSConfig.h`

``` c
#define configMAX_PRIORITIES                            ( 25 )  //This has impact on speed of search for highest priority
```

可以看到当前最大优先级为25，可设置范围为0-24。

如果设置优先级超过最大值，自动更改为24。

`uxTaskPriorityGet()`:取得Handle对应优先级。

同优先级时，先创建先运行，不同时，先运行优先级高的任务。

``` c
void app_main(void)
{

    TaskHandle_t pxTask1 = NULL;
    TaskHandle_t pxTask2 = NULL;

    // 传入时需取参数地址，并转成void指针,数组名本身为地址，无需取地址
    //此处使用xTaskCreatePinnedToCore0()函数，应为我用的ESP32为双核芯片
    //不指定内核可能会分配到不同内核，导致设置优先级效果不明显
    xTaskCreatePinnedToCore(myTask1, "myTask1", 2048, (void *)pcTxt, 1, &pxTask1,0);
    xTaskCreatePinnedToCore(myTask2, "myTask2", 2048, (void *)pcTxt, 2, &pxTask2,0);
    
    //获取优先级并输出
    iPriority = uxTaskPriorityGet(pxTask1);
    printf("iPriority = %d\n", iPriority);

    //设置优先级
    vTaskPrioritySet(pxTask1,3);
}
```



## 任务的挂起（suspend）和恢复（resume）

- 挂起`vTaskSuspend(pxTask1)`

- 恢复`vTaskResume(pxTask1)`

用法和`vTaskDelete`类似，可在外部通过句柄操作，也可在任务内部传参NULL操作。

可在任务中挂起，在外部恢复。但恢复后会在任务中重新挂起。

可通过任务中挂起的方式实现在外部控制任务在需要的时候单次运行。

- 挂起调度器`vTaskSuspendAll()`
- 恢复调度器`xTaskResumeAll()`

挂起后不能再调用freeRTOS的API，只能调用自己的函数。

成对使用时可避免任务运行时被调度。

例如：

``` c
void myTask1(void *pvParam)
{
    printf("task begin\n");
    vTaskSuspendAll();
    for (int i = 0; i < 9999; i++)
    {
        for (int j = 0; j < 4000; j++)
        {
            ;
        }
    }
    xTaskResumeAll();
    printf("task end\n");
    
    vTaskDelete(NULL);
}
```

## vTaskList()

打印系统内所有任务的状态、信息。

使用该函数需要设置勾选menuconfig内

`configUSE_TRACE_FACILITY`和`configUSE_STATS_FORMATTING_FUNCTIONS`

``` c
    static char pcWriteBuffer[512] = {0};
    //函数会将Task信息格式化到数组
    vTaskList(pcWriteBuffer);
    printf("--------------------------------------------------\n");
    printf("Name      State    Priority   Stack     Num   core\n");
    printf("%s\n",pcWriteBuffer);

```

打印信息如下：

``` bash
--------------------------------------------------
Name          State  Priority  Stack   Num    core
myTask2         R       1       1748    8       0
main            X       1       3452    4       0
myTask1         R       1       1756    7       0
IDLE            R       0       1236    6       1
IDLE            R       0       1240    5       0
esp_timer       S       22      3568    3       0
ipc1            B       24      512     2       1
ipc0            B       24      488     1       0

```

State状态：

> R为ready就绪
>
> X为运行态
>
> D为被删除
>
> S为挂起
>
> B为阻塞

Stack为剩余堆栈空间


## Task堆栈设置和调试

`UBaseType_t uxTaskGetStackHighWaterMark( TaskHandle_t xTask );`

获取任务剩余堆栈的值，以此作为参考估算应该设置的任务堆栈大小，节省系统资源。

示例：
``` c
void myTask1(void *pvParam)
{
    while (1)
    {
        printf("task1\n");
        vTaskDelay(1000 / portTICK_PERIOD_MS);
    }
}

void app_main(void)
{
    TaskHandle_t pxTask1 = NULL;

    // 传入时需取参数地址，并转成void指针,数组名本身为地址，无需取地址
    xTaskCreatePinnedToCore(myTask1, "myTask1", 1024, NULL, 1, &pxTask1, 0);

    UBaseType_t iStack;

    while (1)
    {
        iStack = uxTaskGetStackHighWaterMark(pxTask1);
        printf("task1 iStack = %d\n", iStack);
        vTaskDelay(3000 / portTICK_PERIOD_MS);
    }
}
```

串口输出：

``` bash
......
I (0) cpu_start: Starting scheduler on APP CPU.
task1 iStack = 732
task1
task1
task1
task1 iStack = 204
task1
task1
task1
task1 iStack = 204
task1
```

由此可见，当前我的开发板运行task1设置堆栈为1024时剩余空间为204。

