---
title: Added Data Grid View Column won't sort
tags: [c#, winforms, xml, binding]
src: https://social.msdn.microsoft.com/Forums/ru-RU/70cdda5b-05bf-46e6-a9f1-211fca2b467d/added-data-grid-view-column-wont-sort?forum=winforms
---
# Added Data Grid View Column won't sort
*> I have a dataGridview that is boung to a dataTable. [...]  a DataGridViewColumn [...]  I cannot sort on this column and I cannot figure out why this is so.*

try using following example. the sorting works fine.
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
      var ds = new DataSet();
      ds.ReadXml(new StringReader(@"
        <data>
          <item id='1' data='d1' />
          <item id='2' data='d2' />
          <name text='d1' />
          <name text='d2' />
          <name text='d3' />
        </data>"));

      this.Height = 500;
      new DataGridView
      {
        Parent = this,
        Dock = DockStyle.Fill,
        AutoGenerateColumns = false,
        DataSource = ds,
        DataMember = "item"
      }
      .Columns.AddRange(
        new DataGridViewTextBoxColumn { DataPropertyName = "id", HeaderText = "id" },
        new DataGridViewComboBoxColumn
        {
          HeaderText = "data",
          DataPropertyName = "data",
          DataSource = ds,
          DisplayMember = "name.text",
          FlatStyle = FlatStyle.Flat,
          DisplayStyleForCurrentCellOnly = true,
          SortMode = DataGridViewColumnSortMode.Automatic
        }
      );
    }
  }
}
```