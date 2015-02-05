---
title: Style for DataGridTextColumn
tags: [c#, datagrid, winforms]
src: https://social.msdn.microsoft.com/Forums/ru-RU/e23068de-dbb6-4744-9775-51993095b2a2/style-for-datagridtextcolumn?forum=csharpgeneral
---
# Style for DataGridTextColumn
*> how I can set style for DataGridTexcolumn Element style and editing element style*

use DataGridViewColumn.DefaultCellStyle
below is and example.
```c#
using System.Data;
using System.Drawing;
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

      var dg = new DataGridView { Dock = DockStyle.Fill, Parent = this, DataSource = dt };

      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("Test", delegate
      {
        dg.Columns[0].DefaultCellStyle.BackColor = Color.Silver;
      });
    }
  }
}
```
