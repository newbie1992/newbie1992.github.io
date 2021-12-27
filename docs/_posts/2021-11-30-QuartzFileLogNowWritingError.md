---
layout: post
title: 处理Quartz.net在运行Console程序无法写入file log的问题
---

前言：
Quartz.net在运行C# Console 程序时log4net 不会在Console程序出现异常时写入指定的文件C:\Logs\xxxx\Dev\年\日期\xxx_exception.log

Quartz.net的程序码如下:
```
var process = new Process();
using (process)
{
    #if DEV
        process.StartInfo.FileName = @"C:\XXX\Dev\dist\Cron.Task.exe";
    #elif UAT
        process.StartInfo.FileName =  @"C:\XXX\UAT\dist\Cron.Task.exe";
    #elif PROD
        process.StartInfo.FileName =  @"C:\XXX\PROD\dist\Cron.Task.exe";
    #else
        process.StartInfo.FileName =  @"C:\XXX\Dev\dist\Cron.Task.exe";
    #endif
        process.Start();
        process.WaitForExit();
}
```
调试1:
直接在C:\XXX\Dev\dist\文件夹里运行Cron.Task.exe 是会正常写入到指定的文件 ✔

调试2:
在Console程序namespace前加入
```
[assembly: log4net.Config.XmlConfigurator(Watch=true)]
```
由于log4net.config是有用动态变量，文件出现C:\XXX\Dev\dist(null)年\日期\xxx_exception.log 是不正确的 ❌

调试3:
在Console程序移除
```
[assembly: log4net.Config.XmlConfigurator(Watch=true)]
```
然后在添加在Program Main
```
var myLogger = LogManager.GetLogger("LoggingFileAppender");
var myappender = myLogger.Logger.Repository.GetAppenders();
Console.WriteLine($"total appender: {myappender.Length}");
foreach (var appender in myappender)
{
    Console.WriteLine(appender.Name);
}
```

经过此次调试发现当Quartz呼叫Console程式时log4net file appender是空的，造成程序出现异常时不会写入指定的xxx_exception.log.

如何需要检查logger可以改善上面的代码来检查文件的写入路径

```
Console.WriteLine(myappender[0].GetType().GetProperty("File").GetValue(myappender[0]));
```

解决方法：
修改Quartz.net的程序码，利用ProcessStartInfo()来自定义工作目錄以确保Console程序能够在此目录找到log4net.config
```
//var process = new Process();
var startInfo = new ProcessStartInfo();
// using (process)
    // {
        #if DEV
            //process.StartInfo.FileName = @"C:\XXX\Dev\dist\Cron.Task.exe";
            startInfo.FileName = @"C:\XXX\Dev\dist\Cron.Task.exe";
            startInfo.WorkingDirectory =  @"C:\XXX\Dev\dist\;
        #elif UAT
            //process.StartInfo.FileName =  @"C:\XXX\UAT\dist\Cron.Task.exe";
            startInfo.FileName = @"C:\XXX\UAT\dist\Cron.Task.exe";
            startInfo.WorkingDirectory =  @"C:\XXX\UAT\dist\;
        #elif PROD
            //process.StartInfo.FileName =  @"C:\XXX\PROD\dist\Cron.Task.exe";
            startInfo.FileName = @"C:\XXX\PROD\dist\Cron.Task.exe";
            startInfo.WorkingDirectory =  @"C:\XXX\PROD\dist\;
        #else
            //process.StartInfo.FileName =  @"C:\XXX\Dev\dist\Cron.Task.exe";
            startInfo.FileName = @"C:\XXX\Dev\dist\Cron.Task.exe";
            startInfo.WorkingDirectory =  @"C:\XXX\Dev\dist\;
        #endif
        
        //process.Start();
        //process.WaitForExit();

                try
                {
                    using (var process = Process.Start(startInfo))
                    {
                        process.WaitForExit();
                    }
                }
                catch (Exception)
                {
                }
    }

```
