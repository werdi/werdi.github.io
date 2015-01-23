---
title: Работа с командной строкой
tags: [command prompt, c#]
src: http://mindberg.blogspot.ru/2007/06/blog-post_30.html
---
# Работа с командной строкой
Как можно организовать работу с командной строкой (подразумевается, что через командную строку программа получает необходимые данные).

Способ 1:
(код разбора строки и загрузки свойств находится внутри класса)
```c#
public class Host
{
  public static void Main(string[] args)
  {
    Util util = new Util(args);
    util.Start();
  }
}
public class Util
{
  public Util(string[] args)
  {
    // парсинг и валидация значений args[]
  }
  public void Start() {...}
}
```
Плюс: коротко и ясно.

Минус: если Util понадобится, например, в WF-активности, то при каждом вызове будет происходить парсинг и валидация значений args[].

Примечание: конструктор Util можно сделать без параметров, при этом использовать значение Environment.CommandLine.

Способ 2:
(отдельный immutable-класс с параметрами для класса Util)
```c#
public class Host
{
  public static void Main(string[] args)
  {
    UtilSettings us = new UtilSettings(args);
    Util util = new Util(us);
    util.Start();
  }
}
public class Util
{
  public Util(UtilSettings s) {...}   
  public void Start() {...}
}
public class UtilSettings
{
  public UtilSettings(string[] args) {...}
}
```
Плюс: избавились от минуса, который был в «Способ 1».

Минус: данные для UtilSettings передаются через конструктор. Если источников с данными будет несколько, то и конструкторов будет несколько. В итоге, чтобы понять что делать с данными, надо будет смотреть на количество и очередность параметров у конструкторов.

Способ 3:
(модифицированный UtilSettings из «Способ 2»)
```c#
public class UtilSettings
{
  private UtilSettings() { }
  public static UtilSettings FromCommandLine(string[] args) {...}
}
```
Плюс: избавились от минуса, который был в «Способ 2».

Минус: нет.

Теперь, если понадобится загрузить параметры, например, из xml-файла, то в UtilSettings надо просто добавить соответствующий метод, например,
```c#
public static UtilSettings FromXml(string path) {…}
```
