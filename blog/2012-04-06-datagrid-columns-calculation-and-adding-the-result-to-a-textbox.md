---
title: Как посчитать сумму столбца из таблицы в DataGrid и занести результат в textbox?
tags: [c#, winforms, data binding]
src: https://social.msdn.microsoft.com/Forums/ru-RU/88b19331-c795-4564-a5fa-5867f6443fd0/-datagrid-textbox?forum=fordesktopru
---
# Как посчитать сумму столбца из таблицы в DataGrid и занести результат в textbox?
*> Как посчитать сумму столбца из таблицы в DataGrid и занести результат в textbox?*
```c#
var sum = 0;
foreach(DataRow row in mytable.Rows)
  sum += (int)row["mycolumn1"];

textBox.Text = ""+sum;
```

другой вариант - использовать DataBinding для TextBox.
ниже пример привязки к объекту, который считает сумму столбца:
 ```c#
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Windows.Forms;

namespace WindowsFormsApplication1
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var dt = new DataTable();
      dt.Columns.Add("v1", typeof(int));
      dt.Columns.Add("v2", typeof(int));
      dt.Columns.Add("v3", typeof(int), "v1+v2");
      // test data
      for (int i = 0; i < 10; i++)
        dt.Rows.Add(i, i, i);

      this.Size = new Size(500, 400);
      new DataGridView { Dock = DockStyle.Fill, Parent = this, DataSource = dt };
      new TextBox { Dock = DockStyle.Bottom, Parent = this }
        .DataBindings.Add("Text", new Sum(dt, "v3"), "Value");
    }
  }

  class Sum : INotifyPropertyChanged
  {
    public event PropertyChangedEventHandler PropertyChanged = delegate { };

    public Sum(DataTable dt, string col)
    {
      dt.ColumnChanged += (s, e) =>
      {
        var sum = 0;
        foreach (DataRow row in dt.Rows)
          sum += (int)row[col];

        this.Value = sum;
        this.PropertyChanged(this, new PropertyChangedEventArgs("Value"));
      };
    }

    public int Value { get; protected set; }
  }
}
```
