---
title: Storing data locally?
tags: [c#, winforms, ssce, sql server]
src: https://social.msdn.microsoft.com/Forums/ru-RU/d78609b8-94df-4988-88ba-08199087a746/storing-data-locally?forum=winformsdatacontrols
---
# Storing data locally?
*> Now, I'm looking to develop a desktop application , one that will require data to be entered, retrieved, and manipulated. I want this data to be stored on the local machine where the app is running. How can I achieve this? When I build the project, is there some way to include a SQL db into the files that I would give to the end user to run the app on their machine?*

download [SQL Server Compact](http://www.microsoft.com/download/en/details.aspx?id=17876). and take a look at the EF [Code-First Development](http://weblogs.asp.net/scottgu/archive/2010/07/16/code-first-development-with-entity-framework-4.aspx).

*>  I'm looking to develop a desktop application , one that will require data to be entered, retrieved, and manipulated. I want this data to be stored on the local machine where the app is running.*

try using the following code.
it automatically creates items.sdf (SQL Server Compact database) if it does not exists.
and binds data to the controls.
 
create Windows Forms Application project and Add References:
C:\Program Files\Microsoft ADO.NET Entity Framework 4.1\Binaries\EntityFramework.dll
C:\Program Files\Microsoft SQL Server Compact Edition\v4.0\Desktop\System.Data.SqlServerCe.dll
...
```c#
using System.Data.Common;
using System.Data.Entity;
using System.Data.SqlServerCe;
using System.Windows.Forms;

namespace WindowsFormsApplication5
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var ds = new DataStore(new SqlCeConnection(@"Data Source=d:\temp\items.sdf"));
      if (ds.Database.CreateIfNotExists())    // init 
      {
        for (int i = 0; i < 5; i++)
          ds.Items.Add(new Item { Name = "n" + i, Description = "d" + i });
        ds.SaveChanges();
      }

      ds.Items.Load();

      var res = ds.Items.Local.ToBindingList(); 
      var bs = new BindingSource(res, null);

      new RichTextBox { Parent = this, Dock = DockStyle.Fill }
          .DataBindings.Add("Text", bs, "Description");
      new BindingNavigator(bs) { Parent = this, Dock = DockStyle.Top };

      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("Save", (s, e) =>
        {
          foreach (Control c in this.Controls)
        	  foreach (Binding b in c.DataBindings)
              b.BindingManagerBase.EndCurrentEdit();
          ds.SaveChanges();
        });
    }
  }

  public class Item
  {
    public int Id { get; set; }
    public string Name { get; set; }
    public string Description { get; set; }
  }

  public class DataStore : DbContext
  {
    public DbSet<Item> Items { get; set; }
    public DataStore(DbConnection connection) : base(connection, true) { }
    protected override void OnModelCreating(DbModelBuilder m)
    {
      base.OnModelCreating(m);
      m.Entity<Item>().HasKey(e => e.Id).ToTable("Items");
    }
  }
}
```