# 睡眠与计时

## 睡眠时间

### Windows 系统睡眠

用 `<windows.h>` 提供的 `Sleep` 函数。此函数以**毫秒**数作为输入参数， 使用示例：

```C
#include <windows.h>
int main() 
{
    Sleep(1234);    // Sleep for 1.234 seconds (1234 milliseconds)
    return 0;
}
```

### 类 Unix 系统睡眠

用 `<unistd.h>` 提供的**秒级**精度的 `sleep` 函数或者**微秒**级精度的 `usleep` ， 例如：

```C
#include <unistd.h>
int main() 
{
    sleep(5);       // Sleep for 5 seconds
    usleep(500000); // Sleep for 500 milliseconds
    return 0;
}
```

### 跨平台睡眠包装

```C
#if defined(_WIN32) || defined(_WIN64)
    #include <windows.h>
#else
    #include <unistd.h>
#endif

// second precision
void sleeps(int seconds) {
#if defined(_WIN32) || defined(_WIN64)
    Sleep(seconds * 1000); // Sleep(ms) on Windows system
#else
    sleep(seconds);        // sleep(s) on Unix-like systems
#endif
}

// milliseconds precision
void sleepms(int mseconds) {
#if defined(_WIN32) || defined(_WIN64)
    Sleep(mseconds);         // Sleep(ms) on Windows system
#else
    usleep(mseconds * 1000); // usleep(us) on Unix-like systems
#endif
}

// microsecond precision
void sleepus(int useconds) {
    usleep(useconds);        // usleep(us) on Unix-like systems
}
```

## 时间获取及耗时测量

### 微秒级 gettimeofday

此函数用于获取当前时间，精度可达微秒。它通常用于类 Unix 系统。`gettimeofday` 在 `<sys/time.h>` 中， 原型为

```C
int gettimeofday(struct timeval *tv, struct timezone *tz);
```

`tv` 指向的时间结构体定义如下:

```C
struct timeval {
    time_t tv_sec;       // Seconds passed since the reference time (00:00:00 UTC on 1 January 1970).
    suseconds_t tv_usec; // Microseconds passed since tv_sec
};
```

而 `tz` 则一般设为 `NULL` 即空指针。

### 用 gettimeofday 测量耗时

准备时间差计算函数

```C
double dusec(struct timeval tic, struct timeval toc)
{
    long  s = toc.tv_sec  - tic.tv_sec;
    long us = toc.tv_usec - tic.tv_usec;
    double dt = s * 1e6 + us;
    return dt;
}
```

测量示例

```C
#include <math.h>
#include <stdio.h>
#include <sys/time.h>

int main() 
{
    struct timeval tic;
    struct timeval toc; 

    gettimeofday(&tic, NULL);

    for(int k = 0; k < 12345678; k++){
        // do something here like
        double y = sin(k);
    }

    gettimeofday(&toc, NULL);

    double elapsed = dusec(tic, toc);
    printf("Time measured: %f us\n", elapsed);
    return 0;
}
```

### 纳秒级 clock_gettime

> 注意，一些较老的系统或非 POSIX 系统可能不支持它。

此函数可以度量不同类型的时钟，这取决于所提供的时钟 ID。它的原型如下:

```C
int clock_gettime(clockid_t clk_id, struct timespec *tp);
```

其中 `clk_id` 为要测量的时钟类型标识符

具体有:

+ `CLOCK_REALTIME`: 系统时钟，可能会受到系统时间变化的影响，使其不太适合测量运行时间
+ `CLOCK_MONOTONIC`: 从某个起点而产生的单调时间, 建议用它测量运行时间，因为它不受系统时间变化的影响
+ `CLOCK_PROCESS_CPUTIME_ID`: 来自 CPU 的高分辨率进程计时器
+ `CLOCK_THREAD_CPUTIME_ID`: 特定于线程的 CPU 时钟

而 `tp` 指向时间结构体, 定义如下:

```C
struct timespec {
    time_t tv_sec;  // Seconds
    long   tv_nsec; // Nanoseconds
};
```

### 用 clock_gettime 测量耗时

准备时间差计算函数

```C
double dnsec(struct timespec tic, struct timespec toc)
{
    long  s = toc.tv_sec  - tic.tv_sec;
    long ns = toc.tv_nsec - tic.tv_nsec;
    double dt = s * 1e9 + ns;
    return dt;
}
```

测试

```C
#include <math.h>
#include <stdio.h>
#include <sys/time.h>

int main() 
{
    struct timespec tic;
    struct timespec toc; 

    clock_gettime(CLOCK_MONOTONIC, &tic);

    for(int k = 0; k < 12345678; k++){
        // do something here like
        double y = sin(k);
    }

    clock_gettime(CLOCK_MONOTONIC, &toc);

    double elapsed = dnsec(tic, toc);
    printf("Time measured: %f ns\n", elapsed);
    return 0;
}
```
