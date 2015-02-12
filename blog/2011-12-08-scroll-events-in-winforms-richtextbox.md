---
title: Scroll Events in Windows Form RichTextBox
tags: [c#, winforms]
src: https://social.msdn.microsoft.com/Forums/ru-RU/a921601b-f59b-485f-bb88-a7c8b7ccc876/scroll-events-in-windows-form-richtextbox?forum=winforms
---
# Scroll Events in Windows Form RichTextBox
*> I need a reliable method that will allow me to check if the user is at the bottom of the scroll bar*

try the following code:
```c#
using System.Drawing;
using System.Windows.Forms;
using System;

namespace WindowsFormsApplication9
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.Height = 120;
      var rtb = new RichTextBox
      {
        Parent = this,
        Dock = DockStyle.Fill,
        Text = "0\n1\n2\n3\n4\n5\n6\n7\n8\n9",
      };
      Func<string> info = () =>
      {
        var ci = rtb.GetCharIndexFromPosition(new Point(0, rtb.ClientSize.Height - 3));
        int li = rtb.GetLineFromCharIndex(ci);
        return "last " + li + ((li == rtb.Lines.Length - 1) ? " (bottom)" : "");
      };
      rtb.VScroll += (s, e) => this.Text = info();
      rtb.Resize += (s, e) => this.Text = info();
      this.Text = info();
    }
  }
}
```
*> All i did was put random text in the RTF box, each line ending with an Environment.NewLine*

so last line is empty. in this case try using the following code.
it works fine. i've tested it in the Visual Studio 2010, .NET Framework 4
```c#
using System;
using System.Drawing;
using System.Windows.Forms;

namespace WindowsFormsApplication9
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.Height = 120;
      var rtb = new RichTextBox
      {
        Parent = this,
        Dock = DockStyle.Fill,
      };
      for(int i=0; i < 50; i++)
        rtb.AppendText("text"+i+Environment.NewLine);
      Func<string> info = () =>
      {
        var ci = rtb.GetCharIndexFromPosition(new Point(0, rtb.ClientSize.Height - 3));
        int li = rtb.GetLineFromCharIndex(ci);
        return "last " + li + ((li == rtb.Lines.Length - 2) ? " (bottom)" : "");
      };
      rtb.VScroll += (s, e) => this.Text = info();
      rtb.Resize += (s, e) => this.Text = info();
      this.Text = info();
    }
  }
}
```