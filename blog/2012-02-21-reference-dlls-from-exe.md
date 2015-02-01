---
title: Как внедрить dll ссылку в exe
tags: [c#, winforms, resources]
src: https://social.msdn.microsoft.com/Forums/ru-RU/45b6cc60-a859-45fe-b271-ec844c8894cf/-dll-exe?forum=fordesktopru
---
# Как внедрить dll ссылку в exe
*> как внедрить dll ссылку в exe, чтобы для работы программы нужен был только exe файл?*

добавить dll в виде Embedded Resource.
например, есть проект Class Library:
```c#
namespace ClassLibrary1
{
  public class Class1
  {
    public string Test()
    {
      return "hello";
    }
  }
}
```
компилируем. получим ClassLibrary1.dll
 
создаем проект Windows Forms Application. в него добавим файл ClassLibrary1.dll (внимание: добавляем не в References), при этом в свойствах файла надо указать Build Action: Embedded Resources.
 
после этого в рантайме ClassLibrary1.dll можно получить с помощью GetManifestResourceStream
```c#
using System;
using System.Reflection;
using System.Windows.Forms;

namespace WindowsFormsApplication4
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      using(var s = Assembly.GetExecutingAssembly().GetManifestResourceStream("WindowsFormsApplication4.ClassLibrary1.dll"))
      {
        var bytes = new byte[s.Length];
        s.Read(bytes, 0, (int)s.Length);
        var a = Assembly.Load(bytes);
        var t = a.GetType("ClassLibrary1.Class1");
        dynamic o = Activator.CreateInstance(t);
        var res = o.Test();
        MessageBox.Show(res);
      }
    }
  }
}
```
