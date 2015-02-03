---
title: DataGridView
tags: [c#, winforms]
src: https://social.msdn.microsoft.com/Forums/ru-RU/88070817-49ee-405d-b3f9-3a8738c75de0/-?forum=fordataru
---
# DataGridView
*> filterDataGridView1.DataSource = dataSet.Tables[0];  [...] want to copy the data source of gridview to the data table*

there is no need to copy data or anything like that. simply use data binding.
below is an example
```c#
using System.Data;
using System.IO;
using System.Windows.Forms;

namespace WindowsFormsApplication4
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.Size = new System.Drawing.Size(600, 400);
      var sc = new SplitContainer { Parent = this, Dock = DockStyle.Fill, SplitterDistance = 300 };
      var ds = LoadData();

      var gv = new DataGridView
      {
        Dock = DockStyle.Fill,
        Parent = sc.Panel1,
        DataSource = ds,
        DataMember = "item",
        AutoSizeColumnsMode = DataGridViewAutoSizeColumnsMode.Fill,
      };

      var dg = new DataGrid { Parent = sc.Panel2, Dock = DockStyle.Fill, DataSource = ds };
      this.Shown += (s, e) =>
      {
        dg.Focus();
        SendKeys.Send("%({DOWN})");
      };
    }

    DataSet LoadData()
    {
      var ds = new DataSet();
      ds.ReadXml(new StringReader(@"
        <data>
          <item name='itm1'><child name='c1' /></item>
        </data>
      "));
      return ds;
    }
  }
}
```
