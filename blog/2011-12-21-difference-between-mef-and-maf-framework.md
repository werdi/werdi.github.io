---
title: What is the difference between MEF and MAF framework
tags: [c#, winforms, compiler, interop]
src: https://social.msdn.microsoft.com/Forums/ru-RU/cda7966e-9e81-4242-b613-129be6018f19/what-is-the-difference-between-mef-and-maf-framework?forum=wpf
---
# What is the difference between MEF and MAF framework
*> We are making host application, which will load 10000-20000 of UI assemblies. In such case we need to support loading and unloading of UI assemblies in host application.*

below is an example, which creates assembly from source code,  creates AppDomain, then loads created assembly and shows WPF-window.
it should be noted that AppDomain.Unload is fast enough. but first Run after CreateDomain is slower than the rest.  
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
      var tb = new RichTextBox { Parent = this, Dock = DockStyle.Fill };
      this.Menu = new MainMenu();
      AppDomain ad = null;
      Proxy p = null;
      CreateAssembly("test.dll", tb.AppendText);
      this.Menu.MenuItems.Add("CreateAndRun", (s, e) =>
      {
        if (ad == null || p == null)
        {
          ad = AppDomain.CreateDomain(String.Empty);
          p = Proxy.Create(ad);
          tb.AppendText("\nCreated");
        }
        tb.AppendText("\nRun");
        p.Run("test.dll");
      });
      this.Menu.MenuItems.Add("Unload", (s, e) =>
      {
        tb.AppendText("\nUnload");
        p = null;
        if (ad != null)
          AppDomain.Unload(ad);
        ad = null;
      });
      this.FormClosing += delegate { File.Delete("test.dll"); };
    }

    void CreateAssembly(string fname, Action<string> callback)
    {
        var path = 
          @"C:\Program Files\Reference Assemblies\Microsoft\Framework\.NETFramework\v4.0\Profile\Client\";
        var assemlies = new[] { 
              "System.dll",  path+"System.Xaml.dll", path+"WindowsBase.dll", 
              path+"PresentationFramework.dll",  path+"PresentationCore.dll"
            };
        var cs = @"
          public class Test {
              public static void Run() {
                  var w = new System.Windows.Window() { Width=500.0 };
                  w.Content = new System.Windows.Controls.TextBlock { Text = ""hello"" };
                  w.ShowDialog(); }}";
        var res = Compile(fname, assemlies, cs);
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
      return (Proxy)ad.CreateInstanceAndUnwrap(pt.Assembly.FullName, pt.FullName);
    }
    public void Run(string file)
    {
      var a = Assembly.Load(File.ReadAllBytes(file));  // todo 
      var t = a.GetType("Test", false, true);
      var bf = BindingFlags.Public | BindingFlags.IgnoreCase | BindingFlags.Static;
      t.GetMethod("Run", bf).Invoke(null, null);
    }
  }
}
```