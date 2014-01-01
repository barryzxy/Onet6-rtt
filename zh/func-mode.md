
# 线程创建 #

---------------------------------------------------------------------
		所在文件		调用者		配置
------------------  ----------- ----
	rt-thread.c		线程或中断		o
------------------  ----------- -----
```c
    rt_thread_t rt_thread_create(const char* name,
                                 void (*entry)(void* parameter), void* parameter,
                                 rt_uint32_t stack_size,
                                 rt_uint8_t priority, rt_uint32_t tick);
```


一个线程要成为可执行的对象就必须由操作系统的内核来为它创建（初始化）一个线程句柄。可以通过如下的函数接口来创建一个线程。


调用这个函数时，系统会从动态堆内存中分配一个线程句柄（即TCB，线程控制块）以及按照参数中指定的栈大小从动态堆内存中分配相应的空间。分配出来的栈空间是按照rtconfig.h中配置的RT_ALIGN_SIZE方式对齐。






**函数参数**

-----------------------------------------------------------------------
           参数  描述
--------------  -------------------------------------------------------
          name  线程的名称；线程名称的最大长度由rtconfig.h中定义的
                RT_NAME_MAX宏指定，多余部分会被自动截掉。

         entry  线程入口函数

     parameter  线程入口函数参数；

    stack_size  线程栈大小，单位是字节。在大多数系统中需要做栈空间地
                址对齐（例如ARM体系结构中需要向4字节地址对齐）。

      priority  线程的优先级。优先级范围根据系统配置情况（rtconfig.h中的
                RT_THREAD_PRIORITY_MAX宏定义），如果支持的是256级优先级，
                那么范围是从0 ～ 255，数值越小优先级越高，0代表最高优先级。

          tick  线程的时间片大小。时间片（tick）的单位是操作系统的时钟节拍。当系统中存在相同优先级线程时，这个参数指定线程一次调度能够运行的最大时间长度。这个时间片运行结束时，调度器自动选择下一个就绪态的同优先级线程进行运行。
-----------------------------------------------------------------------

**返回**


-------------------
	参数				描述
-----------------------------
	成功				线程句柄
	不成功			  RT_NULL
-----------------------
		
**函数返回**

创建成功返回线程句柄；否则返回RT_NULL。

* 注：确定一个线程的栈空间大小，是一件令人头痛的事情。在RT-Thread中，可以先指定一个稍微大的栈空间，例如指定大小为1024或2048，然后在FinSH shell中通过list_thread()命令查看线程运行的过程中线程所使用的栈的大小，通过此命令，能够看到从线程启动运行时，到当前时刻点，线程使用的最大栈深度，从而可以确定栈空间的大小并加以修改)。
![状态转移](http://i.imgur.com/xWS8KE7.jpg)
下面举例创建一个线程加以说明：

~~~{.c}
/*
 * 程序清单：动态线程
 *
 * 这个程序会初始化2个动态线程：
 * 它们拥有共同的入口函数，相同的优先级
 * 但是它们的入口参数不相同
 */
#include <rtthread.h>

#define THREAD_PRIORITY         25
#define THREAD_STACK_SIZE       512
#define THREAD_TIMESLICE        5

/* 指向线程控制块的指针 */
static rt_thread_t tid1 = RT_NULL;
static rt_thread_t tid2 = RT_NULL;
/* 线程入口 */
static void thread_entry(void* parameter)
{
    rt_uint32_t count = 0;
    rt_uint32_t no = (rt_uint32_t) parameter; /* 获得正确的入口参数 */

    while (1)
    {
        /* 打印线程计数值输出 */
        rt_kprintf("thread%d count: %d\n", no, count ++);

        /* 休眠10个OS Tick */
        rt_thread_delay(10);
    }
}

/* 用户应用入口 */
int rt_application_init()
{
    /* 创建线程1 */
    tid1 = rt_thread_create("t1",
        thread_entry, (void*)1, /* 线程入口是thread_entry, 入口参数是1 */
        THREAD_STACK_SIZE, THREAD_PRIORITY, THREAD_TIMESLICE);
    if (tid1 != RT_NULL)
        rt_thread_startup(tid1);
    else
        return -1;

    /* 创建线程2 */
    tid2 = rt_thread_create("t2",
        thread_entry, (void*)2, /* 线程入口是thread_entry, 入口参数是2 */
        THREAD_STACK_SIZE, THREAD_PRIORITY, THREAD_TIMESLICE);
    if (tid2 != RT_NULL)
        rt_thread_startup(tid2);
    else
        return -1;

    return 0;
}
~~~
