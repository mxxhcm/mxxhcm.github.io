---
title: UNIX time(s)
date: 2019-12-08 14:02:07
tags:
 - UNIX
categories: UNIX
---

## 时间
### 获得时钟时间
#### linux的时间函数
三种获取时间的方式
``` c++
// 获得time_t或者,struct timespec或者struct timeval
// 1. 返回从1970年1月1日（UTC）开始的秒数。
time_t time(time_t *tloc);
// 2.1
// timespec -> tv_sec
int clock_gettime(clockid_t clk_id, struct timpespec *tp);
// 2.2
int clock_getres(clockid_t clk_id, struct timespec *res);
// 3.
// timeval -> tv_sec
int gettimeofday(struct timeval *tv, struct timezone *tz);

//将time_t结构化成struct tm
// 1.1
// 本地时间
struct tm *localtime(const time_t *timep);
// 1.2
// 协调世界时
struct tm *gmtime(const time_t *timep);
// 2.
// 将结构化时间转换成time_t
time_t mktime(struct tm *tm);
```

#### C的时间函数
``` c++
// 1.格式化日期和时间
size_t strftime(char *s, size_t max, const char *format, const struct tm *tm);
// 2.
char *strptime(const char *s, const char *format, struct tm *tm);
```

### 获得进程时间
#### linux进程时间
``` c++
clock_t clock(void);
```

#### C的进程时间
``` c++
clock_t times(struct tms *buf);
```

## 数据结构
- `struct timespec`
- `struct timeval`
- `struct tm`
- `struct tms`
- `struct timezone` 

### `struct timespec`
``` c++
struct timespec
{
    time_t tv_sec;
    long tv_nsec;
};
```

### `struct timeval`
``` c++
struct timeval
{
    time_t tv_sec;
    suseconds_t tv_usec;
};
```

### `struct tm`
``` c++
struct tm{
    int tm_sec;    /* Seconds (0-60) */
    int tm_min;    /* Minutes (0-59) */
    int tm_hour;   /* Hours (0-23) */
    int tm_mday;   /* Day of the month (1-31) */
    int tm_mon;    /* Month (0-11) */
    int tm_year;   /* Year - 1900 */
    int tm_wday;   /* Day of the week (0-6, Sunday = 0) */
    int tm_yday;   /* Day in the year (0-365, 1 Jan = 0) */
    int tm_isdst;  /* Daylight saving time */
};
```

### `struct tms`
``` c++
struct tms
{
    clock_t tms_utime;
    clock_t tms_stime;
    clock_t tms_cutime;
    clock_t tms_cstime;
    
};
```

### `struct timezone` 
``` c++
struct timezone
{
    int tz_minuteswest;
    int tz_dsttime;
};
```

## 参考文献
1.《APUE》第三版
2.https://en.cppreference.com/w/c/chrono/timespec
3.https://www.gnu.org/software/libc/manual/html_node/Elapsed-Time.html
