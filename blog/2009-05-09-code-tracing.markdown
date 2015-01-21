---
title: Трассировка кода
tags: [.net, trace, c#]
src: http://cognitex.blogspot.ru/2009/05/blog-post_6462.html
---
# Трассировка кода
Трассировка (журналирование / аудит) - это вывод информации о текущем состоянии программы.

Для реализации трассировки в классе надо создать объект типа TraceSource, а для вывода информации использовать метод, например, TraceInformatio:
```c#
public class MyClass
{
    	private readonly static TraceSource T;
    	static MyClass()
    	{
        	T = new TraceSource("MyClass", SourceLevels.All);
    	}

    	public void Method()
    	{
        	T.TraceInformation("Method start");
        	// ...
        	T.TraceInformation("Method done");
    	}
}
```

В результате запуска debugger'а в окне Immadiate Window или Output Window (зависит от настроек) появятся следующие строки:
```c#
MyClass Information: 0 : Method start
MyClass Information: 0 : Method done
```
Если отладочную информацию требуется вывести в файл trace.log, то в конструкторе формы должен быть следующий код:
```c#
Trace.Listeners.Clear();    // удалить DefaultTraceListener
Trace.Listeners.Add(new TextWriterTraceListener("trace.log"));

MyClass mc = new MyClass();
mc.Method();
```
Для создания своего получателя трассировочной информации надо наследовать класс TraceListener и переопределить два метода: Write и WriteLine.

Управлять выводом трассировочной информацией можно посредством файла конфигурации приложения (config-файла). Подробнее см. [здесь] (http://msdn.microsoft.com/ru-ru/library/system.diagnostics.traceswitch.aspx).
