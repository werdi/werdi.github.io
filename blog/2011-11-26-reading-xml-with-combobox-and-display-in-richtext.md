---
title: C# reading XML with combobox and display in Richtext
tags: [c#, winforms, xml]
src: https://social.msdn.microsoft.com/Forums/ru-RU/0efdaa9c-1a44-46ce-8f50-b00010c5d9b3/c-reading-xml-with-combobox-and-display-in-richtext?forum=winforms
---
# C# reading XML with combobox and display in Richtext
*> i am looking to create an application which has many recipes from "Chinese", "Mexican" to "European" cuisine using xml files. [...] So what i want to do is to click on the Mexican tab and select from my combobox and then choose a recipe; the recipe is displayed on the MexicanRichText1.*
```c#
using System.Data;
using System.Drawing;
using System.IO;
using System.Windows.Forms;

namespace WindowsFormsApplication4
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var xml = @"
      <Cookbook>
        <Cuisine type='Chinese'>
          <Recipe name='Chowmein' calories='450'>Noodles, Eggs, Veggies</Recipe>
          <Recipe name='Friedrice' calories='560'>Rice, celeries, corn</Recipe>
          <Recipe name='Noodlessoup' calories='622'>Noodles, Chicken broth, 2 spoons of salt, 5 minutes</Recipe>
        </Cuisine>
        <Cuisine type='Mexican'>
          <Recipe name='Burito' calories='450'>Oven, eggs, bacon, beans</Recipe>
          <Recipe name='Sangria' calories='135'>Cranberries, Pineapple, juice</Recipe>
          <Recipe name='Nacho' calories='677'>Chips, mozzarella, cheddar, frieds, bacon</Recipe>
          <Recipe name='Chicken'>5hrs, skin, bbq</Recipe>
        </Cuisine>
      </Cookbook>";

      var ds = new DataSet();
      ds.ReadXml(new StringReader(xml));

      this.Font = new System.Drawing.Font("verdana", 12);
      this.Size = new System.Drawing.Size(700, 500);

      var tc = new TabControl() { Parent = this, Dock = DockStyle.Fill };
      // bind SelectedIndex to the selected Cuisine;
      var cm = this.BindingContext[ds, "Cuisine"] as CurrencyManager;
      tc.DataBindings.Add("SelectedIndex", cm, "Position", false, DataSourceUpdateMode.OnPropertyChanged);
      foreach (DataRowView drv in cm.List)
      {
        var type = (string)drv["type"];   //= ds.Tables["Cuisine"].Rows[*]["type"]
        var tp = new TabPage(type);
        tc.TabPages.Add(tp);
        var dv = new DataView(ds.Tables["Recipe"]);
        dv.RowFilter = "Parent.type = '" + type + "'";
        new Page(dv) { Parent = tp, Dock = DockStyle.Fill };
      }

      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("save", (s, e) => 
        {
          ds.AcceptChanges();
          ds.WriteXml("Cookbook.xml");
        });
    }
  }

  class Page : UserControl
  {
    public Page(DataView dv)
    {
      new RichTextBox { Parent = this, Dock = DockStyle.Fill }.DataBindings.Add("Text", dv, "Recipe_text");
      new TextBox { Parent = this, Dock = DockStyle.Top }.DataBindings.Add("Text", dv, "calories");
      new ComboBox
      {
        Parent = this,
        Dock = DockStyle.Top,
        DataSource = dv,
        DisplayMember = "name",
        FlatStyle = FlatStyle.Flat,
        DropDownStyle = ComboBoxStyle.DropDownList,
        BackColor = Color.FromKnownColor(KnownColor.Info)
      };
    }
  }
}
```
*> My XML is updated with new recipe every day / week. How can we integrate it in this and get the updates from the XML?*

if your xml format is like in first post, then you need to transform format. 
it's possible with [XslCompiledTransform](http://msdn.microsoft.com/en-us/library/system.xml.xsl.xslcompiledtransform.aspx)
 
if you need to load xml from file, then use ds.ReadXml(yourxmlfilepath);
 instead of ds.ReadXml(new StringReader(xml)); 

*> when i click on save, the xml is not saved nor updated*

take a look at your project folder and then /bin/debug/cookbook.xml