---
title: Find the object under the mouse
tags: [c#, winforms]
src: https://social.msdn.microsoft.com/Forums/ru-RU/3df9fc84-4020-4ae1-8b4f-942bce65568f/find-the-object-under-the-mouse?forum=winforms
---
# Find the object under the mouse
*> Imagine a Form with a lot of controls in it. When I right click on it, a context menu appears. By the given location of the mouse i want a new command to show up to my contexMenu relative to the underlying object*
```c#
using System.ComponentModel;
using System.Windows.Forms;
using System.Drawing;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      Application.AddMessageFilter(new Filter());

      Control last = this;
      var colors = new[] { Color.Red, Color.Green, Color.Blue };
      for (int i = 0; i < colors.Length; i++)
      {
        last = new Panel
        {
          Parent = last,
          Dock = DockStyle.Fill,
          Padding = new Padding(50),
          BackColor = Color.FromArgb(150, colors[i]),
          Name = "Panel " + i
        };
      }
    }
  }

  class Filter : IMessageFilter
  {
    public bool PreFilterMessage(ref Message m)
    {
      if(m.Msg == 517)    // WM_RBUTTONUP 
      {
        var p = Control.FromHandle(m.HWnd) as Panel;
        if(p != null)
        {
          var cm = new ContextMenu();
          cm.MenuItems.Add(p.Name);
          cm.Show(p, p.PointToClient(Control.MousePosition));
        }
      }
      return false;
    }
  }
}
```
*> I'm aware of how to get every object that exists under the mouse*

use [WindowFromPoint] (http://msdn.microsoft.com/en-us/library/windows/desktop/ms633558(v=vs.85).aspx) Function and [Control.FromHandle] (http://msdn.microsoft.com/en-us/library/system.windows.forms.control.fromhandle.aspx) Method.
```c#
using System.Runtime.InteropServices;
using System.Windows.Forms;
...
[DllImport("user32.dll")]
static extern IntPtr WindowFromPoint(Point point);
...
var hwnd = WindowFromPoint(Control.MousePosition);
var c = Control.FromHandle(hwnd);
if(c != null)
{
}
```
