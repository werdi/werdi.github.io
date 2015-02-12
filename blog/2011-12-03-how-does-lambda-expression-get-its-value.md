---
title: How does this lambda expression get its value?
tags: [c#, winforms, linq, expression]
src: https://social.msdn.microsoft.com/Forums/ru-RU/51e96dc0-c519-40e3-9dab-509120fbd330/how-does-this-lambda-expression-get-its-value?forum=csharplanguage
---
# How does this lambda expression get its value?
*> NotifyPropertyChanged(p => QuantitySaved); [...] MvvmHelper.NotifyPropertyChanged(property, PropertyChanged); [...] where does the ViewModelBase come from at runtime?*

in general if you have to know about method, the best you can do is open assemly into .NET-assembly browser (such as ILSpy or Reflector) and take a look at the source code of the method.
i can't write much in english. so below is an example. hope it helps.
```c#
using System;
using System.ComponentModel;
using System.Diagnostics;
using System.Linq;
using System.Linq.Expressions;
using System.Windows.Forms;

namespace WindowsFormsApplication9
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var vm = new SampleViewModel();
      vm.PropertyChanged += (s, e) =>
      {
        var c = from f in new StackTrace(1).GetFrames()
                let m = f.GetMethod()
                where m.DeclaringType == typeof(SampleViewModel)
                select m;
        System.Diagnostics.Trace.WriteLine(e.PropertyName, c.First().Name);
      };
      vm.Test1();
      vm.Test2();
    }
  }

  public class ViewModelBase : INotifyPropertyChanged
  {
    public event PropertyChangedEventHandler PropertyChanged;

    protected virtual void NotifyPropertyChanged<T>(Expression<Func<ViewModelBase, T>> prop)
    {
      var eh = this.PropertyChanged;
      if (eh != null)
      {
        var name = (prop.Body as MemberExpression).Member.Name;
        eh(this, new PropertyChangedEventArgs(name));
      }
    }
  }

  public class SampleViewModel : ViewModelBase
  {
    public int Prop { get; set; }

    public void Test0()
    {
      this.NotifyPropertyChanged(p => this.Prop);
    }
    public void Test1()
    {
      // e.ToString() = "{p => value(WindowsFormsApplication9.SampleViewModel).Prop}"
      Expression<Func<ViewModelBase, int>> e = p => this.Prop;
      this.NotifyPropertyChanged(e);
    }
    public void Test2()
    {
      // e.ToString() = "p => value(WindowsFormsApplication9.SampleViewModel).Prop"
      var e = Expression.Lambda<Func<ViewModelBase, int>>(
                Expression.Property(Expression.Constant(this), "Prop"),
                Expression.Parameter(typeof(ViewModelBase), "p"));
      this.NotifyPropertyChanged(e);
    }
  }
}
```
*> how the ViewModelBase walks the tree (I assume at runtime?) to find the property name?*

there is no tree in your case. ViewModelBase at runtime just gets the Member.Name
```   
Expression<Func<ViewModelBase, T>> prop
...
var name = (prop.Body as MemberExpression).Member.Name;
``` 
the rest of the code is above. just run it in the debugger and you can see how it works.
