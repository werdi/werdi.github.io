---
title: Key Press Error in C#.net
tags: [c#, winforms]
src: https://social.msdn.microsoft.com/Forums/ru-RU/3943b61b-c947-450d-b09a-46af139062a7/key-press-error-in-cnet?forum=winforms
---
# Key Press Error in C#.net
*> i also use this code but its not working in run time.*
try using the following code. it works fine.
```c#
using System.Windows.Forms;

namespace WindowsFormsApplication6
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      new TextBox { Parent = this, Dock = DockStyle.Top }
        .KeyDown += (s, e) =>
          {
            if (e.KeyCode == Keys.Enter)
              MessageBox.Show("Enter pressed", "Attention");
          };
    }
  }
}
```
*> But when i use textbox1 and textbox2 in Form1 then whats procedure?*

if you need to share one handler for several textbox's, then try using the following
```c#
using System.Windows.Forms;
using System.Data;

namespace WindowsFormsApplication6
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      new TextBox { Parent = this, Dock = DockStyle.Top, Name = "textbox1" }
        .KeyDown += TextBox_KeyDown;
      new TextBox { Parent = this, Dock = DockStyle.Top, Name = "textbox2" }
        .KeyDown += TextBox_KeyDown;
    }

    void TextBox_KeyDown(object sender, KeyEventArgs e)
    {
      if (e.KeyCode == Keys.Enter)
        MessageBox.Show("Enter pressed in " + (sender as TextBox).Name, "Attention");
    }
  }
}
```