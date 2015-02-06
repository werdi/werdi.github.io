---
title: Хук в C# на PrintScn
tags: [c#, wpf]
src: https://social.msdn.microsoft.com/Forums/ru-RU/88034682-d811-438f-9cca-f052e94b0e26/-c-printscn?forum=programminglanguageru
---
# Хук в C# на PrintScn
*> как будет выглядеть хук для приложения на C#, выполняющий при нажатии клавиши PrintScn определённые действия.*

см. "[Low-Level Keyboard Hook in C#] (http://blogs.msdn.com/b/toub/archive/2006/05/03/589423.aspx)" или "[Global Mouse and Keyboard Hooks in C#] (http://globalmousekeyhook.codeplex.com/)"
другой вариант на основе GetAsyncState:
```c#
KeyState.Wait(Key.PrintScreen, state =>
{
  System.Diagnostics.Trace.WriteLine(state, "changed");
});
```
см. [здесь] (http://social.msdn.microsoft.com/Forums/en/csharpgeneral/thread/7fc58f05-fccc-46ec-b58e-8755afb6e90f/#57de7967-b7ba-4cd0-8fe8-48fde41466dd)

*> KeyState.Wait(Key.PrintScreen, [...] можете на примере простенького консольного приложения показать ...*

[console app]

подключить сборки: PresentationCore.dll, PresentationFramework.dll, WindowsBase.dll
```c#
using System;
using System.Windows;
using System.Windows.Input;

namespace ConsoleApplication1
{
  class Program
  {
    static void Main(string[] args)
    {
      KeyState.Wait(Key.PrintScreen, state =>
      {
        Console.Beep();
        if (state == KeyStates.Down)
          MessageBox.Show("1");
      });

      Console.ReadKey();
    }
  }
}
```
[wpf app]
```c#
using System.Windows;
using System.Windows.Input;

namespace WpfApplication3
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      KeyState.Wait(Key.PrintScreen, state =>
      {
        this.Activate();
        if (state == KeyStates.Down)
          MessageBox.Show("1");
      });
    }
  }
}
```
