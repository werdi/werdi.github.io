---
title: AppDomain, create instance from class in assembly, invoke member and unload assembly
tags: [c#, winforms, reflection, interop, compiler]
src: https://social.msdn.microsoft.com/Forums/ru-RU/b33bef19-9cd6-4a2f-bd44-3007c2298f05/load-a-assembly-from-a-arbitrary-folder-into-newly-created-appdomain-create-instance-from-class-in?forum=csharpgeneral
---
# AppDomain, create instance from class in assembly, invoke member and unload assembly
*> Loading the assembly into a separate newly created appdomain, create a instance from an included class, invoke a member on it, get its result [...]  then unload the assembly*

the following sample dynamically creates an assembly from the source code;
then creates AppDomain, loads created assembly, invokes method, unloads AddDomain, and deletes assembly.
```c#
using System;
using System.CodeDom.Compiler;
using System.Drawing;
using System.IO;
using System.Reflection;
using System.Windows.Forms;
using Microsoft.CSharp;

namespace WindowsFormsApplication6
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.Size = new Size(500, 450);

      var tb = new RichTextBox
        {
          Parent = this,
          Dock = DockStyle.Fill,
          Text = "CurrentDomain: " + AppDomain.CurrentDomain.GetHashCode() + "\n"
        };

      var index = 0;
      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("Run", (s, e) => 
        Test(index++, str => tb.AppendText(str + Environment.NewLine)));
    }

    void Test(int index, Action<string> callback)
    {
      var fname = "test" + index + ".dll";
      // create test*.dll from the code;
      var res = Compile(
        fname, new [] {"System.dll", "System.Windows.Forms.dll", "System.Drawing.dll"},  @"
        using System; 
        using System.Windows.Forms;
        using System.Drawing;
        public class Test {  
          public string Run(int data) { 
            return String.Concat(data, ""; "", Control.MousePosition, ""; "", " + index + @"); 
          }
        }");
      if (res.Count > 0)
      {
        foreach(CompilerError err in res)
          callback(err.ErrorText + Environment.NewLine);
      }
      else
      {
        var ad = AppDomain.CreateDomain(String.Empty);
        ad.DomainUnload += (s, e) =>  // invoked from the created AppDomain
          System.Diagnostics.Trace.WriteLine(s.GetHashCode(), "DomainUnload");

        var p = Proxy.Create(ad);
        var r = p.Run(fname, "test", "run", 1);
        callback(r);

        AppDomain.Unload(ad);
        File.Delete(fname);
      }
    }

    CompilerErrorCollection Compile(string path, string[] assemblies, string cs)
    {
      var csp = new CSharpCodeProvider();
      var cps = new CompilerParameters();
      cps.ReferencedAssemblies.AddRange(assemblies);
      cps.GenerateInMemory = false;
      cps.OutputAssembly = path;
      return csp.CompileAssemblyFromSource(cps, cs).Errors;
    }
  }

  public class Proxy : MarshalByRefObject
  {
    public static Proxy Create(AppDomain ad)
    {
      var pt = typeof(Proxy);
      return ad.CreateInstanceAndUnwrap(pt.Assembly.FullName, pt.FullName) as Proxy;
    }

    public string Run(string file, string type, string method, int some)
    {
      var ret = "AppDomain: " + AppDomain.CurrentDomain.GetHashCode();
      var a = Assembly.Load(File.ReadAllBytes(file));   // load test*.dll
      var t = a.GetType(type, false, true);
      if (t != null)
      {
        var bfs = BindingFlags.Public | BindingFlags.IgnoreCase | BindingFlags.Instance;
        var m = t.GetMethod(method, bfs);
        var o = Activator.CreateInstance(t);
        if (m != null)
          ret += " - result: " + m.Invoke(o, new object[] { some });
      }
      return ret;
    }
  }
}
```
*> after some time the object seems to be either disconnected or/and disposed by the runtime/GC.*

the Form should hold a reference to the Proxy.
below is an example with GC.Collect().
```c# 
using System;
using System.CodeDom.Compiler;
using System.Drawing;
using System.IO;
using System.Reflection;
using System.Windows.Forms;
using Microsoft.CSharp;

namespace WindowsFormsApplication6
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.Size = new Size(500, 450);

      var tb = new RichTextBox
        {
          Parent = this,
          Dock = DockStyle.Fill,
          Text = "CurrentDomain: " + AppDomain.CurrentDomain.GetHashCode() + "\n"
        };
      CreateAssembly("test.dll", tb.AppendText);
      Proxy p = null;
      AppDomain ad = null;
      var index = 0;
      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("Create", (s, e) =>
        {
          if (ad == null)
          {
            ad = AppDomain.CreateDomain(String.Empty);
            p = Proxy.Create(ad);
          }
          tb.AppendText("\nAppDomain created.");
        });
      this.Menu.MenuItems.Add("Run", (s, e) =>
        {
          if (p == null || ad == null)
            tb.AppendText("\nAppDomain doesn't exist.");
          else
            tb.AppendText("\n" + p.Run("test.dll", "test", "run", index++));
        });
      this.Menu.MenuItems.Add("Unload", (s, e) =>
        {
          AppDomain.Unload(ad);
          p = null;
          ad = null;
          tb.AppendText("\nAppDomain Unload");
        });
      this.Menu.MenuItems.Add("GC", (s, e) =>
        {
          GC.Collect();
          GC.WaitForFullGCComplete();
          tb.AppendText("\nGC Complete");
        });
      this.FormClosing += delegate { File.Delete("test.dll"); };
    }

    void CreateAssembly(string fname, Action<string> callback)
    {
      var res = Compile(
        fname, new[] { "System.dll", "System.Windows.Forms.dll", "System.Drawing.dll" }, @"
        using System; 
        using System.Windows.Forms;
        using System.Drawing;
        public class Test {  
          public string Run(int data) { return String.Concat(data, ""; "", 
            Control.MousePosition); }}");
      foreach (CompilerError err in res)
          callback(err.ErrorText + Environment.NewLine);
    }

    CompilerErrorCollection Compile(string path, string[] assemblies, string cs)
    {
      var csp = new CSharpCodeProvider();
      var cps = new CompilerParameters();
      cps.ReferencedAssemblies.AddRange(assemblies);
      cps.GenerateInMemory = false;
      cps.OutputAssembly = path;
      return csp.CompileAssemblyFromSource(cps, cs).Errors;
    }
  }
  
  public class Proxy : MarshalByRefObject
  {
    public static Proxy Create(AppDomain ad)
    {
      var pt = typeof(Proxy);
      return ad.CreateInstanceAndUnwrap(pt.Assembly.FullName, pt.FullName) as Proxy;
    }

    public string Run(string file, string type, string method, int some)
    {
      var ret = "AppDomain: " + AppDomain.CurrentDomain.GetHashCode();
      var a = Assembly.Load(File.ReadAllBytes(file));   // load test*.dll
      var t = a.GetType(type, false, true);
      if (t != null)
      {
        var bfs = BindingFlags.Public | BindingFlags.IgnoreCase | BindingFlags.Instance;
        var m = t.GetMethod(method, bfs);
        var o = Activator.CreateInstance(t);
        if (m != null)
          ret += " - result: " + m.Invoke(o, new object[] { some });
      }
      return ret;
    }
  }
}
```
