---
title: How to raise an event by its name?
tags: [c#, winforms, interop]
src: https://social.msdn.microsoft.com/Forums/ru-RU/b1912411-f722-415a-b5ae-dd32bff10efd/how-to-raise-an-event-by-its-name?forum=csharpgeneral
---
# How to raise an event by its name?
*> My project has about 200 form (or user control), and in each form(or user controller) has about 10 buttons. Users just click on the buttons which they are allowed to use, but the code in each event hasn's been processed to do it. It takes a lot of time to change all of events, so i have to find out a way to solve it.*

there is another way: process windows message, find button and disable it.
```c#
using System;
using System.Drawing;
using System.Runtime.InteropServices;
using System.Windows.Forms;

namespace WindowsFormsApplication2
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      ...
    }

    protected override void WndProc(ref Message m)
    {
      // 0x210 - WM_PARENTNOTIFY; 0x0201 - WM_LBUTTONDOWN
      if (m.Msg == 0x210 && (int) m.WParam == 0x0201)
      {
        var wnd = WindowFromPoint(Control.MousePosition);
        var b = Control.FromHandle(wnd) as Button;  
        if(b != null)
        {
          if( ... )
            b.Enabled = false;
        }
      }
      base.WndProc(ref m);
    }

    [DllImport("user32.dll")]
    static extern IntPtr WindowFromPoint(Point point);
  }
}
```
also there is another way based on IMessageFilter
```c#
using System;
using System.Windows.Forms;

namespace WindowsFormsApplication2
{
  static class Program
  {
    class Filter : IMessageFilter
    {
      public bool PreFilterMessage(ref Message m)
      {
        // mouse left button messages  
        if (m.Msg >= 513 && m.Msg <= 515) 
        {
          var b = Control.FromHandle(m.HWnd) as Button;
          if (b != null)
          {
            if (b.Name == "b1")
              return true;    // block messages
          }
        }
        return false;
      }
    }

    [STAThread]
    static void Main()
    {
      Application.AddMessageFilter(new Filter());
      Application.EnableVisualStyles();
      Application.SetCompatibleTextRenderingDefault(false);
      Application.Run(new Form1());
    }
  }
}
...
```
*> All your buttons should have a seperated unique name in your complete application.*

Buttons can be distinguished by the Type.FullName or/and by Text property