---
title: WPF. Существование объекта Window без объекта Application
tags: [c#, wpf, winforms, compiler]
src: https://social.msdn.microsoft.com/Forums/ru-RU/8a3d1ffb-183f-4b83-a9f8-1167cc0b8fc2/wpf-window-application?forum=programminglanguageru
---
# WPF. Существование объекта Window без объекта Application
*> MainWindow.Show(); [...] Окно появляется на мгновение, но затем исчезает, как же так получается?*

в windows существует очередь сообщений. см. [About Messages and Message Queues] (http://msdn.microsoft.com/en-us/library/windows/desktop/ms644927(v=vs.85).aspx)
чтобы окно не закрылось должен работать цикл обработки сообщений. его можно запустить с помощью Application.Run или Window.ShowDialog;
на реализацию методов можно посмотреть с помощь браузера сборок : [ILSpy] (http://wiki.sharpdevelop.net/ILSpy.ashx) или [Reflector] (http://ru.wikipedia.org/wiki/.NET_Reflector)

*> мне кажется Application такой объект, что без него даже старт приложения невозможен.*

ниже пример winforms-приложения, в котором создается, компилируется и 
запускается wpf-приложение в отдельном процессе.
используется ShowDialog.
```c#
using System;
using System.CodeDom.Compiler;
using System.Diagnostics;
using System.Windows.Forms;
using Microsoft.CSharp;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      Start("testwpf.exe");
    }

    void Start(string exe)
    {
      var dir = @"C:\Program Files\Reference Assemblies\Microsoft\Framework\v3.0\";
      var cps = new CompilerParameters
      {
        GenerateInMemory = true,
        CompilerOptions = @"/target:winexe",
        OutputAssembly = exe,
        GenerateExecutable = true
      };
      cps.ReferencedAssemblies.AddRange(new[] {
        dir+"PresentationCore.dll", 
        dir+"PresentationFramework.dll", 
        dir+"WindowsBase.dll",
        "System.dll"});
      var csp = new CSharpCodeProvider();
      var res = csp.CompileAssemblyFromSource(cps,
        @"using System;
          using System.Windows;
          using System.Windows.Controls;
          public static class Program { 
            [STAThread]
            static void Main() {
              var wnd = new Window() { Width = 500, Height=500 };
              wnd.Content = new Label { Content = ""WPF"", FontSize=50.0 };
              wnd.ShowDialog();    // или new Application().Run(wnd);
            }}");
      if (res.Errors.Count > 0)
        throw new Exception(res.Errors[0].ErrorText);
      Process.Start(exe);
    }
  }
}
```
