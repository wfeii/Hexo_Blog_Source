---
title: Log分析
tags:
---
Bug在任何开发中都会出现的，而Log又是解决bug的关键。  
logcat文件以文字形式存储了所有了的日志信息，**system**部分存储了framework的相关log，而**main**包含了所有log。
每行log都以 ** timestamp PID（Process ID） TID (Thread ID) log-level**开头。  

'''  
\------ SYSTEM LOG (logcat -v threadtime -d *:v) ------
--------- beginning of system  
Blah  
Blah  
Blah  

--------- beginning of main  
Blah   
Blah  
Blah    
'''  

### 阅读Event log   
Event log 里面包含了文字形式的二进格式的log信息，它比logcat里面的信息简洁但是更难理解。当阅读event log的时候，我们应该搜索特定的PID来看进程做了什么，格式为**timestamp PID TID log-level log-tag tag-values.**   

log等级包括：
- V: verbose
- D: debug
- I: information
- W: warning
- E: error  

’‘’  
------ EVENT LOG (logcat -b events -v threadtime -d *:v) ------  
09-28 13:47:34.179   785  5113 I am_proc_bound: [0,23054,com.google.android.gms.unstable]  
09-28 13:47:34.777   785  1975 I am_proc_start: [0,23134,10032,com.android.chrome,broadcast,com.android.chrome/org.chromium.chrome.browser.precache.PrecacheServiceLauncher]  
09-28 13:47:34.806   785  2764 I am_proc_bound: [0,23134,com.android.chrome]  
...  
‘’’  
对于Event log的标签更详细的信息,可以阅读[Event log tags](https://android.googlesource.com/platform/frameworks/base/+/master/services/core/java/com/android/server/EventLogTags.logtags)  

### ANR和死锁  
log能帮助我们找到ANR错误和死锁

#### 识别无响应的APP  
当应用一段时间无响应的时候，通常是由于主线程阻塞了或者是主线程做的事太多.系统会kill掉应用并且把相关的堆栈信息存储在/data/anr目录下。
