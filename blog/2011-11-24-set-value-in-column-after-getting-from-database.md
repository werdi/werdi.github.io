---
title: Set Value in datagridviewcomboboxcolumn after getting from database
tags: [c#, winforms, data, ef]
src: https://social.msdn.microsoft.com/Forums/ru-RU/be38218d-1acb-4cb8-b595-1c7460a61322/set-value-in-datagridviewcomboboxcolumn-after-getting-from-database?forum=winformsdatacontrols
---
# Set Value in datagridviewcomboboxcolumn after getting from database
*> I am using DataGridViewComboBoxColumn and column datasource is bound with a datatable and on loading data from Database column value will be show as empty... It doesn't work....*

there is no need to use a DataTable, if you want to populate combobox with data from the database.
 try the following example.
 before launch, ensure SQL Server (SQLEXPRESS) service is working.
```c# 
using System.Data.Entity;
using System.Windows.Forms;

namespace WindowsFormsApplication4
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var ds = LoadData();
      var dg = new DataGridView
      {
        Parent = this,
        Dock = DockStyle.Fill,
        AutoGenerateColumns = false,
      }
      .Columns.Add(new DataGridViewComboBoxColumn
      {
        DataSource = ds,
        FlatStyle = System.Windows.Forms.FlatStyle.Flat,
        DisplayMember = "id",
      });
    }

    private object LoadData()
    {
      var d = CreateData();
      d.Tags.Load();
      return d.Tags.Local.ToBindingList();
    }

    private Data CreateData()
    {
      var d = new Data();
      if (d.Database.Exists() == false)
      {
        // .\SQLEXPRESS - Databases - WindowsFormsApplication4.Data
        d.Database.Create(); 
        for (int i = 0; i < 5; i++) d.Tags.Add(new Tag { Name = "tag" + i });
        d.SaveChanges();
      }
      return d;
    }
  }

  public class Tag
  {
    public int Id { get; set; }
    public string Name { get; set; }
  }

  public class Data : DbContext
  {
    public DbSet<Tag> Tags { get; set; }
    protected override void OnModelCreating(DbModelBuilder mb)
    {
      base.OnModelCreating(mb);
      mb.Entity<Tag>().HasKey(t => t.Id).ToTable("Tags");
    }
  }
}
```