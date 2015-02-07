---
title: Проблема с биндингом [WinForms]
tags: [c#, winforms, binding]
src: https://social.msdn.microsoft.com/Forums/ru-RU/a9f261b7-9f36-455a-942e-1225a077a4a4/-winforms?forum=fordesktopru
---
# Проблема с биндингом [WinForms]
*> Проблема в том, что биндинг к свойству Text не работает. При изменении свойства Text, прибинденные контролы не обновляются.*

надо реализовать в классе интерфейс INotifyPropertyChanged.
и в setter также добавить вызов: 
this.PropertyChanged(this, new PropertyChangedEventArgs("Text"));

другой способ: менять значение свойства через PropertyDescriptor.
чтобы биндинг получил сигнал о изменении значения.
```c#
using System;
using System.ComponentModel;
using System.Windows.Forms;

namespace WindowsFormsApplication5
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var f = new Foo();
      new TextBox { Parent = this, Dock = DockStyle.Top }
        .DataBindings.Add("Text", f, "Text");

      new Button { Parent = this, Top = 50, Text = "Test" }
          .Click += (s, e) =>
            TypeDescriptor.GetProperties(f)["Text"].SetValue(f, DateTime.Now.Ticks.ToString());
    }
  }

  public class Foo
  {
    public string Text { get; set; }
  }
}
```
*> реализация INotifyPropertyChanged не помогает.*

в таком случае можно определить событие TextChanged, как реализацию интерфейса.
```c#
using System;
using System.ComponentModel;
using System.Windows.Forms;

namespace WindowsFormsApplication5
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      IFoo f = new Foo() { Text = "test" };
      f.TextChanged += (s, e) =>
        System.Diagnostics.Trace.WriteLine("TextChanged");

      this.Width = 500;
      new TextBox { Parent = this, Dock = DockStyle.Top }
        .DataBindings.Add("Text", f, "Text");

      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("Test", (s, e) => f.Text += "!");
    }
  }
  
  public class MyEventArgs : EventArgs { }
  
  public interface IFoo
  {
    event EventHandler<MyEventArgs> TextChanged;
    string Text { get; set; }
  }
  
  public class Foo : INotifyPropertyChanged, IFoo
  {
    public event PropertyChangedEventHandler PropertyChanged = delegate { };

    public string Text
    {
      get { return _Text; }
      set
      {
        _Text = value;
        this.PropertyChanged(this, new PropertyChangedEventArgs("Text"));
        _TextChanged(this, new MyEventArgs());
      }
    }
    string _Text;

    event EventHandler<MyEventArgs> IFoo.TextChanged
    {
      add { _TextChanged = 
              Delegate.Combine(_TextChanged, value) as EventHandler<MyEventArgs>; }
      remove { _TextChanged =  
              Delegate.Remove(_TextChanged, value) as EventHandler<MyEventArgs>; }
    }
    EventHandler<MyEventArgs> _TextChanged = delegate { };
  }
}
```
*> Получается из Foo исчезнет событие TextChanged. А этот Foo уже давно в продакшене. Что я скажу юзерам? Извините, но теперь событие TextChanged только через интерфейс IFoo? Да и в дизайн моде его не будет видно совсем...*

можно с помощью TestDescriptionProvider подключить свой [CustomTypeDescriptor] (http://msdn.microsoft.com/ru-ru/library/system.componentmodel.customtypedescriptor.aspx), см.: GetEvents, GetProperties. пример подключения [здесь] (http://social.msdn.microsoft.com/Forums/ru-RU/programminglanguageru/thread/3c68579f-a86f-4a9e-abf7-38e239907625/#dc739281-aae4-4b6d-bc8e-c6314ca931ba).
как вариант: использовать [Mono.Cecil] (http://www.mono-project.com/Cecil) (для инъекции кода в сборки); 
или для привязок создавать обертки с помощью кодогенератора T4 (Text Templates; .tt)
