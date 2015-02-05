---
title: How to disable the paste in textbox
tags: [c#, winforms, interop]
src: https://social.msdn.microsoft.com/Forums/ru-RU/4d0455b4-52b8-426a-a830-5dca9f3911cb/how-to-disable-the-paste-in-textbox?forum=winforms 
---
# How to disable the paste in textbox
*> How to disable the paste in the textbox with allow typing?*

you can handle WM_PASTE message as suggested above.
it's easy to do in the IMessageFilter.PreFilterMessage Method
```c#
using System;
using System.Windows.Forms;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form, IMessageFilter
  {
    public Form1()
    {
      Application.AddMessageFilter(this);
      // ...
    }

    public bool PreFilterMessage(ref Message m)
    {
      if(m.Msg == 0x0302)    // WM_PASTE
      {
        // ...
      } 
      return false;
    }
  }
}

unfortunately this (WM_PASTE) doesn't work for TextBox.
but you can handle the TextBox.TextChanged Event
below is an example.
```c#
using System;
using System.Windows.Forms;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var tb = new TextBox { Parent = this };
      var old = "";
      var pasted = false;
      tb.TextChanged += (s, e) =>
      {
        if (pasted) return;
        if (Math.Abs(tb.TextLength - old.Length) > 1)
        {
          pasted = true;
          tb.Text = old;
          pasted = false;
        }
        old = tb.Text;
      };
      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("paste", (s, e) => tb.Paste("test"));
    }
  }
}
```
also you can intercept WM_PASTE in the NativeWindow
below is an example.
```c#
using System.Windows.Forms;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var tb = new TextBox { Parent = this, Location = new System.Drawing.Point(10, 10) };
      _Ntb = new NativeTextBox();
      _Ntb.AssignHandle(tb.Handle);
    }
    NativeTextBox _Ntb;
}
class NativeTextBox : NativeWindow
{
    protected override void WndProc(ref Message m)
    {
      if (m.Msg == 0x0302) return;
      base.WndProc(ref m);
    }
  }
}
```
but i'm curious why WM_PASTE doesn't intercept in the  
IMessageFilter.PreFilterMessage as mentioned above?
