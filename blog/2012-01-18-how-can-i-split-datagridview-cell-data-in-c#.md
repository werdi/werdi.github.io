---
title: How can i split DataGridView cell data in c#
tags: [c#, winforms]
src: https://social.msdn.microsoft.com/Forums/ru-RU/10d8b618-021b-41d7-8fa5-bfe22e9b94f8/how-can-i-split-datagridview-cell-data-in-c?forum=winformsdatacontrols
---
# How can i split DataGridView cell data in c#
*> i want to split 5 th column data i.e string of datagridview by '.'*

you can handle CellPainting event and draw any string.
below is an example.
```c# 
using System.Data;
using System.Drawing;
using System.Text.RegularExpressions;
using System.Windows.Forms;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var dt = new DataTable();
      dt.Columns.Add("text", typeof(string));
      dt.Rows.Add("test. test");
      dt.Rows.Add("test");

      var dg = new DataGridView { 
        Parent = this, Dock = DockStyle.Fill, DataSource = dt, AllowUserToAddRows = false };
      dg.RowTemplate.Height = 50;
      dg.CellPainting += (s, e) =>
      {
        var str = (e.FormattedValue as string);
        if (str != null && str.Contains("."))
        {
          e.PaintBackground(e.CellBounds, true);
          str = Regex.Replace(str, @"\.\s*", "\n");
          e.Graphics.DrawString(str, dg.Font, Brushes.Black, e.CellBounds);
          e.Handled = true;
        }
      };
    }
  }
}
```
*> it can not work with following code. If any changes please tell me.*

try to temporarily remove the following part of your code: "List<string> pageno = [...] dTable.AcceptChanges();" and instead of "DataSource = dTable" use "DataSource = dt".

another approach is to use CellFormatting and WrapMode instead of CellPainting.
below is an example
```c#
using System.Data;
using System.Text.RegularExpressions;
using System.Windows.Forms;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var dt = new DataTable();
      dt.Columns.Add("text", typeof(string));
      dt.Rows.Add("test. test");
      dt.Rows.Add("test");

      var dg = new DataGridView
      {
        Parent = this,
        Dock = DockStyle.Fill,
        DataSource = dt,
        AllowUserToAddRows = false
      };
      dg.RowTemplate.Height = 50;
      dg.DefaultCellStyle.WrapMode = DataGridViewTriState.True;
      dg.CellFormatting += (s, e) =>
      {
        var str = e.Value as string;
        if(str != null)
        {
          e.Value = Regex.Replace(str, @"\.\s*", "\n");
          e.FormattingApplied = true;
        }
      };
    }
  }
}
```
