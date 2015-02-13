---
title: C#. Treeview - организовать поиск
tags: [c#, winforms, treeview, xml, linq]
src: https://social.msdn.microsoft.com/Forums/ru-RU/4f749320-835d-4fa6-948e-85310dfeefc9/ctreeview-?forum=fordesktopru
---
# C#. Treeview - организовать поиск
*> организовать поиск по табельному и Фио сотрудника, которые будут вводиться в textbox?*

в предыдущем примере: public Form1() {...} -- замените следующим кодом:
```c#
public Form1()
{
  this.Size = new System.Drawing.Size(600, 800);
  var tv = new TreeView { Parent = this, Dock = DockStyle.Fill };
  var xe = XElement.Load("..\\..\\data.xml");
  Fill(tv.Nodes, xe.XPathSelectElement("//Department[Parent_Dept_Code='00000000']"));
  tv.ExpandAll();
  var nodes = Flatten(tv.Nodes);
  var selected = new List<TreeNode>();
  new TextBox { Parent = this, Dock = DockStyle.Top }
    .TextChanged += (s, e) =>
    {
      var tb = s as TextBox;
      foreach (var n in selected) n.BackColor = Color.Transparent;
      selected.Clear();
      if (string.IsNullOrWhiteSpace(tb.Text))
        return;
      foreach (var n in nodes.Where(tn => tn.Text.IndexOf(tb.Text, StringComparison.OrdinalIgnoreCase) > -1))
      {
        n.BackColor = Color.AntiqueWhite;
        selected.Add(n);
      }
    };
}
IEnumerable<TreeNode> Flatten(TreeNodeCollection tc)
{
  foreach (TreeNode n in tc)
  {
    yield return n;
    foreach (var c in Flatten(n.Nodes)) yield return c;
  }
}
```
если надо подсветить/выделить часть TreeNode.Text, то см. [здесь](http://social.msdn.microsoft.com/Forums/en-US/winforms/thread/7e7b25bd-7adf-43c1-8546-08a308084cf5/#b2b2dbd6-94ff-4725-9f61-970914fe28a6)

*> После того как  нашёл необходимого сотрудника по событию afterselect должна появляться новая форма, привязанная именно для этого сотрудника.*

в конструктор Form1 надо добавить примерно следующий код:
```c#
tv.AfterSelect += (s, e) => 
{
  var frm = new SomeForm(e.Node);
  frm.ShowDialog();
};
```
или примерно такой код:
```c#
var p = new Panel { Parent = this, Dock = DockStyle.Bottom, Height = 200 };
tv.AfterSelect += (s, e) => 
{
  p.Controls.Clear();
  new Label { Parent = p, Text = e.Node.Text };   // далее поля ввода
}; 
```

*> не определяет var nodes = Flatten(tv.Nodes); - что за flatten???*

выше по теме есть определение метода Flatten.
```c#
IEnumerable<TreeNode> Flatten(TreeNodeCollection tc)
{
  foreach (TreeNode n in tc)
  {
    yield return n;
    foreach (var c in Flatten(n.Nodes)) yield return c;
  }
} 
```

*> А как сделать помимо выделения дерево раскрылось  и указатель был на в ведённом сотруднике?*

в обработчик TextChanged (см. выше в примере) надо добавить примерно такой код: 
```c#
var sn = nodes.Where(tn => tn.Text.IndexOf(tb.Text, StringComparison.OrdinalIgnoreCase) > -1).FirstOrDefault();
if(sn != null)
  sn.TreeView.SelectedNode = sn;
```

*> нужен поиск только по фамилии.*

т.е. в узлах надо хранить информацию о типе данных. для этого добавляйте в дерево наследников TreeNode. например, 
```c#
class Person : TreeNode { public string FamilyName { get; set; } }
```
при поиске узлов проверяйте тип узла и есть ли совпадение. примерно так: 
```c#
Person p = node as Person;
if(p != null && p.FamilyName.IndexOf(txt) > -1)
{
  p.TreeView.SelectedNode = p;
}
```
*> организовать поиск по табельному сотрудника и фамилии, т.е в xml есть поле Personal_Number и LastName*

как уже говорил, надо создать наследников TreeNode. в них надо хранить дополнительную информацию. при поиске по дереву проверять тип узла и выполнять необходимы действия.

*> Нашли первого сотрудника и выделили, затем ищем 2-го и его тоже выделили, а как снять выделение с 1-го и со всех предыдущих???*

выше в примере см. var selected = new List<TreeNode>(); 
предназначен для снятия выделение при каждом новом поиске. 

*> var selected = new List<TreeNode>(); - предназначен, но почему не очищает?*

selected должен быть определен за пределами обработчика button1_Click, примерно так: 
```c#
List<TreeNode> selected = new List<TreeNode>();
private void button1_Click(object sender, EventArgs e)
{
}
```

*> создать наследников TreeNode. в них надо хранить дополнительную информацию. при поиске по дереву проверять тип узла и выполнять необходимы действия.*

примерно так.
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
      var tv = new TreeView { Dock = DockStyle.Fill, Parent = this };
      var td = new Department() { Text = "Отдел" };
      var ts = new Staff() { Text = "Должность" };
      var p1 = new Person() { Text = "11 22 33", FamilyName = "11" };
      var p2 = new Person() { Text = "Qq Ww 33", FamilyName = "Qq" };
      td.Nodes.Add(ts);
      ts.Nodes.Add(p1);
      ts.Nodes.Add(p2);
      tv.Nodes.Add(td);
      tv.ExpandAll();

      var nodes = Flatten(tv.Nodes);
      var selected = new List<TreeNode>();
      var tb = new TextBox { Dock = DockStyle.Top, Parent = this };
      this.Shown += delegate { tb.Focus(); };
      tb.TextChanged += (s, e) =>
      {
        // убрать фон у выделенных узлов
        foreach (var n in selected) n.BackColor = Color.Transparent;
        selected.Clear();
        if (string.IsNullOrWhiteSpace(tb.Text))
          return;

        // найти и выделить узлы типа Person, в которых встречается введенный текст
        foreach (var n in nodes.OfType<Person>().Where(n => n.Text.IndexOf(tb.Text, StringComparison.OrdinalIgnoreCase) > -1))
        {
          n.BackColor = Color.Silver;
          selected.Add(n);
        }

        // выделить узел типа Person, у которого свойство FamilyName == tb.Text
        var sn = selected.OfType<Person>().Where(n => n.FamilyName.Equals(tb.Text, StringComparison.OrdinalIgnoreCase)).FirstOrDefault();
        if (sn != null)
        {
          tv.SelectedNode = sn;
          tv.Focus();
        }
      };
    }
  
    class Department : TreeNode { }
    class Staff : TreeNode { }
    class Person : TreeNode { public string FamilyName { get; set; } }
  
    IEnumerable<TreeNode> Flatten(TreeNodeCollection tc)
    {
      foreach (TreeNode n in tc)
      {
        yield return n;
        foreach (var c in Flatten(n.Nodes)) yield return c;
      }
    }
  }
}
```
