---
title: Can anyone give me some advice on a possible way to make these if statements easier?
tags: [c#, winforms, xml, data]
src: https://social.msdn.microsoft.com/Forums/ru-RU/caa905c1-17ef-4a2d-b0fc-6f391aff023d/can-anyone-give-me-some-advice-on-a-possible-way-to-make-these-if-statements-easier?forum=csharplanguage
---
# Can anyone give me some advice on a possible way to make these if statements easier?
*> Can anyone give me some advice on a possible way to make these if statements easier?*

instead of your code use the following:
```c#
listBox1.Items.Clear();
for (int j=0; j < 2; j++)
  listBox1.Items.Add(recipe.fillBox(comboBox1.SelectedIndex, j));
```
*> Basically. The program will allow her to pick a category from a combobox. Based on the category selected it will fill a listbox below with names of different recipes*
  
if i understood correctly, you want to create a cookbook editor.
below is an example.
```c#
using System.Data;
using System.IO;
using System.Windows.Forms;
using System.Drawing;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.Size = new System.Drawing.Size(600, 500);
      var ds = new DataSet();
      if (System.IO.File.Exists("cookbook.xml"))
        ds.ReadXml("cookbook.xml");
      else
        ds.ReadXml(new StringReader(@"
        <cookbook>  <!-- test data -->
          <category name='category 1'>
            <recipe title='recipe 1'>text 1</recipe>
            <recipe title='recipe 2'>text 2</recipe> 
          </category>
          <category name='category 2'>
            <recipe title='recipe 3'>text 3</recipe> 
          </category>
        </cookbook>"));

      var sc1 = new SplitContainer { 
                    Parent = this, Dock = DockStyle.Fill, Orientation = Orientation.Horizontal };
      var sc2 = new SplitContainer { Parent = sc1.Panel1, Dock = DockStyle.Fill };
      
      var c1 = CreateGrid(ds, "category");
      c1.Parent = sc2.Panel1;
      c1.Columns.Add(new DataGridViewTextBoxColumn { 
        HeaderText = "Category", DataPropertyName = "name" });

      var c2 = CreateGrid(ds, "category.category_recipe");
      c2.Parent = sc2.Panel2;
      c2.Columns.Add(new DataGridViewTextBoxColumn { 
        HeaderText = "Recipe", DataPropertyName = "title" });

      var c3 = new RichTextBox { Dock = DockStyle.Fill };
      c3.Parent = sc1.Panel2;
      c3.DataBindings.Add("Text", ds, "category.category_recipe.recipe_text");

      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("save", delegate
      {
        ds.WriteXml("cookbook.xml");
        MessageBox.Show("saved");
      });
      this.Menu.MenuItems.Add("new category", delegate
      {
        var dv = AddNew(ds, "category");
        dv["name"] = "new category";
      });
      this.Menu.MenuItems.Add("new recipe", delegate
      {
        var dv = AddNew(ds, "category.category_recipe");
        dv["title"] = "new recipe";
      });
    }
    DataRowView AddNew(object ds, string dm)
    {
      var cm = this.BindingContext[ds, dm] as CurrencyManager;
      cm.AddNew();
      cm.EndCurrentEdit();
      return cm.Current as DataRowView;
    }
    DataGridView CreateGrid(object ds, string dm)
    {
      return new DataGridView 
      {
        DataSource = ds,
        Dock = DockStyle.Fill,
        RowHeadersVisible = false,
        AllowUserToAddRows = false,
        DataMember = dm,
        AllowUserToResizeRows = false,
        BackgroundColor = Color.White,
        AutoSizeColumnsMode = DataGridViewAutoSizeColumnsMode.Fill,
        AutoGenerateColumns = false
      };
    }
  }
}
```
