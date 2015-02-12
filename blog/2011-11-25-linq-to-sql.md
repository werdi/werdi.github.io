---
title: Linq to sql
tags: [c#, winforms, linq, data]
src: https://social.msdn.microsoft.com/Forums/ru-RU/968f9ba9-4a1f-4979-a228-8aafe0bd54da/linq-to-sql?forum=linqtosql
---
# Linq to sql
*> Using linq to sql, I would like to know how to: a. Find out what the minimum and maximum date values are in the receive date column in a sql server 2008 r2 database. The receive date column is used as the key to the table.*

the code below creates a database with the dates table 
and requests min and max date
```c#
using System;
using System.Data.Entity;
using System.Linq;
using System.Windows.Forms;

namespace WindowsFormsApplication4
{
  public partial class Form1 : Form
  {
    public Form1()
    {  
      var d = new DataStore();
      // creates database: WindowsFormsApplication4.DataStore
      if (d.Database.CreateIfNotExists())
      {
        // for test
        for (int i = 0; i < 10; i++)
          d.Dates.Add(new Date { Receive = DateTime.Now.AddYears(i) });
        d.SaveChanges();
      }

      var min = new TextBox { Parent = this, Dock = DockStyle.Top };
      var max = new TextBox { Parent = this, Dock = DockStyle.Top };

      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("Load", delegate
      {
        min.Text = d.Dates.Min(r => r.Receive).ToString();
        max.Text = d.Dates.Max(r => r.Receive).ToString();
      });
  	}
	}

	public class Date
	{
	  public int Id { get; set; }
	  public DateTime Receive { get; set; }
	}

	public class DataStore : DbContext
	{
    public DbSet<Date> Dates { get; set; }
    protected override void OnModelCreating(DbModelBuilder mb)
    {
      base.OnModelCreating(mb);
      mb.Entity<Date>().HasKey(t => t.Id).ToTable("Dates");
    }
  }
}
```
instead of DbContext you can use DataContext. an example is [here](http://social.msdn.microsoft.com/Forums/en-US/linqtosql/thread/46967632-c9cc-4289-9743-3d844d26337b/#fd25fe6b-21e4-4e37-be34-5504c39a8f38).