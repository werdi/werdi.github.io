---
title: Мониторинг процессов
tags: [c#, wmi, winforms, process]
src: https://social.msdn.microsoft.com/Forums/ru-RU/d5cfa002-a24e-4bb4-acbe-19c05f3b1b71/-?forum=fordesktopru 
---
# Мониторинг процессов
*> мониторинга процессов [...] как реализовать проверку какие новые процессы запущены. Тоесть я буду брать список раз в пару секунд и писать в лог мне нужно только те что появились.*

можно подключить обработчик к WMI событию.
```c#
var query = new WqlEventQuery("SELECT * FROM __InstanceCreationEvent WITHIN 1 WHERE TargetInstance ISA 'Win32_Process'");
var watcher = new ManagementEventWatcher(new ManagementScope(ManagementPath.DefaultPath), query);
watcher.EventArrived += (ws, we) => { /* ... */ };
```
полный пример [здесь] (http://social.msdn.microsoft.com/Forums/en-us/wpf/thread/6bc619b0-bb7f-481d-87a3-edd77ecd5dd8/#3a9b6ad5-1a8e-459f-8bf6-4d72e517c3c9).

*> Мне нужно именно сравнинеи двух List<String> и нахождения новых елементов.*

если надо выявлять новые процессы (были запущены после начала работы монитора процессов), то так:
```c#
using System;
using System.Collections.Concurrent;
using System.Diagnostics;
using System.Management;
using System.Windows.Forms;

namespace WindowsFormsApplication4
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.Width = 400;

      var tbl = new ConcurrentDictionary<string, Some>(StringComparer.OrdinalIgnoreCase);
      foreach (var p in Process.GetProcesses())
        tbl.TryAdd(p.ProcessName, new Some());

      var rtb = new RichTextBox { Parent = this, Dock = DockStyle.Fill };
      Action<string> notify = name => rtb.Text += "new process: " + name + "\n";

      var query = new WqlEventQuery(
        "SELECT * FROM __InstanceCreationEvent WITHIN 1 WHERE TargetInstance ISA 'Win32_Process'");
      var watcher = new ManagementEventWatcher(
        new ManagementScope(ManagementPath.DefaultPath), query);
      watcher.EventArrived += (s, e) =>
      {
        var mo = e.NewEvent["TargetInstance"] as ManagementBaseObject;
        var name = mo.GetPropertyValue("Name") as string;
        if (tbl.ContainsKey(name) == false)
          this.BeginInvoke(notify, name);
      };
      this.FormClosing += delegate { watcher.Stop(); };
      watcher.Start();
    }

    class Some
    {
      // ...
    }
  }
}
```
