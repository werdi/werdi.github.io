---
title: Как связать две таблицы datatable которые хранятся в xml файлах
tags: [c#, winforms, xml]
src: https://social.msdn.microsoft.com/Forums/ru-RU/2bdba906-a313-4556-aede-e0c474321f83/-datatable-xml-?forum=fordesktopru
---
# Как связать две таблицы datatable которые хранятся в xml файлах
*> У меня есть два файла 1.xml и 2.xml (т.е. две таблици datatable), мне необходимо их связать по ключевому полю. как это можно реализовать.*
```c#
using System.Data;
using System.IO;
using System.Windows.Forms;

namespace WindowsFormsApplication5
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var ds1 = new DataSet();
      ds1.ReadXml(new StringReader(@"
        <root>
          <i2 id='0' p='0' />
          <i2 id='1' p='0' />
          <i2 id='2' p='1' />
        </root>"));
      
      var ds2 = new DataSet();
      ds2.ReadXml(new StringReader(@"
        <root>
          <i1 id='0' />
          <i1 id='1' />
        </root>"));

      ds1.Merge(ds2);
      ds1.Relations.Add("i1_i2", ds1.Tables["i1"].Columns["id"], ds1.Tables["i2"].Columns["p"]);

      this.Width = 600;
      var sc = new SplitContainer { Dock = DockStyle.Fill, Parent = this, SplitterDistance = 250 };
      new DataGridView { Dock = DockStyle.Fill, Parent = sc.Panel1, AllowUserToAddRows = false,
        DataSource = ds1, DataMember = "i1" };
      new DataGridView { Dock = DockStyle.Fill, Parent = sc.Panel2, AllowUserToAddRows = false,
        DataSource = ds1, DataMember = "i1.i1_i2" };
    }
  }
}
```
