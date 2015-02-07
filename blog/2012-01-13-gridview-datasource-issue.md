---
title: GridView datasource issue
tags: [c#, winforms, data binding]
src: https://social.msdn.microsoft.com/Forums/ru-RU/c363b4c4-2238-4a4c-8ee5-84b7ecefa413/gridview-datasource-issue?forum=Offtopic 
---
# GridView datasource issue
*> I have a datagridview and I have populated it using a collection (list). [...] My requirement is to add or remove few rows from the grid.*
```c#
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Windows.Forms;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    class Info
    {
      public int Id { get; set; }
      public string Text { get; set; }
    }

    public Form1()
    {
      var bl = new BindingList<Info>();
      for (int i = 0; i < 3; i++)
          bl.Add(new Info { Id = i, Text = "text" + i });

      new DataGridView { Parent = this, Dock = DockStyle.Fill, 
                AllowUserToAddRows = false, DataSource = bl };
      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("Add", Add);
      this.Menu.MenuItems.Add("Remove", Remove);
      this.Menu.MenuItems.Add("Update", Update);
    }

    void Update(object sender, EventArgs e)
    {
      var dg = this.Controls[0] as DataGridView;
      var ci = dg.CurrentRow.DataBoundItem as Info;
      ci.Text += "!";
      dg.InvalidateRow(dg.CurrentRow.Index);
    }

    void Add(object sender, EventArgs e)
    {
      var dg = this.Controls[0] as DataGridView;
      var list = dg.DataSource as IList<Info>;
      list.Add(new Info { Id = list.Count, Text = "new" });
    }

    void Remove(object sender, EventArgs e)
    {
      var dg = this.Controls[0] as DataGridView;
      var ci = dg.CurrentRow.DataBoundItem as Info;
      var list = dg.DataSource as IList<Info>;
      list.Remove(ci);
    }
  }
}
```
