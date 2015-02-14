---
title: Динамический запрос LINQ
tags: [c#, winforms, linq, compiler]
src: https://social.msdn.microsoft.com/Forums/ru-RU/135f891d-9a27-4dd1-913f-d1e7bc57421b/-linq?forum=fordataru
---
# Динамический запрос LINQ
*> Можно ли собрать строку запроса как string и выполнить её, примерно такstring query = "from i in source where .условия . условия.... select i"; //Каким нибудь макаром выполнить всё это.*

можно с помощью CSharpCodeProvider. 
примерно так: 
```c#
using System;
using System.CodeDom;
using System.CodeDom.Compiler;
using System.Collections;
using System.Collections.Generic;
using System.Dynamic;
using System.Linq;
using System.Reflection;
using System.Windows.Forms;
using Microsoft.CSharp;

namespace WindowsFormsApplication2
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var arr = new [] { 1, 2, 3, 4, 5 };

      var res1 = LinqHelper.Invoke("from i in src select i*100", arr);
      var lv = new ListView { Parent = this, Dock = DockStyle.Fill };
      foreach(var i in res1)
        lv.Items.Add(i.ToString());

      var res2 = LinqHelper.Invoke("from i in src where i % 2 == 0 select new { Value=i*100 }", arr);
      new DataGrid { Parent = this, Dock = DockStyle.Bottom, Height=200, DataSource = res2.Cast<object>().ToList() };
    }
  }

  public static class LinqHelper
  {
    public static IEnumerable Invoke<T>(string code, IEnumerable<T> en)
    {
      var a = Compile(String.Concat(@"
                using System;
                using System.Linq;
                using System.Collections.Generic;
                public static class Some { public static object Run(IEnumerable<", 
                  typeof(T).FullName, 
                  "> src) { return (", code, "); }}"),
                "System.dll",
                "System.Core.dll");
      var t = a.CompiledAssembly.GetType("Some");
      var m = t.GetMethod("Run", BindingFlags.Static | BindingFlags.Public);
      return m.Invoke(null, new object[] { en }) as IEnumerable;
    }

    public static CompilerResults Compile(string code, params string[] assemblies)
    {
      var csp = new CSharpCodeProvider();
      var ccu = new CodeCompileUnit();
      var cps = new CompilerParameters();
      cps.ReferencedAssemblies.AddRange(assemblies);
      cps.GenerateInMemory = true;
      return csp.CompileAssemblyFromSource(cps, code);
    }
  }
} 
```
также см. [dynamic query LINQ helper library](http://weblogs.asp.net/scottgu/archive/2008/01/07/dynamic-linq-part-1-using-the-linq-dynamic-query-library.aspx)