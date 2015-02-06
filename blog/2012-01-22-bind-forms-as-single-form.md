---
title: Bind Forms as Single Form
tags: [c#, winforms]
src: https://social.msdn.microsoft.com/Forums/ru-RU/a88a2334-62a3-4ae6-b771-66604773e96d/bind-forms-as-single-form?forum=winformsdatacontrols 
---
# Bind Forms as Single Form
*> I want to bind one form with 2nd, so that forms can be activated as single form. [...] Form1 (Main Form): Visible in Taskbar Form2: Not visible in Taskbar*

you can use Form.Owner property
below is an example
```c#
using System.Drawing;
using System.Windows.Forms;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      new Button { Parent = this, Location = new Point(10, 10), Text = "Form2" }
        .Click += (s, e) =>
          {
            var frm = new Form2 { Owner = this };
            frm.Show();
          };
    }
  }
  class Form2 : Form
  {
    public Form2()
    {
      this.ShowInTaskbar = false;
    }
  }
}
```
