---
title: Автозапускатор
tags: [c#, xaml, wpf, winforms, wmi]
src: https://social.msdn.microsoft.com/Forums/ru-RU/56b6da1d-e3af-493d-858e-86902225bda4?forum=fordesktopru
---
# Автозапускатор
*> как проследить за процессом, то есть узнать работает он или нет?*

см. [Process.GetProcessesByName](http://msdn.microsoft.com/en-us/library/z3w4xdc9.aspx)

*> и еще вопрос он постоянно проверяет или надо водить wait?*

если требуется монитор процессов, то можно спользовать WMI. 
примерно так.
```c#
using System;
using System.Management;
using System.Windows;

namespace WpfApplication6
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();

      Action<string> cb = pid => 
        rtb.AppendText(pid + Environment.NewLine);

      var query = new WqlEventQuery(
        "SELECT * FROM __InstanceCreationEvent WITHIN 1 WHERE TargetInstance ISA 'Win32_Process'");
      var watcher = new ManagementEventWatcher(
        new ManagementScope(ManagementPath.DefaultPath), query);
      watcher.EventArrived += (ws, we) =>
      {
        var mo = we.NewEvent["TargetInstance"] as ManagementBaseObject;
        var pid = Convert.ToString(mo.GetPropertyValue("ProcessId"));
        this.Dispatcher.BeginInvoke(cb, pid);
      };
      watcher.Start();
      this.Closing += (ws, we) => watcher.Stop();
    }
    private void Button_Click(object sender, RoutedEventArgs e)
    {
      System.Diagnostics.Process.Start("notepad");
    }
  }
}
```
```xaml
<Window x:Class="WpfApplication6.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="350" Width="525"
  Topmost="True">
  <StackPanel>
    <Button Content="Run Notepad" Click="Button_Click" HorizontalAlignment="Left" />
    <RichTextBox x:Name="rtb" />
  </StackPanel>
</Window>
```
свойства Win32_Process см. [здесь](http://msdn.microsoft.com/en-us/library/windows/desktop/aa394372(v=vs.85).aspx)

*> if (procces.worked"ser"){ ...*
```c#
using System.Diagnostics;
using System.Drawing;
using System.Windows.Forms;

namespace WindowsFormsApplication0
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      new Button { Parent = this, Text = "Start", Location = new Point(10, 40) }
        .Click += (s, e) =>
        {
          var ps = Process.GetProcessesByName("Notepad");
          if (ps.Length == 0)
            Process.Start("Notepad");
        };
    }
  }
}
```
*> (ps.Length == 0) // Как понял если переменная не работает*

ps - список процессов с именем Notepad. ps.Length - количество элементов в списке. 
== 0 - нет запущенных процессов.

*> вызова функции Process.GetProcessesByName, например раз в секунду. [...] а как?*
```c#
using System.Diagnostics;
using System.Drawing;
using System.Windows.Forms;

namespace WindowsFormsApplication0
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      new Button { Parent = this, Text = "Start", Location = new Point(10, 40) }
        .Click += (s, e) =>
        {
          if (IsActive("Notepad") == false)
            Process.Start("Notepad");
        };

      var tb = new TextBox { Parent = this, Dock = DockStyle.Top };

      new Timer { Interval = 1000, Enabled = true }
        .Tick += (s, e) =>
        {
          tb.Text = this.IsActive("Notepad") ? "работает" : "";
        };
    }

    bool IsActive(string pname)
    {
      var ps = Process.GetProcessesByName(pname);
      return (ps.Length > 0);
    }
  }
}
```