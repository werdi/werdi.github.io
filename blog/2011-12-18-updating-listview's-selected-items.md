---
title: Updating ListView in SelectedItems only (Windows Form)
tags: [c#, winforms, linq, data]
src: https://social.msdn.microsoft.com/Forums/ru-RU/7c06c9d1-389b-4ba9-9687-a6b07a48762d/help-updating-listview-in-selecteditems-only-windows-form-?forum=winformsdatacontrols
---
# Updating ListView in SelectedItems only (Windows Form)
*> any suggestion on how to update just a single selected data on ListView?*

you should update ListViewItem.Text or ListViewSubItem.Text,
but a more convenient way is to use DataGridView.
below is example for ListView and DataGridView.
```c#
using System.Data.Entity;
using System.Drawing;
using System.Linq;
using System.Windows.Forms;

namespace WindowsFormsApplication6
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.Size = new Size(500, 450);

      var ds = new DataStore();
      if (ds.Database.CreateIfNotExists())
      {
        for (int i = 0; i < 20; i++) ds.Items.Add(new Item { Name = "item" + i });
        ds.SaveChanges();
      }

      var items = ds.Items.Where(i => i.Name.Contains("3")).ToList();
      
      // example 1
      var dg = new DataGridView
      {
        GridColor = Color.WhiteSmoke,
        BackgroundColor = Color.White,
        Parent = this,
        RowHeadersVisible = false,
        Dock = DockStyle.Fill,
        DataSource = items
      };
      dg.Columns["Id"].ReadOnly = true;
      dg.Columns["Name"].AutoSizeMode = DataGridViewAutoSizeColumnMode.Fill;

      new Label 
        { Parent = this, Dock = DockStyle.Bottom, Text = "ListView", 
          TextAlign = ContentAlignment.MiddleLeft };

      // example 2
      var lv = new ListView 
          { Parent = this, Dock = DockStyle.Bottom, Height = 200, View = View.Details };
      lv.Columns.Add("Id");
      lv.Columns.Add("Name");
      foreach(var item in items)
      {
        var lvi = new ListViewItem(item.Id.ToString());
        lvi.SubItems.Add(item.Name);
        lvi.Tag = item;
        lv.Items.Add(lvi);
      }
      lv.Columns[0].Width = 100;
      lv.Columns[1].Width = 200;

      this.FormClosing += (s, e) => ds.SaveChanges();

      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("Test (DataGridView)", (s, e) =>
      {
        var rowIndex = 1;
        var itm = dg.Rows[rowIndex].DataBoundItem as Item;
        itm.Name += "3";
        ds.SaveChanges(); 
        dg.UpdateCellValue(dg.Columns["Name"].Index, rowIndex);   // or: dg.Refresh();
      });
      this.Menu.MenuItems.Add("Test (ListView)", (s, e) =>
      {
        var lvi = lv.Items[1];
        var itm = lvi.Tag as Item;
        itm.Name += "3";
        lvi.SubItems[1].Text = itm.Name;
        ds.SaveChanges(); 
      });
	  }
	}

	public class Item
	{
	  public int Id { get; set; }
	  public string Name { get; set; }
	}

	public class DataStore : DbContext
	{
    public DbSet<Item> Items { get; set; }
    protected override void OnModelCreating(DbModelBuilder m)
    {
      base.OnModelCreating(m);
      m.Entity<Item>().HasKey(i => i.Id).ToTable("Items");
    }
  }
}
```