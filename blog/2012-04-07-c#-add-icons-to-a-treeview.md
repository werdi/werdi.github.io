---
title: C#. Treeview - добавление иконок
tags: [c#, winforms, treeview]
src: https://social.msdn.microsoft.com/Forums/ru-RU/7a809ea6-2d2c-4734-b185-aa312dac3b03/ctreeview-?forum=fordesktopru
---
# C#. Treeview - добавление иконок
*> Имеется treeview [...] Как добавить 4 вида иконкок для отделов, бюро, должностей, сотрудников?*
```c#
using System.Drawing;
using System.Windows.Forms;

namespace WindowsFormsApplication1
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var lst = new ImageList();
      lst.Images.Add(Image.FromFile("1.png"));
      lst.Images.Add(Image.FromFile("2.png"));
      
      TreeView tv = new TreeView { Parent = this, Dock = DockStyle.Fill };
      tv.ImageList = lst;
      
      tv.Nodes.Add(new TreeNode { Text = "n1", ImageIndex = 0 });
      tv.Nodes.Add(new TreeNode { Text = "n2", ImageIndex = 1 });
    }
  }
}
```
*> Спс, но как мне применить данное предложение для следующего кода?*

надо добавить примерно следующее: 
```c#
enum NodeType
{
  отдел,
  бюро,
  должность,
  сотрудник,
}

void Fill(TreeNodeCollection nc, XElement xd)
{  
  ...  
  var tn = nn.Nodes.Add(x.Key);
  tn.ImageIndex = (int) GetNodeType(xd);
  ...
}

NodeType GetNodeType(XElement xe)
{
  ... на основе данных в xe определить и вернуть тип узла
} 
```
*> на основе данных в xe определить и вернуть тип узла. Как это сделать?*

названии проверяйте наличие слов "отдел", "бюро". 
если нет, то остается либо должность, либо фио. 
в строке: var tn = nn.Nodes.Add(x.Key); -- должность. 
в строке: tn.Nodes.Add(...) -- фио.
