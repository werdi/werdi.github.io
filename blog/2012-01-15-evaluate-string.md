---
title: Расчёт строки из текстбокса
tags: [c#, xaml, wpf, winforms, compiler]
src: https://social.msdn.microsoft.com/Forums/ru-RU/b741c190-cbaa-4349-94a3-7ea112112ee4/-?forum=fordesktopru 
---
# Расчёт строки из текстбокса
*> Можно ли на C# сделать метод который посчитает строку : 458454/32*56+56*52-45 [...] и выведет это всё в Label*

можно с помощью CSharpCodeProvider:
```c#
using System;
using System.CodeDom.Compiler;
using System.Reflection;
using System.Windows.Forms;
using Microsoft.CSharp;
using System.Drawing;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.Padding = new Padding(10);
      var lbl = new Label { Parent = this, Dock = DockStyle.Top };
      var tb = new TextBox { Parent = this, Dock = DockStyle.Top, Text = "458454/32*56+56*52-45" };
      new Button { Parent = this, Location = new Point(10, 60), Text = "Eval" }
        .Click += delegate
        {
          try
          {
            lbl.Text = "Result: " + Eval(tb.Text);
          }
          catch (Exception exc)
          {
            lbl.Text = "Error: " + exc.Message;
          }
        };
    }

    object Eval(string cs)
    {
      var csp = new CSharpCodeProvider();
      var cps = new CompilerParameters { GenerateInMemory = true };
      var res = csp.CompileAssemblyFromSource(cps,
        @"public static class Some { public static object Run() { return " + cs + ";} }");
      if (res.Errors.Count > 0)
        throw new Exception(res.Errors[0].ErrorText);

      var t = res.CompiledAssembly.GetType("Some");
      var m = t.GetMethod("Run", BindingFlags.Static | BindingFlags.Public);
      return m.Invoke(null, null);
    }
  }
}
```
*> у меня WPF - много поменяется? [...] Т.е. в TextBox я могу занести любое сочетание 5+5 или 2*5+6+7/8 и т.п.*

пример для WPF:
(значение Label вычисляется сразу при изменении TextBox)
```xaml
<Window x:Class="WpfApplication1.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="500" Width="500" FontSize="16"
  xmlns:app="clr-namespace:WpfApplication1">
  <StackPanel>
    <TextBox x:Name="tb" Text="{Binding Code}" />
    <Label Content="{Binding ElementName=tb, Path=Text, Converter={app:Converter}}" />
  </StackPanel>
</Window>
```
```c#
using System.Windows;
using System.Xml.Linq;
using System.Windows.Data;
using System.Windows.Markup;
using Microsoft.CSharp;
using System.CodeDom.Compiler;
using System;
using System.Reflection;

namespace WpfApplication1
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      
      this.Code = "458454/32*56+56*52-45";
      this.DataContext = this;
    }

    public string Code { get; set; }
  }

  class Converter : MarkupExtension, IValueConverter
  {
    public static object Eval(string cs)
    {
      var csp = new CSharpCodeProvider();
      var cps = new CompilerParameters { GenerateInMemory = true };
      var res = csp.CompileAssemblyFromSource(cps,
        @"public static class Some { public static object Run() { return " + cs + ";} }");
      if (res.Errors.Count > 0)
        throw new Exception(res.Errors[0].ErrorText);

      var t = res.CompiledAssembly.GetType("Some");
      var m = t.GetMethod("Run", BindingFlags.Static | BindingFlags.Public);
      return m.Invoke(null, null);
    }

    public object Convert(object value, Type targetType, object parameter, System.Globalization.CultureInfo culture)
    {
      try
      {
        return Eval((string)value);
      }
      catch (Exception exc)
      {
        return exc.Message;
      }
    }

    public object ConvertBack(object value, Type targetType, object parameter, System.Globalization.CultureInfo culture)
    {
      throw new System.NotImplementedException();
    }

    public override object ProvideValue(IServiceProvider serviceProvider)
    {
      return this;
    }
  }
}
```
