---
title: VS2010 Изменить форму
tags: [c#, winforms]
src: https://social.msdn.microsoft.com/Forums/ru-RU/ecb63bdf-812c-48a5-83c3-8ed6457ffc3d/vs2010-?forum=fordesktopru
---
# VS2010 Изменить форму
*> В форме много контролов (текстбоксы и комбобоксы). Есть ли какое-нибудь свойство (или событие) у формы, кот. показывает, был ли изменен какой-либо из этих контролов?*

нет. надо подписываться на события в контролах. например, на TextBox.TextChanged, ComboBox.SelectedVaueChanged и т.д.
 но если контролы привязаны к DataSet, то см. метод DataSet.GetChanges

*> заинтересовал метод DataSet.GetChanges [...]. Но  как им воспользоваться?*

пример формы с контролами, привязанными к DataSet;
```c#
using System.Data;
using System.Drawing;
using System.IO;
using System.Windows.Forms;
using System;

namespace WindowsFormsApplication4
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var ds = LoadData();
      this.Size = new Size(500, 500);
      this.Padding = new Padding(10);
      this.FormBorderStyle = FormBorderStyle.FixedToolWindow;
      this.Font = new Font("verdana", 12);
      var bs = new BindingSource(ds, "order");
      new ListBox
      {
        Parent = this,
        Dock = DockStyle.Fill,
        IntegralHeight = false,
        DataSource = bs,
        DisplayMember = "order_i.id"
      };
      new RichTextBox { Parent = this, Dock = DockStyle.Right, BorderStyle = BorderStyle.FixedSingle, Width = 200 }
        .DataBindings.Add("Text", bs, "order_i.i_text");
      new TextBox { Parent = this, Dock = DockStyle.Top }
        .DataBindings.Add("Text", bs, "id");
      new ComboBox
      {
        Parent = this,
        Dock = DockStyle.Top,
        DropDownStyle = ComboBoxStyle.DropDownList,
        DataSource = new BindingSource(ds, "order-type"),
        DisplayMember = "name",
        ValueMember = "name"
      }
      .DataBindings.Add("SelectedValue", bs, "type");
      new BindingNavigator(bs) { Parent = this, Dock = DockStyle.Top };
      new DataGrid { Parent = this, Dock = DockStyle.Bottom, Height = 200, DataSource = bs };
      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("Save", (s, e) =>
      {
        foreach (Control c in this.Controls)
          foreach (Binding b in c.DataBindings)
            b.BindingManagerBase.EndCurrentEdit();
        var cs = ((DataSet)ds).GetChanges();  // получить изменения
        MessageBox.Show(cs != null ? cs.GetXml() : "none");
        ((DataSet)ds).AcceptChanges();
      });
    }
    object LoadData()
    {
      var ds = new DataSet();
      ds.ReadXml(new StringReader(@"
      <data>
        <order type='ot1' id='o1'><i id='i1'>txt1</i><i id='i2'>txt2</i></order>
        <order type='ot2' id='o2'><i id='i3'>txt3</i><i id='i4'>txt4</i><i id='i5' /></order>
        <order-type name='ot1' />
        <order-type name='ot2' />
        <order-type name='ot3' />
      </data>"));
      ds.AcceptChanges();
      foreach(DataTable dt in ds.Tables)
        dt.RowChanging += (s, e) =>
          System.Diagnostics.Trace.WriteLine(e.Action + " " + String.Join("; ", e.Row.ItemArray));
      return ds;
    }
  }
}
```