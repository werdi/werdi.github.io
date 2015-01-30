---
title: How to make a ListBox that changes values of another ListBox by selection?
tags: [c#, listbox, winforms, xml, data binding]
src: https://social.msdn.microsoft.com/Forums/ru-RU/eafd678a-5e2b-4f25-aece-e82089102990/newbiwhow-to-make-a-listbox-that-changes-values-of-another-listbox-by-selection?forum=winforms
---
# How to make a ListBox that changes values of another ListBox by selection?
*> a ListBox that changes values of another ListBox by selection [...] without using any External DataBase?*
you can use DataSet and databindings.
below is a complete example.
```c#

using System.Data;
using System.IO;
using System.Windows.Forms;

namespace WindowsFormsApplication0
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var ds = new DataSet();
      ds.ReadXml(new StringReader(@"<data>
          <c name='Fruit'><i>Apple</i><i>Orange</i><i>Bannana</i></c>
          <c name='Meat'><i>Ham</i><i>Beaf</i><i>Steak</i></c>
          <c name='Vegetable'><i>Potato</i><i>Tomato</i><i>Onion</i></c>
        </data>"));
      new ListBox { Parent = this, Dock = DockStyle.Top, 
        DataSource = ds, DisplayMember = "c.c_i.i_text" };
      new ListBox { Parent = this, Dock = DockStyle.Top, 
        DataSource = ds, DisplayMember = "c.name" };
    }
  }
}
```
