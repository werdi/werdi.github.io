---
title: Display data from a database using a data grid
tags: [c#, winforms, binding]
src: https://social.msdn.microsoft.com/Forums/ru-RU/578867d2-db87-4369-aba6-ba360b0c5743/display-data-from-a-database-using-a-data-grid?forum=csharpgeneral
---
# Display data from a database using a data grid
*> windows application development environment [...] grid-view [...] inserting, updating and displaying data from in-built SQL Server*

the following example uses SQL Server, EF Code First, and 
data binding for display, insert, update data.
```c#
using System.Windows.Forms;
using System.Data.Entity;

namespace WindowsFormsApplication9
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var ds = new DataStore();
      if (ds.Database.CreateIfNotExists())
      {
        for (int i = 0; i < 10; i++)
          ds.Items.Add(new Item { Text = "text " + i });
        ds.SaveChanges();
      }

      var bs = new BindingSource();
      ds.Items.Load();
      bs.DataSource = ds.Items.Local.ToBindingList();

      this.Size = new System.Drawing.Size(500, 400);
      new DataGridView
      {
        Parent = this,
        Dock = DockStyle.Fill,
        AutoGenerateColumns = false,
        DataSource = bs,
        SelectionMode = DataGridViewSelectionMode.FullRowSelect,
         AutoSizeColumnsMode = DataGridViewAutoSizeColumnsMode.Fill
      }
      .Columns.Add( new DataGridViewTextBoxColumn { DataPropertyName = "Text" } );

      new RichTextBox { Dock = DockStyle.Bottom, Parent = this, Height = 100 }
        .DataBindings.Add("Text", bs, "Text");
      new BindingNavigator(bs) { Parent = this, Dock = DockStyle.Bottom, };

      this.FormClosing += (s, e) =>
      {
        foreach (Control c in this.Controls)
          foreach (Binding b in c.DataBindings)
            b.BindingManagerBase.EndCurrentEdit();

        ds.SaveChanges();
      };
    }
  }

  public class Item
  {
    public int Id { get; set; }
    public string Text { get; set; }
  }
  public class DataStore : DbContext
  {
    public DbSet<Item> Items { get; set; }
    protected override void OnModelCreating(DbModelBuilder mb)
    {
      base.OnModelCreating(mb);
      mb.Entity<Item>().HasKey(e => e.Id).ToTable("Items");
    }
  }
}
```