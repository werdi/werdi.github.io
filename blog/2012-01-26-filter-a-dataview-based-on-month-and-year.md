---
title: Filter a DataView based on Month and Year ONLY
tags: [c#, winforms, data]
src: https://social.msdn.microsoft.com/Forums/ru-RU/1d5c3f19-fab1-4af9-9d57-d4b26b15841b/filter-a-dataview-based-on-month-and-year-only?forum=adodotnetdataset
---
# Filter a DataView based on Month and Year ONLY
*> it has a DateTime DataColumn with data in default datetime format (e.g. 12/31/2011 12:00:00 AM) i'd like to set a RowFilter property on the DataView that will filter the rows*
```c#
using System;
using System.Windows.Forms;
using System.Data;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var dt = new DataTable();
      dt.Columns.Add("date", typeof(DateTime));
      for (int i = 0; i < 10; i++)
        dt.Rows.Add(DateTime.Now.AddDays(i));

      new DataGridView {
        Parent = this,
        Dock = DockStyle.Fill,
        DataSource = dt,
        AllowUserToDeleteRows = false,
        AutoSizeColumnsMode = DataGridViewAutoSizeColumnsMode.Fill
      };

      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("filter", (s, e) =>
      {
        dt.DefaultView.RowFilter =
          "date > #" + DateTime.Now.AddDays(5).ToShortDateString() + "#";
      });
      this.Menu.MenuItems.Add("all", (s, e) => dt.DefaultView.RowFilter = "");
    }
  }
}
```
*> but where is the filtering taking place in terms of Months and year only ?*

for the year 2012:
```
"Convert(date, 'System.String') LIKE '*/2012 *'";
```
for the month 1:
```
"Convert(date, 'System.String') LIKE '1/*'";
```
note: "Wildcard characters are not allowed in the middle of a string." [...] (http://msdn.microsoft.com/ru-ru/library/system.data.datacolumn.expression.aspx)
