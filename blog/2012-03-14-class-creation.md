---
title: Как создать вот такой вот класс
tags: [c#, winforms, mef]
src: https://social.msdn.microsoft.com/Forums/ru-RU/e08c8717-4acc-4559-a0db-3e8ae5f485df/-?forum=programminglanguageru
---
# Как создать вот такой вот класс
*> класс должен работать без инициализации объекта. (например методы класса Math можно же использовать сразу) [...] совершенно не понимаю как можно реализовать именно 4-й пункт.*

см. [Static Members] (http://msdn.microsoft.com/en-us/library/79b3xss3)

*> результаты одних и тех же вычислений, могут использоваться в разных контролах приложения, а т.к. вычисления достаточно трудоемкие, то постоянно их вычислять, совершенно не рентабельно*

см. [Lazy<T>] (http://msdn.microsoft.com/ru-ru/library/dd642331.aspx); примерно так:
```c#
class Test
{
  public int Value
  {
    get { return _Value.Value; }
  }
  Lazy<int> _Value;

  public Test()
  {
    _Value = new Lazy<int>(Compute, true);
  }

  int Compute()
  {
    var ret = 0;
    //
    // сложные вычисления ret;
    //
    return ret;
  }
}
```
*> есть еще классы не контролы, в которых нет прямого  способа получить Tag приложения и в такие классы приходится егопередавать, что совсем не удобно*
в такой ситуации можно использовать [System.ComponentModel.Composition] (http://msdn.microsoft.com/ru-ru/library/system.componentmodel.composition.aspx)

[c:\temp\addin\some.dll]
```c#
using System;
using System.ComponentModel.Composition;
namespace Test
{
  class Some
  {
    [Export("v1")]
    string Value
    {
      get { return DateTime.Now.ToString(); }
    }
  }
}
```
значение Some.Value можно получить в своем приложении так:
```c#
var v1 = this.Get<string>("v1");
```
для этого надо к проекту добавить сборку System.ComponentModel.Composition.dll
и в форме должен быть следующий код:
```c#
using System.ComponentModel.Composition;
using System.ComponentModel.Composition.Hosting;
using System.Windows.Forms;

namespace WindowsFormsApplication1
{
  public partial class Form1 : Form
  {
    readonly CompositionContainer _Container;
    T Get<T>(string name = null)
    {
      return _Container.GetExportedValueOrDefault<T>(name);
    }

    public Form1()
    {
      _Container = new CompositionContainer(
        new DirectoryCatalog(@"c:\temp\addin\", "*.dll"));
      _Container.ComposeParts(this);

      var v1 = this.Get<string>("v1");
    }
  }
}
```
