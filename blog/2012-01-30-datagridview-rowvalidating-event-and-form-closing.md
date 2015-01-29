---
title: DataGridView RowValidating event and form closing issue
tags: [c#, datagridview]
src: https://social.msdn.microsoft.com/Forums/ru-RU/6eda62ed-742f-46ec-9812-bf4c4c71540f/datagridview-rowvalidating-event-and-form-closing-issue?forum=winformsdatacontrols
---
# DataGridView RowValidating event and form closing issue
*> tries to close the form, the form does not close and the red icon appears [...] How can we remedy this problem without asking user to first fill in some values*
try to handle FormClosing Event and call DataGridView.CancelEdit Method.
below is an example.
```c#
using System.Data;
using System.Windows.Forms;

namespace WindowsFormsApplication2
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var dt = new DataTable();
      dt.Columns.Add("Id", typeof(int));
      dt.Columns.Add("Value", typeof(string)).AllowDBNull = false;
      for (int i = 0; i < 5; i++) dt.Rows.Add(i, "value" + i);

      var dg = new DataGridView { 
        Dock = DockStyle.Fill, Parent = this, DataSource = dt };
      dg.DataError += (s, e) =>  System.Diagnostics.Trace.WriteLine(e);

      this.FormClosing += (s, e) => 
      { 
        if(dg.IsCurrentCellInEditMode)
        {
          dg.CancelEdit();
          this.Close();
        }
      };
    }
  }
}
```
*> I had already tried the Form_closing event. This event actually doesn't fire until DataGridView.RowValidating event that doesn't allow you to leave the new row until filling in the values there.*
you can ignore RowValidating. 
try using the following example.
```c#
using System;
using System.Data;
using System.Threading;
using System.Windows.Forms;

namespace WindowsFormsApplication2
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var dt = new DataTable();
      dt.Columns.Add("Id", typeof(int));
      dt.Columns.Add("Value", typeof(string));
      for (int i = 0; i < 5; i++) dt.Rows.Add(i, "value" + i);

      var dg = new DataGridView
      {
        Dock = DockStyle.Fill,
        Parent = this,
        DataSource = dt
      };
      dg.DataError += (s, e) => System.Diagnostics.Trace.WriteLine(e.Exception.Message);

      var closing = false;
      dg.RowValidating += (s, e) =>
      {
        if (closing) return;

        if (String.IsNullOrWhiteSpace(dg.Rows[e.RowIndex].Cells["Value"].Value.ToString()))
          e.Cancel = true;
      };

      this.FormClosing += (s, e) =>
      {
        closing = true;
        ThreadPool.QueueUserWorkItem(_ => { Thread.Sleep(250); Application.Exit(); });
      };

      this.Shown += (s, e) =>
      {
        dg.CurrentCell = dg.Rows[2].Cells["Value"];
        SendKeys.SendWait(" ");
      };
    }
  }
}
```
