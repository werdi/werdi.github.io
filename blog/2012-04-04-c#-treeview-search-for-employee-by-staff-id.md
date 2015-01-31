---
title: C#. Treeview - поиск сотрудника по табельному номеру
tags: [c#, winforms]
src: https://social.msdn.microsoft.com/Forums/ru-RU/805f7da0-f72e-4632-ac92-8b55c61eae5a/ctreeview-?forum=fordesktopru
---
# C#. Treeview - поиск сотрудника по табельному номеру
*> вопрос о поиске сотрудника по  табельному (в xml есть поле Personal_Number)  остается открытым*

решение почти такое же как и с FamilyName. 
в наследника TreeNode надо добавить свойство PersonalNumber; значение берется из xml в методе Fill. 
для этого надо под "Title = x.Position.Element("Short_Name").Value" добавить:
```c#
Number = x.Person.Element("Personal_Number").Value
```

а также вместо "tn.Nodes.Add(new Person { Text = p.LastName+" "+p.FirstName, FamilyName = p.LastName });" указать: 
```c#
tn.Nodes.Add(new Person { Text = p.LastName + " " + p.FirstName, FamilyName = p.LastName, PersonalNumber = p.Number });
```

*> после того как  нашёл необходимого сотрудника [...] должны вывестись на экран данные из mysql, относящиеся именно к этому сотруднику*

для загрузки данных используется наследник DataAdapter. для mysql см. здесь (MySQL Connector/Net).
ему надо передать идентификатор сотрудника, например, его персональный номер.
пример передачи параметров [здесь] (http://msdn.microsoft.com/ru-ru/library/system.data.sqlclient.sqldataadapter.selectcommand.aspx)  
для вывода данных на экран см. [Связывание элементов управления Windows Forms с данными  ] (http://msdn.microsoft.com/ru-ru/library/ef2xyb33.aspx)
 
*> поиск табельному не работает.*
```c#
using System;
using System.Collections.Generic;
using System.Drawing;
using System.Linq;
using System.Windows.Forms;

namespace WindowsFormsApplication1
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.Size = new System.Drawing.Size(500, 500);
      var tv = new TreeView { Dock = DockStyle.Fill, Parent = this };
      Fill(tv.Nodes);
      tv.ExpandAll();

      var nodes = Flatten(tv.Nodes);
      var sel = new Selection(tv);

      var p = new Panel { Parent = this, Dock = DockStyle.Top, Height = 22 };
      var tb = new TextBox { Parent = p, Dock = DockStyle.Fill };

      tb.TextChanged += (s, e) =>
          sel.Select(nodes.OfType<Person>().Where(n => n.Text.IndexOf(tb.Text, StringComparison.OrdinalIgnoreCase) > -1));

      new Button { Text = "по фамилии", Dock = DockStyle.Right, Parent = p, Width = 100 }
        .Click += (s, e) =>
          sel.Select(nodes.OfType<Person>().Where(n =>
            tb.Text.Equals(n.FamilyName, StringComparison.OrdinalIgnoreCase)), true);

      new Button { Text = "по табельному", Dock = DockStyle.Right, Parent = p, Width = 100 }
        .Click += (s, e) =>
          sel.Select(nodes.OfType<Person>().Where(n =>
            tb.Text.Equals(n.Id.ToString(), StringComparison.OrdinalIgnoreCase)), true);

      this.Shown += delegate { tb.Focus(); };
    }

    IEnumerable<TreeNode> Flatten(TreeNodeCollection tc)
    {
      foreach (TreeNode n in tc)
      {
        yield return n;
        foreach (var c in Flatten(n.Nodes)) yield return c;
      }
    }

    private void Fill(TreeNodeCollection nc)
    {
      var n1 = nc.Add("Отдел САПР").Nodes.Add("Инженер");
      n1.Nodes.Add(new Person { Text = "Иванов Иван", FamilyName = "Иванов", Id = 123 });
      n1.Nodes.Add(new Person { Text = "Петров Петр", FamilyName = "Петров", Id = 321 });
    }
  }

  class Person : TreeNode
  {
    public string FamilyName { get; set; }
    public int Id { get; set; }
  }

  class Selection
  {
    TreeView TreeView;
    List<TreeNode> Selected;

    public Selection(TreeView tv)
    {
      this.TreeView = tv;
      this.Selected = new List<TreeNode>();
    }

    public void Select(IEnumerable<TreeNode> nodes, bool activate = false)
    {
      foreach (var n in this.Selected) n.BackColor = Color.Transparent;
      this.Selected.Clear();
      foreach (var n in nodes)
      {
        n.BackColor = Color.WhiteSmoke;
        this.Selected.Add(n);
      }

      if (this.Selected.Count > 0 && activate)
      {
        var n = this.Selected[0];
        n.TreeView.SelectedNode = n;
        n.TreeView.Focus();
      }
    }
  }
}
```
