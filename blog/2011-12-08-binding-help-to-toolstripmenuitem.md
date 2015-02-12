---
title: Каким образом к ToolStripMenuItem в MenuStrip сделать привязку Help?
tags: [c#, winforms]
src: https://social.msdn.microsoft.com/Forums/ru-RU/17b50d36-70c6-424e-b0db-5a35d2de61d1/-toolstripmenuitem-menustrip-help?forum=fordesktopru
---
# Каким образом к ToolStripMenuItem в MenuStrip сделать привязку Help?
*> необходимо сделать привязку help.chm к каждому пункту меню - при нажатии на f1.*

в форме надо определить ProcessDialogKey ...
```c#
public partial class Form1 : Form
{
  protected override bool ProcessDialogKey(Keys keyData)
  {
    if(keyData == Keys.F1)
    {
      // ...
      return true;
    }
    return base.ProcessDialogKey(keyData);
  }
}
```
*> попробовал, но не работает метод. Когда заходишь в меню - оно перехватывает ввод всех клавиш.*
```c#
using System.Windows.Forms;

namespace WindowsFormsApplication5
{
  public partial class Form1 : Form
  {
    MenuStrip ms;

    public Form1()
    {
      ms = new MenuStrip { Parent = this };
      ms.Items.AddRange(new[] {
          new ToolStripButton { Text = "test1" },
          new ToolStripButton { Text = "test2" }
        });
    }

    protected override bool ProcessDialogKey(Keys keyData)
    {
      if (keyData == Keys.F1)
      {
        var itm = ms.GetItemAt(ms.PointToClient(Control.MousePosition));
        System.Diagnostics.Trace.WriteLine(keyData + " " + itm);
        return true;
      }
      return base.ProcessDialogKey(keyData);
    }
  }
}
```