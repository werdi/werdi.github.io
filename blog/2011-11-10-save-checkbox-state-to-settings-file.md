---
title: Save checkbox state to settings file
tags: [c#, winforms, linq, xml, data binding]
src: https://social.msdn.microsoft.com/Forums/ru-RU/bde3e597-0fd2-44a9-954d-0b8efeecee40/save-checkbox-state-to-settings-file?forum=winforms
---
# Save checkbox state to settings file
*> i need to save the 3 checkbox states to the settings file*

you can use databinding. 
below is an example with two approaches: v1 and v2.
```c#
using System.Data;
using System.IO;
using System.Windows.Forms;
using System.Xml.Linq;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var xml = @"<root>
        <data checked='true' />
      </root>";

      // v1: load xml and binding to xelement
      var xe = XElement.Parse(xml);
      var cb1 = new CheckBox { Parent = this, Dock = DockStyle.Top, Text = "XElement" };
      cb1.DataBindings.Add("Checked", xe.Element("data").Attribute("checked"), "Value");

      // v2: load xml and binding to dataset
      var ds = new DataSet();
      ds.ReadXml(new StringReader(xml));
      var cb2 = new CheckBox { Parent = this, Dock = DockStyle.Top, Text = "DataSet" };
      cb2.DataBindings.Add("Checked", ds, "data.checked");

      // test
      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("trace", (s, e) => 
      {
        foreach(Control c in this.Controls)
          foreach(Binding b in c.DataBindings)
            b.WriteValue();

        // todo: save to file
        MessageBox.Show("DataSet: " + ds.GetXml() + "\n\n XElement: " + xe);
      });
    }
  }
}
```
*> i want to save xml results in the debug/bin folder and this example where is saving my 3 checkboxes wich is best to use v1 or v2?*

v1 is best way if you need to manage settings for users and maybe store a versions of settings and another information.
below is an example, which loads information from file and saves file when form closing
```c#
using System.Data;
using System.IO;
using System.Windows.Forms;
using System.Xml.Linq;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var ds = new DataSet();
      var file = "settings.xml";
      if(File.Exists(file))
        ds.ReadXml(file);
      else
      {
        var dt = new DataTable("data");
        dt.Columns.Add("MinTarefas", typeof(bool)).DefaultValue = true;
        dt.Columns.Add("BaixarVolSkype", typeof(bool)).DefaultValue = true;
        dt.Columns.Add("SkypeMute", typeof(bool)).DefaultValue = true;
        ds.Tables.Add(dt);
        dt.Rows.Add(true, true, true);
      }
      
      new CheckBox { Parent = this, Dock = DockStyle.Top, Text = "MinTarefas" }
        .DataBindings.Add("Checked", ds, "data.MinTarefas");
      new CheckBox { Parent = this, Dock = DockStyle.Top, Text = "BaixarVolSkype" }
        .DataBindings.Add("Checked", ds, "data.BaixarVolSkype");
      new CheckBox { Parent = this, Dock = DockStyle.Top, Text = "SkypeMute" }
        .DataBindings.Add("Checked", ds, "data.SkypeMute");

      this.FormClosing += (s, e) => 
      {
        foreach(Control c in this.Controls)
          foreach(Binding b in c.DataBindings)
            b.WriteValue();

        ds.AcceptChanges();
        ds.WriteXml(file);
      };
    }
  }
}
```
*> please adapt you code to correspond with my similar code is frist post [...] 'i have a lkistcheckbox with 3 checkboxes and the frist is independent and the second and 3rd is if 2 is maked 3rd is unmarked and vice versa'*

if i understood correctly then try following
```c#
using System.Data;
using System.IO;
using System.Windows.Forms;
using System.Xml.Linq;
using System;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var ds = new DataSet();
      var file = "settings.xml";
      if(File.Exists(file))
        ds.ReadXml(file);
      else
      {
        var dt = new DataTable("data");
        dt.Columns.Add("MinTarefas", typeof(bool)).DefaultValue = true;
        dt.Columns.Add("BaixarVolSkype", typeof(bool)).DefaultValue = true;
        dt.Columns.Add("SkypeMute", typeof(bool)).DefaultValue = true;
        ds.Tables.Add(dt);
        dt.Rows.Add(true, true, true);
      }
      
      new CheckBox { Parent = this, Dock = DockStyle.Top, Text = "MinTarefas" }
        .DataBindings.Add("Checked", ds, "data.MinTarefas");

      var cb2 = new CheckBox { Parent = this, Dock = DockStyle.Top, Text = "BaixarVolSkype" };
      cb2.DataBindings.Add("Checked", ds, "data.BaixarVolSkype");
      
      var cb3 = new CheckBox { Parent = this, Dock = DockStyle.Top, Text = "SkypeMute" };
      cb3.DataBindings.Add("Checked", ds, "data.SkypeMute");

      cb2.CheckedChanged += (s, e) => 
      {
        if(cb2.CheckState == CheckState.Checked)
          cb3.CheckState = CheckState.Unchecked;
      };
      
      cb3.CheckedChanged += (s, e) => 
      {
        if(cb3.CheckState == CheckState.Checked)
          cb2.CheckState = CheckState.Unchecked;
      };

      this.FormClosing += (s, e) => 
      {
        foreach(Control c in this.Controls)
          foreach(Binding b in c.DataBindings)
            b.WriteValue();

        ds.AcceptChanges();
        ds.WriteXml(file);
      };
    }
  }
}
```
*> 3- need to save the checkboxes state in a xml file in the project folder*

if you run my code above, then you'll see settings.xml in folder \bin\Debug.
you can change the settings path as you want. something like this:
```
var file = @"..\..\settings.xml";
```
- or -
```
var file = @"c:\temp\settings.xml";
```
*> so withyour code i cant distinct if your are talking if its form3 or form 1 because i just see form1
form 3 is the preferences formform1 should read checkbox state from the xml file!*

do you need to have CheckBoxes in subform?
am i understand you properly: form1 loads xml, and after some user action opens form3 with checkboxes?

*> ... a preferences/options menu wich is called form3, and the form1 wich is the main form. [...]
the checkbox states takes effect on form1 [...] form3 have a button , a single button, named save and exit*

if i understood correctly, you have to get the second form (some kind of settings editor with checkbox's).
second form contains save'n'exit button. after button click the checkboxes states should be reflected on the main form.
so below is an example. hope that is what you wanted.
```c#
using System;
using System.Data;
using System.IO;
using System.Windows.Forms;

namespace WindowsFormsApplication0
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var file = "settings.xml";
      var ds = LoadData(file);

      var lbl = new Label
      {
        Parent = this,
        Dock = DockStyle.Top,
        Height = 100,
        Text = GetText(ds)
      };

      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("Settings", (s, e) =>
      {
        ds.AcceptChanges();
        var frm = new Form2(this.BindingContext, ds);
        frm.Owner = this;
        var res = frm.ShowDialog();
        if(res != System.Windows.Forms.DialogResult.OK)
          ds.RejectChanges();

        lbl.Text = GetText(ds);
      });

      this.Closing += (s, e) =>
      {
        ds.AcceptChanges();
        ds.WriteXml(file, XmlWriteMode.WriteSchema);
      };
    }

    string GetText(object ds)
    {
      return String.Concat(
        "SkypeMute=", GetState(ds, "data.SkypeMute"),
        "\nBaixarVolSkype=", GetState(ds, "data.BaixarVolSkype"),
        "\nMinTarefas=", GetState(ds, "data.MinTarefas")
        );
    }

    bool GetState(object ds, string dm)
    {
      return Convert.ToBoolean(this.BindingContext[ds, dm].Current);
    }

    DataSet LoadData(string file)
    {
      var ds = new DataSet();
      if (File.Exists(file))
        ds.ReadXml(file);
      else
      {
        var dt = new DataTable("data");
        dt.Columns.Add("MinTarefas", typeof(bool)).DefaultValue = true;
        dt.Columns.Add("BaixarVolSkype", typeof(bool)).DefaultValue = true;
        dt.Columns.Add("SkypeMute", typeof(bool)).DefaultValue = true;
        ds.Tables.Add(dt);
        dt.Rows.Add(true, true, true);
      }
      ds.AcceptChanges();
      return ds;
    }
  }

  class Form2 : Form
  {
    public Form2(BindingContext bc, object ds)
    {
      this.BindingContext = bc;

      new CheckBox { Parent = this, Dock = DockStyle.Top, Text = "MinTarefas" }
        .DataBindings.Add("Checked", ds, "data.MinTarefas");

      var cb2 = new CheckBox { Parent = this, Dock = DockStyle.Top, Text = "BaixarVolSkype" };
      cb2.DataBindings.Add("Checked", ds, "data.BaixarVolSkype");

      var cb3 = new CheckBox { Parent = this, Dock = DockStyle.Top, Text = "SkypeMute" };
      cb3.DataBindings.Add("Checked", ds, "data.SkypeMute");

      cb2.CheckedChanged += (s, e) =>
      {
        if (cb2.CheckState == CheckState.Checked)
          cb3.CheckState = CheckState.Unchecked;
      };

      cb3.CheckedChanged += (s, e) =>
      {
        if (cb3.CheckState == CheckState.Checked)
          cb2.CheckState = CheckState.Unchecked;
      };

      this.Closing += (s, e) =>
      {
        foreach (Control c in this.Controls)
          foreach (Binding b in c.DataBindings)
            b.WriteValue();
      };

      new Button 
      {
        Text = "Save'n'Exit",
        Parent = this,
        Location = new System.Drawing.Point(5, 100),
        DialogResult = System.Windows.Forms.DialogResult.OK
      };
    }
  }
}
```