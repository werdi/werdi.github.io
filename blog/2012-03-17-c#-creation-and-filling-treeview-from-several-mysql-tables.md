---
title: C#. Создание и заполнение treeview из нескольких таблиц Mysql
tags: [c#, winforms, xml]
src: https://social.msdn.microsoft.com/Forums/ru-RU/3de49489-88e1-4ab4-8b27-effb37076949/c-treeview-mysql?forum=fordataru
---
# C#. Создание и заполнение treeview из нескольких таблиц Mysql
*> Нужно чтобы при запуске программы (VS2010, c#) создавался treeview на основании всех этих таблиц... [...] -department (отдел) - post - staff ...*

примерно так:
```c#
using System.Data;
using System.IO;
using System.Windows.Forms;

namespace WindowsFormsApplication1
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var ds = new DataSet("org");
      ds.ReadXml(new StringReader(@" <!-- test -->
        <departament name='d1'>
          <post name='p1'>
            <staff name='fio1' />
          </post>
          <departament name='d2'>
            <post name='p2'>
              <staff name='fio2' />
              <staff name='fio3' />
            </post>
          </departament>
          <departament name='d3'>
            <post name='p3' />
          </departament>
          <departament name='d4' />
        </departament>"));

      foreach (DataTable dt in ds.Tables)
        foreach (DataColumn dc in dt.Columns)
          if (dc.ColumnMapping == MappingType.Hidden)
            dc.ColumnMapping = MappingType.Element;

      this.Size = new System.Drawing.Size(900, 700);
      var sc = new SplitContainer { Parent = this, Dock = DockStyle.Fill, SplitterDistance = (int)(this.Width * .45) };
      var dg = new DataGrid { DataSource = ds, Dock = DockStyle.Fill, Parent = sc.Panel2, ReadOnly = true };
      new RichTextBox { Parent = sc.Panel2, Dock = DockStyle.Bottom, Height = 350, Text = ds.GetXml() };

      var tv = new TreeView { Dock = DockStyle.Fill, Parent = sc.Panel1 };
      foreach (DataRow dr in ds.Tables["departament"].Rows)
        if (dr.GetParentRow("departament_departament") == null)
          Fill(tv.Nodes, dr);

      tv.ExpandAll();

      this.Shown += (s, e) =>
      {
        dg.Focus();
        SendKeys.SendWait("%({DOWN})");
        tv.Focus();
      };
    }

    private static void Fill(TreeNodeCollection nodes, DataRow dr)
    {
      var dn = nodes.Add((string)dr["name"]);
  
      var ps = dr.GetChildRows("departament_post");
      if (ps != null)
        foreach (DataRow pr in ps)
        {
          var pn = dn.Nodes.Add((string)pr["name"]);
          var ss = pr.GetChildRows("post_staff");
          if (ss != null)
            foreach (DataRow sr in ss)
              pn.Nodes.Add((string)sr["name"]);
        }
  
        var ds = dr.GetChildRows("departament_departament");
        if (ds != null)
          foreach (DataRow cr in ds)
            Fill(dn.Nodes, cr);
      }
    }
  }
```

*> данный пример составляет treview из xml и в таком же виде выводит*

пример выводит данные из DataSet. как они попадают в DataSet не имеет значения. 
данные могут быть загружены из xml (как в примере) или из базы данных.

*> Для построения такой структуры нужно как-то изменить приведённый код ...*

можно ли изменить структуру базы? что возвращает метод this.hrDataSet.GetXml()?

*> А где вы увидели в предыдущем коде метод this.hrDataSet.GetXml()?*

покажите xml, который возвращает метод GetXml. возможно, что проще заполнить TreeView на основе xml, а не на основе DataSet.

*> Вот пример по 1 сотруднику.*

примерно так: 
```c#
using System.Windows.Forms;
using System.Xml.Linq;
using System.Xml.XPath;

namespace WindowsFormsApplication1
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var tv = new TreeView { Parent = this, Dock = DockStyle.Fill };
      var xe = XElement.Load("data.xml");
      foreach (var d in xe.XPathSelectElements("//Department[Parent_Dept_Code='00000000']"))
        Run(d, tv.Nodes);
      tv.ExpandAll();
    }

    void Run(XElement d, TreeNodeCollection nc)
    {
      var dc = d.Element("Dept_Code").Value;
      var nn = nc.Add(d.Element("Full_Name").Value);
      
      var re = d.XPathSelectElement("//Relation[Dept_Code/text()='" + dc + "']");
      var pn = re.Element("Personal_Number");
      foreach(var t in re.Element("Titles_Codes").Elements("Title_Code"))
      {
        foreach(var x in d.XPathSelectElements("//Title[Title_Code='" + t.Value + "']"))
        {
          var s = nn.Nodes.Add(x.Element("Full_Name").Value);
          var p = x.XPathSelectElement("//Staff[Personal_Number='" + pn.Value + "']");
          if(p != null)
            s.Nodes.Add(p.Element("LName").Value);
        }
      }
      foreach (var s in d.XPathSelectElements("//Department[Parent_Dept_Code='" + dc + "']"))
        Run(s, nn.Nodes);
    }
  }
}
```

*> изменились поля в xml. [...] Вот новый xml... Что нужно изменить в вашем коде?*
```c#
using System.Windows.Forms;
using System.Xml.Linq;
using System.Xml.XPath;

namespace WindowsFormsApplication1
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var tv = new TreeView { Parent = this, Dock = DockStyle.Fill };
      var xe = XElement.Load("data.xml");
      foreach (var d in xe.XPathSelectElements("//Department[Parent_Dept_Code='00000000']"))
        Run(d, tv.Nodes);
      tv.ExpandAll();
    }

    void Run(XElement d, TreeNodeCollection nc)
    {
      var dc = d.Element("Dept_Code").Value;
      var nn = nc.Add(d.Element("Full_Name1").Value);
      
      var re = d.XPathSelectElement("//Relation[Dept_Code1/text()='" + dc + "']");
      var pn = re.Element("Personal_Number1");
      foreach(var t in re.Element("Titles_Codes").Elements("Title_Code2"))
      {
        foreach(var x in d.XPathSelectElements("//Title[Title_Code='" + t.Value + "']"))
        {
          var s = nn.Nodes.Add(x.Element("Full_Name").Value);
          var p = x.XPathSelectElement("//Staff[Personal_Number='" + pn.Value + "']");
          if(p != null)
            s.Nodes.Add(p.Element("LName").Value);
        }
      }
      foreach (var s in d.XPathSelectElements("//Department[Parent_Dept_Code='" + dc + "']"))
        Run(s, nn.Nodes);
    }
  }
}
```
код работает, если в xml: "<Dept_Code1>51001586</Dept_Code1>" заменить на "<Dept_Code1>51001583</Dept_Code1>" - иначе нет связи между департаментом и должностью.

*> ошибка в строке varpn =re.Element("Personal_Number1");*

т.к. бывает, чтор re == null -- в ситуации, когда у подразделения нет должностей.
решение:
```c#
using System.Windows.Forms;
using System.Xml.Linq;
using System.Xml.XPath;

namespace WindowsFormsApplication1
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.Size = new System.Drawing.Size(600, 800);
      var tv = new TreeView { Parent = this, Dock = DockStyle.Fill };
      var xe = XElement.Load("..\\..\\data.xml");
      foreach (var d in xe.XPathSelectElements("//Department[Parent_Dept_Code='00000000']"))
        Run(d, tv.Nodes);
      tv.ExpandAll();
    }
    void Run(XElement d, TreeNodeCollection nc)
    {
      var dc = d.Element("Dept_Code").Value;
      var nn = nc.Add(d.Element("Full_Name1").Value);
      var re = d.XPathSelectElement("//Relation[Dept_Code1/text()='" + dc + "']");
      if (re != null)
      {
        var pn = re.Element("Personal_Number1");
        foreach (var t in re.Element("Titles_Codes").Elements("Title_Code2"))
        {
          foreach (var x in d.XPathSelectElements("//Title[Title_Code='" + t.Value + "']"))
          {
            var s = nn.Nodes.Add(x.Element("Full_Name").Value);
            var p = x.XPathSelectElement("//Staff[Personal_Number='" + pn.Value + "']");
            if (p != null)
              s.Nodes.Add(p.Element("LName").Value);
          }
        }
      }
      foreach (var s in d.XPathSelectElements("//Department[Parent_Dept_Code='" + dc + "']"))
        Run(s, nn.Nodes);
    }
  }
}
```
*> Нужно выводить начальников отделов и должности этого отдела, которые не должны  повторяются в пределах этого бюро*

```c#
using System;
using System.Linq;
using System.Windows.Forms;
using System.Xml.Linq;
using System.Xml.XPath;
namespace WindowsFormsApplication1
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.Size = new System.Drawing.Size(600, 800);
      var tv = new TreeView { Parent = this, Dock = DockStyle.Fill };
      var xe = XElement.Load("..\\..\\data.xml");
      Fill(tv.Nodes, xe.XPathSelectElement("//Department[Parent_Dept_Code='00000000']"));
      tv.ExpandAll();
    }
    void Fill(TreeNodeCollection nc, XElement xd)
    {
      Func<string, XElement> findTitle = code => xd.XPathSelectElement("//Title[Title_Code='" + code + "']");
      var dc = xd.Element("Dept_Code").Value;
      var nn = nc.Add(xd.Element("Short_Name1").Value);
      var rels =
          from x in xd.XPathSelectElements("//Relation[Dept_Code1='" + dc + "']")
          let position = findTitle(x.XPathSelectElement("Titles_Codes/Title_Code2[1]").Value)
          let chief = findTitle(position.XPathSelectElement("Chief_Title_Codes/Title_Code1[1]").Value)
          select new
          {
            Person = x.XPathSelectElement("//Staff[Personal_Number='" + x.Element("Personal_Number1").Value + "']"),
            Position = position,
            Chief = chief
          };
      var details =
          from x in rels
          select new
          {
            LastName = x.Person.Element("LName").Value,
            FirstName = string.Concat(x.Person.Element("FName").Value, " ", x.Person.Element("MName").Value),
            Title = x.Position.Element("Short_Name").Value,
          };
      foreach (var x in details.GroupBy(d => d.Title))
      {
        var tn = nn.Nodes.Add(x.Key);
        foreach (var p in x.OrderBy(n => n.LastName))
          tn.Nodes.Add(p.LastName + " " + p.FirstName);
      }
      foreach (var s in xd.XPathSelectElements("//Department[Parent_Dept_Code='" + dc + "']"))
        Fill(nn.Nodes, s);
    }
  }
}
```
