---
title: Данные в таблице не обновляются на экране
tags: [c#, winforms, async, data]
src: https://social.msdn.microsoft.com/Forums/ru-RU/847b4c39-87fc-4c25-a42e-4d183613c042/-?forum=programminglanguageru
---
# Данные в таблице не обновляются на экране
*> Message "Недопустимая операция в нескольких потоках ...*

т.к. DataTable привязан к DataGridView, то вызов DataTable.Rows.Add(DataRow) выполнять в основном потоке. 
пример добавления строк в DataTable из отдельного потока:
```c#
using System;
using System.ComponentModel;
using System.Data;
using System.Threading;
using System.Threading.Tasks;
using System.Windows.Forms;

namespace WindowsFormsApplication1
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var dt = new DataTable();
      dt.Columns.Add("Тип", typeof(string));
      dt.Columns.Add("Дата", typeof(DateTime));
      dt.Columns.Add("Сообщение", typeof(string));

      // создать отдельный поток
      Task.Factory.StartNew(new Action<object>(Run), AsyncOperationManager.CreateOperation(dt));

      this.Size = new System.Drawing.Size(600, 500);
      new DataGridView { Parent = this, Dock = DockStyle.Fill, DataSource = dt, AllowUserToAddRows=false };
    }

    void Run(object state)
    {
      var ao = state as AsyncOperation;
      var dt = ao.UserSuppliedState as DataTable;
      SendOrPostCallback add = nr => // выполняется в потоке, в котором создан AsyncOperation
        dt.Rows.Add((DataRow)nr);

      for (int i = 0; i < 1000; i++)
      {
        var dr = dt.NewRow();
        dr["Тип"] = "t" + i;
        dr["Дата"] = DateTime.Now;
        dr["Сообщение"] = "msg" + i;
        ao.Post(add, dr);   // передать dr в основной поток

        Thread.Sleep(500);  // только для теста
      }
    }
  }
}
```
