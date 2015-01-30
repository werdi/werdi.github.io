---
title: Многопоточность и блокировки в List(T)
tags: [list(T), c#, wcf, threading, winforms, data binding]
src: https://social.msdn.microsoft.com/Forums/ru-RU/33d5fac4-837e-4483-8e82-b8cca8b76358/-listt?forum=fordataru
---
# Многопоточность и блокировки в List(T)
*> класс HostState который содержит имя хоста и по методу Update() проверяет доступность хоста и обновляет данные статистики экземпляра. [...]  обновлять обязательно асинхронно*
```c#
using System;
using System.Data;
using System.Linq;
using System.Threading;
using System.Windows.Forms;

namespace WindowsFormsApplication1
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      // коллекция хостов
      var arr = Enumerable.Range(0, 10).Select(i => new HostState() { HostName = "host" + i });

      // для вывода в DataGridView
      var dt = new DataTable();
      dt.Columns.Add("host", typeof(string));
      dt.Columns.Add("count", typeof(int));
      dt.DefaultView.Sort = "host ASC";
      foreach (var h in arr)
        dt.Rows.Add(h.HostName, 0);
      var dg = new DataGridView
      {
        Dock = DockStyle.Fill,
        Parent = this,
        DataSource = dt.DefaultView,
        AllowUserToAddRows = false,
        AllowUserToDeleteRows = false,
        RowHeadersVisible = false,
        ReadOnly = true
      };
      Action<HostState, string> cb = (h, s) =>   // вызывается из !UI потока
      {
        var i = dt.DefaultView.Find(h.HostName);
        var dr = dt.DefaultView[i];
        // передать данные в UI поток
        dg.BeginInvoke(new Action<DataRowView>(r => r["count"] = (int)r["count"] + 1), dr);
      };

      // для остановки !UI потоков
      var end = false;
      this.FormClosing += (s, e) => end = true;

      // запуск потоков
      foreach (var hs in arr)
      {
        ThreadPool.QueueUserWorkItem(state =>
        {
          while (!end)
            ((HostState)state).Update(cb);
        }, hs);
      }
    }
  }

  class HostState
  {
    public string HostName { get; set; }
    public void Update(Action<HostState, string> callback = null)
    {
      var rnd = new Random();
      Thread.Sleep(rnd.Next(100, 500));      // test
      callback(this, "ok");
    }
  }
}
```
*> еще есть необходимость по запросу WCF передавать текущее состояние списка клиенту*

ниже пример, который создает "снимок" данных по таймеру (или можно по wcf-запросу).
```c#
using System;
using System.Data;
using System.Linq;
using System.Threading;
using System.Windows.Forms;

namespace WindowsFormsApplication1
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.Size = new System.Drawing.Size(600, 500);

      // коллекция хостов
      var arr = Enumerable.Range(0, 10).Select(i => new HostState() { HostName = "host" + i });

      // для остановки потоков
      var end = false;
      this.FormClosing += (s, e) => end = true;

      // для вывода в UI
      var dt = new DataTable();
      dt.Columns.Add("host", typeof(string));
      dt.Columns.Add("count", typeof(int));
      dt.DefaultView.Sort = "host ASC";
      foreach (var h in arr)
          dt.Rows.Add(h.HostName, 0);
      var dg = new DataGridView
      {
        Dock = DockStyle.Fill,
        Parent = this,
        DataSource = dt.DefaultView,
        AllowUserToAddRows = false,
        AllowUserToDeleteRows = false,
        RowHeadersVisible = false,
        ReadOnly = true
      };
      Action<HostState, string> cb = (h, s) =>   // вызывается из !UI потока
      {
        if(end) return;

        var i = dt.DefaultView.Find(h.HostName);
        var dr = dt.DefaultView[i];
        // передать данные в UI поток
        dg.BeginInvoke(new Action<DataRowView>(r => r["count"] = (int)r["count"] + 1), dr);
      };

      var tb = new RichTextBox { Parent = this, Dock = DockStyle.Right, Width = 250 };
      var t = new System.Windows.Forms.Timer() { Interval = 1000 };
      t.Tick += (s, e) =>
        tb.Text = "snapshot - " + DateTime.Now 
                    + "\n" + string.Join(";\n", dt.Rows.OfType<DataRow>().Select(r => string.Join("; ", r.ItemArray)));
      t.Start();

      // запуск потоков
      foreach (var hs in arr)
      {
        ThreadPool.QueueUserWorkItem(state =>
        {
          while (!end)
            ((HostState)state).Update(cb);
        }, hs);
      }
    }
  }

  class HostState
  {
    public string HostName { get; set; }
    public void Update(Action<HostState, string> callback = null)
    {
      var rnd = new Random();
      Thread.Sleep(rnd.Next(100, 500));      // test
      callback(this, "ok");
    }
  }
}
```
