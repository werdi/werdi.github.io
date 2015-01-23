---
title: Single-Instance Apps
tags: [single-instance, c#]
src: http://mindberg.blogspot.ru/2007/05/single-instance-apps.html
---
# Single-Instance Apps
Очень часто на форумах встречается одна и таже тема под разными заголовками, например: "Один экземпляр приложения", "Защита от повторного запуска", "Как определить запущено приложение или нет", "Одна копия приложения ... как лучше".
 
В ответ предлагаются разные решения (на основе Mutex, Semaphore и т.п.), но все они не учитывают "подводные камни", о которых сказано в статье "Stream Decorator, Single-Instance Apps" (MSDN Magazine). 
 
В .NET 2.0 есть стандартное решение (см. Figure 5 Using SingleInstanceApplication):
```c#
static class Program
{
    [STAThread]
    static void Main()
    { 
        Application.EnableVisualStyles();
        SingleInstanceApplication.Run(new Form1(), 
            StartupNextInstanceHandler);
    }

    static void StartupNextInstanceHandler(
        object sender, StartupNextInstanceEventArgs e)
    {
        // do whatever you want here with e.CommandLine...
    }
}
```
Класс SingleInstanceApplication находится в сборке Microsoft.VisualBasic.dll в пространстве имен Microsoft.VisualBasic.ApplicationServices. Правда, эта сборка в C# проекте выглядит необычно, но работает отлично.
