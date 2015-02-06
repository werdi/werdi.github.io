---
title: How to make an object which will always work in a thread different than the UI thread?
tags: [c#, winforms, proxy, threading]
src: https://social.msdn.microsoft.com/Forums/ru-RU/9f1d787a-0586-4fd6-90e6-1e0a61d7f2b6/how-to-make-an-object-which-will-always-work-in-a-thread-different-than-the-ui-thread?forum=csharplanguage
---
# How to make an object which will always work in a thread different than the UI thread?
*> there is a better way to include the BackgroundWorker inside the Caller class, so that any method will be called from the Caller object will be automatically in different thread.*

try to use TPL, Task Class.
below is an example.
```c#
using System;
using System.Windows.Forms;
using System.Threading.Tasks;
using System.Threading;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    public static void Trace(object o)
    {
      System.Diagnostics.Trace.WriteLine(o, Thread.CurrentThread.ManagedThreadId.ToString());
    }

    public Form1()
    {
      var tst = new Test();
      Trace("Test created");

      // start Method1
      Task.Factory.StartNew(() => tst.Method1());
      // start Method2
      Task.Factory
        .StartNew(() => tst.Method2())
        .ContinueWith(t => Trace("Result=" + t.Result));
    }
  }

  public class Test
  {
    public void Method1()
    {
      Form1.Trace("Test.Method1");
    }
    public int Method2()
    {
      Form1.Trace("Test.Method2");
      return 1;
    }
  }
}
```
*> If I understand he wants to do something like [...] have Method1 and Method2 auto-magically run in a new thread*

some 'magic' is possible with RealProxy Class.
below is an example.
```c#
using System;
using System.Reflection;
using System.Runtime.Remoting.Messaging;
using System.Runtime.Remoting.Proxies;
using System.Threading;
using System.Threading.Tasks;
using System.Windows.Forms;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      Form1.Trace("Test created");
      var t = Proxy.CreateProxy<ITest>(new Test());
      t.Method1();
    }

    public static void Trace(object o)
    {
      System.Diagnostics.Trace.WriteLine(o, Thread.CurrentThread.ManagedThreadId.ToString());
    }
  }


  interface ITest
  {
    void Method1();
  }
  
  public class Test : ITest
  {
    public void Method1()
    {
      Form1.Trace("Test.Method1");
    }
  }


  public sealed class Proxy : RealProxy
  {
    public object Instance { get; private set; }
    
    public static T CreateProxy<T>(object instance)
    {
      var p = new Proxy(typeof(T)) { Instance = instance };
      return (T)p.GetTransparentProxy();
    }
    
    private Proxy(Type type) : base(type) { }
    
    public override IMessage Invoke(IMessage msg)
    {
      var mc = new MethodCallMessageWrapper((IMethodCallMessage)msg);
      var method = mc.MethodBase as MethodInfo;
      object res = null;
      var args = ((method.Attributes & MethodAttributes.SpecialName) == MethodAttributes.SpecialName)
        ? mc.InArgs : mc.Args;
      Task.Factory.StartNew(() => method.Invoke(this.Instance, args));
      return new ReturnMessage(res, mc.Args, mc.Args.Length, mc.LogicalCallContext, mc);
    }
  }
}
```
