---
title: Create, Modify and Read XML Files in C#
tags: [c#, winforms, xml, interop]
src: https://social.msdn.microsoft.com/Forums/ru-RU/2aa61d5c-6804-46e3-b186-7515a2dfe7e4/create-modify-and-read-xml-files-in-c?forum=xmlandnetfx
---
# Create, Modify and Read XML Files in C#
*> It's for a WinForms application. What I need to do is create the XML file [...] If, by any chance, the user cancel and continues at a later stage, I need to provide them with the values they have provided earlier and be able to modify those values.*

take a look at my examples here.
 below is another one with different type of bindings.
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
      this.Size = new System.Drawing.Size(600, 600);
      var ds = new DataSet();
      ds.ReadXml(new StringReader(@"
        <wizard>
          <pages current-page='1'>
            <page id='1'>
              <address id='1'>address 1</address>
              <type text='type 1'>true</type>
              <type text='type 2'>true</type>
              <type text='type 3'>true</type>
              <match text='m 1' value='true' />
              <match text='m 2' />
              <match text='m 3' />
            </page>
            <page id='2'>
              <address id='2'>address 2</address>
              <item value='v1'>txt</item>
              <item value='v2' />
              <item value='v3' />
            </page>
          </pages>
        </wizard>"));
      // pages
      var tc = new TabControl { Parent = this, Dock = DockStyle.Fill };
      CreatePage1(ds, new TabPage { Parent = tc, Padding = new Padding(5) });
      CreatePage2(ds, new TabPage { Parent = tc, Padding = new Padding(5) });
      // navigator
      var p = new Panel { Parent = this, Dock = DockStyle.Bottom, Height = 30, Padding = new Padding(0, 0, 100, 5) };
      new Button { Parent = p, Dock = DockStyle.Right, Text = "Prev", Width = 100 }
          .Click += (s, e) => tc.SelectTab(0);
      new Button { Parent = p, Dock = DockStyle.Right, Text = "Next", Width = 100 }
        .Click += (s, e) => tc.SelectTab(1);
      // xml view
      var trace = new RichTextBox { Parent = this, Dock = DockStyle.Bottom, Height = 250, ReadOnly = true };
      foreach (DataTable dt in ds.Tables)
        dt.ColumnChanged += (s, e) => trace.Text = ds.GetXml();

      tc.DataBindings.Add("SelectedIndex", ds, "pages.current-page");
  }
  void BindTextBox(object ds, string dm, TextBoxBase tb)
  {
      var b = tb.DataBindings.Add("Text", ds, dm);
      tb.TextChanged += (s, e) =>
      {
        var ss = tb.SelectionStart;
        b.WriteValue();
        tb.SelectionStart = ss;
      };
    }
    private void CreatePage1(DataSet ds, Control parent)
    {
      var dv = new DataView { Table = ds.Tables["page"], RowFilter = "id=1" };
      BindTextBox(dv, "page_address.address_text",
        new RichTextBox { Parent = parent, Dock = DockStyle.Bottom, Height = 150 });
      var types = dv[0].Row.GetChildRows("page_type");
      var p1 = new Panel {Dock = DockStyle.Left, Width = 100 };
      for (var i=0; i < types.Length; i++)
      {
        var row = types[i];
        var cb = new CheckBox { Parent = p1, Dock = DockStyle.Top, Text = row["text"].ToString() };
        cb.CheckedChanged += (s, e) =>
          row["type_text"] = (s as CheckBox).Checked;
      }
      var matches = dv[0].Row.GetChildRows("page_match");
      var p2 = new Panel {Dock = DockStyle.Left, Width = 100 };
      for (var i=0; i < matches.Length; i++)
      {
        var row = matches[i];
        var rb = new RadioButton { Parent = p2, Dock = DockStyle.Top, Text = row["text"].ToString(), 
          Checked = (row["value"] is DBNull) ? false : bool.Parse(row["value"] as string) };
        rb.CheckedChanged += (s, e) =>
          row["value"] = (s as RadioButton).Checked;
      }
      new Panel { Dock = DockStyle.Fill, Parent = parent }.Controls.AddRange(new [] { p1, p2 });
    }
    private void CreatePage2(DataSet ds, Control parent)
    {
      var dv = new DataView { Table = ds.Tables["page"], RowFilter = "id=2" };
      BindTextBox(dv, "page_address.address_text",
        new RichTextBox { Parent = parent, Dock = DockStyle.Fill });
      var dv2 = new DataView { Table = ds.Tables["item"], RowFilter = "parent(page_item).id=2" };
      new DataGridView
      {
        Parent = parent,
        DataSource = dv2,
        Dock = DockStyle.Right,
        AllowUserToAddRows = false,
        AllowUserToDeleteRows = false,
        Width = 400
      };
    }
    class TabControl : System.Windows.Forms.TabControl
    {
      protected override void WndProc(ref Message m)
      {
        // hide tabs
        if (m.Msg == 0x1328)     // TCM_ADJUSTRECT
          m.Result = (IntPtr)1;
        else
          base.WndProc(ref m);
      }
    }
  }
}
```