---
title: How to use table in C#
tags: [c#, winforms, ssce, sql server]
src: https://social.msdn.microsoft.com/Forums/ru-RU/e90e1c71-aa62-432f-b538-4d3fae9d3d00/how-to-use-table-in-c?forum=csharpgeneral
---
# How to use table in C#
*> I have three column customerid, itemname ,rate i want to use table to take entry from user. and then insert them all in the sql database.*

try using the example below. it generates a database automatically based on the Item class; lets you edit data in datagridview and saves the data when the form is closed.

for more infomation about EF, see [Code-First Development] (http://weblogs.asp.net/scottgu/archive/2010/07/16/code-first-development-with-entity-framework-4.aspx).
```c#
using System.Data.Entity;
using System.Data.SqlServerCe;
using System.Windows.Forms;

namespace WindowsFormsApplication5
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var ds = new DataStore();
      ds.Items.Load();

      this.Size = new System.Drawing.Size(600, 500);
      new DataGridView
      {
        Parent = this,
        Dock = DockStyle.Fill,
        DataSource = ds.Items.Local.ToBindingList()
      };

      this.FormClosing += (s, e) => ds.SaveChanges();
    }
  }

  public class Item
  {
    public int CustomerId { get; set; }
    public string Name { get; set; }
    public int Rate { get; set; }
  }

  public class DataStore : DbContext
  {
    public DbSet<Item> Items { get; set; }
    protected override void OnModelCreating(DbModelBuilder m)
    {
      base.OnModelCreating(m);
      m.Entity<Item>().HasKey(e => e.CustomerId).ToTable("Items");
    }
  }
}
```
if you need edit data in controls like RichTextBox, etc., 
then take a look at the [example] (http://social.msdn.microsoft.com/Forums/en/winformsdatacontrols/thread/d78609b8-94df-4988-88ba-08199087a746/#92fd742b-6a88-49ed-9483-d5d485887183)
