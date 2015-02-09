---
title: How to delete empty cells of datatable
tags: [c#, winforms, data, threading]
src: https://social.msdn.microsoft.com/Forums/ru-RU/06d04ee2-a730-4617-846e-926c234d2484/how-to-delete-empty-cells-of-datatable?forum=csharpgeneral
---
# How to delete empty cells of datatable
*> I have a DataTable in C#. [...] What I want is deleting empty cells*

so if you need shrink columns, then below is an example. it works for any number of columns.
it uses Parallel Class for processing each columns. also it uses Barrier for synchronization.
(the shrinker2 below is a refactored and faster version of the shrinker1).
```c#
using System;
using System.Collections.Generic;
using System.Data;
using System.Diagnostics;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using System.Windows.Forms;

namespace WindowsFormsApplication9
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var dt = LoadData();

      this.Size = new System.Drawing.Size(800, 400);
      var dg = new DataGridView { 
        AutoSizeColumnsMode = DataGridViewAutoSizeColumnsMode.Fill,
        Parent = this, Dock = DockStyle.Fill, DataSource = dt };
      new DataGridView { 
        AutoSizeColumnsMode = DataGridViewAutoSizeColumnsMode.Fill,
        Parent = this, Dock = DockStyle.Right, Width = 250, DataSource = ((dynamic)Shrinker1.Run(dt)).Data };
      new DataGridView { 
        AutoSizeColumnsMode = DataGridViewAutoSizeColumnsMode.Fill,
        Parent = this, Dock = DockStyle.Right, Width = 250, DataSource = ((dynamic)Shrinker2.Run(dt)).Data };
      this.Shown += (s, e) => dg.FirstDisplayedScrollingRowIndex = 10000;
    }
    DataTable LoadData()
    {
      var dt = new DataTable();
      dt.Columns.Add("Col1", typeof(int));
      dt.Columns.Add("Col2", typeof(int));
      dt.Columns.Add("Col3", typeof(string));
      for (int i = 0; i < 10000; i++) dt.Rows.Add(null, null, null);
      dt.Rows.Add(1, null, "a");
      dt.Rows.Add(2, null, "b");
      dt.Rows.Add(null, 3, null);
      dt.Rows.Add(null, 4, null);
      dt.Rows.Add(5, 6, "c");
      dt.Rows.Add(7, null, "d");
      dt.Rows.Add(null, null, null);
      dt.Rows.Add(null, 8, null);
      dt.Rows.Add(null, 9, null);
      dt.Rows.Add(null, null, 10);
      for (int i = 0; i < 10000; i++) dt.Rows.Add(null, null, null);
      return dt;
    }
  }
  
  public static class Shrinker1
  {
    public static object Run(DataTable dt)
    {
      var sw = Stopwatch.StartNew();
      var infos = new Info[dt.Columns.Count];
      foreach (DataColumn dc in dt.Columns)
        infos[dc.Ordinal] = new Info { Ordinal = dc.Ordinal, Table = dt };

      var ret = dt.Clone();
      var mb = new Barrier(infos.Length, b =>
        ret.Rows.Add(infos.Select(inf => inf.Complete ? null : inf.Result).ToArray()));

      Parallel.ForEach(infos, info =>
      {
        while (info.Run()) mb.SignalAndWait();
        mb.RemoveParticipant();
      });
      sw.Stop();
      return new { Data = ret, ElapsedMilliseconds = sw.ElapsedMilliseconds };
    }
    class Info
    {
      public Info() { this.Position = -1; }
      public int Ordinal { get; set; }
      public DataTable Table { get; set; }
      public object Result { get; private set; }
      public int Position { get; private set; }
      public bool Complete { get; private set; }
      public bool Run()
      {
        if (this.Complete) return false;
        while (++this.Position < Table.Rows.Count)
        {
          var val = Table.Rows[this.Position][Ordinal];
          if (val != DBNull.Value)
          {
            this.Result = val;
            return true;
          }
        }
        return !(this.Complete = true);
      }
    }
  }
  
  public static class Shrinker2
  {
    public static object Run(DataTable dt)
    {
      var sw = Stopwatch.StartNew();
      var ret = dt.Clone();
      var result = new object[dt.Columns.Count];
      var mb = new Barrier(dt.Columns.Count, b =>
        {
          ret.Rows.Add(result);
          result = new object[dt.Columns.Count];
        });
      Parallel.ForEach(dt.Columns.OfType<DataColumn>(), dc =>
      {
        foreach (var val in Run(dt, dc.Ordinal))
        {
          result[dc.Ordinal] = val;
          mb.SignalAndWait();
        }
        mb.RemoveParticipant();
      });
      sw.Stop();
      return new { Data = ret, ElapsedMilliseconds = sw.ElapsedMilliseconds };
    }
    static IEnumerable<object> Run(DataTable dt, int ordinal)
    {
      for (int i = 0; i < dt.Rows.Count; i++)
      {
        var val = dt.Rows[i][ordinal];
        if (val != DBNull.Value)
          yield return val;
      }
    }
  }
}
```
