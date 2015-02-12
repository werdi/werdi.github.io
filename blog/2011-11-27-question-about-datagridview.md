---
title: Вопрос по DataGridView
tags: [c#, winforms, binding]
src: https://social.msdn.microsoft.com/Forums/ru-RU/7e737d46-21b1-4cf0-a39b-e7942d4cb46b/-datagridview?forum=fordesktopru
---
# Вопрос по DataGridView
* Areas([Id],[RegionId],[Name]) и Regions([Id],[RegionName]). Соотношение многие к одному. [...] чтобы в DataGridView вместо RegionId отображался ComboBox со значениями RegionName из таблицы Regions?*
```c#
using System;
using System.Data;
using System.Drawing;
using System.Windows.Forms;

namespace WindowsFormsApplication4
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var ta = new DataTable("Areas");
      ta.Columns.Add("Id", typeof(int));
      ta.Columns.Add("RegionId", typeof(int));
      ta.Columns.Add("Name", typeof(string));

      var tr = new DataTable("Regions");
      tr.Columns.Add("Id", typeof(int));
      tr.Columns.Add("RegionName", typeof(string));

      var ds = new DataSet();
      ds.Tables.AddRange(new[] { tr, ta });
      ds.Relations.Add("Region_Areas", tr.Columns["id"], ta.Columns["RegionId"]);

      // test data
      var ai = 0;
      var rnd = new Random();
      for (var i = 0; i < 5; i++)
      {
        tr.Rows.Add(i, "region" + i);
        for (var j = 0; j < rnd.Next(2, 5); j++)
          ta.Rows.Add(ai++, i, "area" + j);
      }

      this.Size = new Size(700, 400);
      new DataGridView
      {
        Parent = this,
        Dock = DockStyle.Fill,
        AutoGenerateColumns = false,
        DataSource = ds,
        DataMember = "Areas",
      }
      .Columns.AddRange(
        new DataGridViewTextBoxColumn
        {
          HeaderText = "Areas",
          DataPropertyName = "Name"
        },
        new DataGridViewComboBoxColumn
        {
          HeaderText = "Region",
          FlatStyle = FlatStyle.Flat,
          DataSource = tr,
          DataPropertyName = "RegionId",
          DisplayMember = "RegionName",
          ValueMember = "Id"
        }
    	);
  	}
  }
}
```
