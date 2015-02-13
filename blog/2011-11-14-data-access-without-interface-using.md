---
title: Data access without using a Interface
tags: [c#, winforms, linq]
src: https://social.msdn.microsoft.com/Forums/ru-RU/cfff93e9-fc30-4622-9868-bee6744e1333/data-access-without-using-a-interface?forum=csharpgeneral
---
# Data access without using a Interface
*> Normally a 3trd party component must implement a interface i defined in one of my assemblies.
But, is there a other solution without using a interface? [...] If the external class doesn't habe a Method named Test() a NotSupportedException will thrown.*

you can check existence of interface through the following methods:
```c#
var r1 = typeof(ITest).IsAssignableFrom(typeof(External.Some));
var r2 = typeof(External.Some).GetInterface("ITest");
```
they return true in case:
```c#
namespace MyApp
{
  interface ITest { /*...*/ }
}
namespace External
{
  public class Some : MyApp.ITest { /*...*/ }
}
```
another approach is verifying a method's signature
```c#
using System.Collections.Generic;
using System.Linq;
using System.Windows.Forms;
using System;

namespace WindowsFormsApplication2
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var res = Verify<ITest, External.Some>();
      new RichTextBox
      {
        Parent = this,
        Dock = DockStyle.Fill,
        Text = res.Count == 0 ? "ok" : "missing: \n" + String.Join("\n", res)
      };
    }

    private List<string> Verify<T1, T2>() where T2 : class
    {
      Dictionary<string, bool> tbl = new Dictionary<string, bool>();
      foreach (var mi in typeof(T1).GetMembers())
        tbl.Add(mi.ToString(), false);

      foreach (var mi in typeof(T2).GetMembers())
      {
        var key = mi.ToString();
        if (tbl.ContainsKey(key))
          tbl[key] = true;
      }
      return tbl.Where(kv => !kv.Value).Select(kv => kv.Key).ToList();
    }
  }

  interface ITest
  {
    void Test(string message);
    string Text { get; set; }
  }
}

namespace External
{
  public class Some 
  {
    public void Test(string message) { }
    public string Text { get { return null; } set { } }
  }
}
```
