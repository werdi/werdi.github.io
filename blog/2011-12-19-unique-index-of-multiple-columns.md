---
title: How To Create a unique index of multiple columns with code first
tags: [c#, winforms, data]
src: https://social.msdn.microsoft.com/Forums/ru-RU/9eb7e8a7-9cf3-43bd-ad53-d38a0f61fe83/how-to-create-a-unique-index-of-multiple-columns-with-code-first?forum=adodotnetentityframework
---
# How To Create a unique index of multiple columns with code first
*> How To Create a unique index of multiple columns with code first*

override method OnModelCreating and from HasKey return new {Key1=..., Key2=...}
below is an example.
```c#
using System;
using System.Data.Entity;
using System.Linq;
using System.Windows.Forms;

namespace WindowsFormsApplication5
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var ds = new DataStore();
      if (ds.Database.CreateIfNotExists())
      {
        ds.Items.Add(new Item { Name = "n1", Surname = "s1" });
        ds.Items.Add(new Item { Name = "n1", Surname = "s2" });
        ds.SaveChanges();
      }

      this.Width = 500;
      new DataGridView { Parent = this, Dock = DockStyle.Fill, DataSource = ds.Items.ToList() };

      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("Test", (s, e) =>
        {
          try
          {
            ds.Items.Add(new Item { Name = "n1", Surname = "s1" });
            ds.SaveChanges();
          }
          catch (Exception exc)
          {
            MessageBox.Show(exc.ToString());
          }
        });
    }
	}

	public class Item
	{
    public string Name { get; set; }
    public string Surname { get; set; }

    public string Note { get; set; }
	}

	public class DataStore : DbContext
	{
    public DbSet<Item> Items { get; set; }
    protected override void OnModelCreating(DbModelBuilder m)
    {
      base.OnModelCreating(m);
      m.Entity<Item>()
       .HasKey(e => new { Name = e.Name, Surname = e.Surname })
       .ToTable("Items");
    }
  }
}
```