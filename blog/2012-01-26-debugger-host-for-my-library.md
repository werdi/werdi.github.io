---
title: Создание Host-отладчика для отладки своей библиотеки (DLL)
tags: [c#, visual studio]
src: https://social.msdn.microsoft.com/Forums/ru-RU/a2c8c882-d53c-40f8-b6fb-d2288c4a1f76/-host-dll?forum=vsru
---
# Создание Host-отладчика для отладки своей библиотеки (DLL)
*> Создание Host-отладчика для отладки свой библиотеки (DLL)*

если требуется управляемый отладчик, то см. [Debugging Interfaces] (http://msdn.microsoft.com/ru-ru/library/ms404484.aspx)
и [Notes on Managed Debugging, ICorDebug, and random .NET stuff] (http://blogs.msdn.com/b/jmstall/archive/tags/mdbg/)

*> есть в поле "Аргументы командной строки" можно указать какой-то параметр, который вернет путь в dll проекта?*

в Visual Studio - главное меню - Project - <название проекта>Properties... - Build Events: Post-build event command line - Edit Post-build... - Macros>>: $(TargetPath)

т.е. в Post-build event command line можно указать:
c:\myhost.exe "$(TargetPath)"

после успешной компиляции сборки будет запускаться myhost.exe
в который передается путь к созданной сборке.
в myhost.exe должен быть примерно такой код:
```c#
using System;
using System.Windows.Forms;

namespace WindowsFormsApplication3
{
  static class Program
  {
    [STAThread]
    static void Main(string[] args)
    {
      // ... args[0] - путь к сборке
    }
  }
}
```
*> После закрытия myhost.exe выдается сообщение что это библиотека классов и не может быть запущена.*

выводится в MessageBox? или в окне Visual Studio? если можно приведите полное сообщение.

*> нужно обработать команду на запуск отладки и подсунуть свой отладчик. [...] Если есть что-то в эту сторону, то подскажите.*

см. [Extend Visual Studio] (http://msdn.microsoft.com/en-us/vstudio/ff718165.aspx),  VS Shell
