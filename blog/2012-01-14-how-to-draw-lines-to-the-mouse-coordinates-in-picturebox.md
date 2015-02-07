---
title: How to draw lines to the mouse coordinates in picturebox?
tags: [c#, winforms, graphics]
src: https://social.msdn.microsoft.com/Forums/ru-RU/70b5ec2f-be69-43e8-9d3a-545c29137869/how-to-draw-lines-to-the-mouse-coordinates-in-picturebox?forum=csharpgeneral
---
# How to draw lines to the mouse coordinates in picturebox?
*> I have a problemwith drawing straight lines [...] in the PictureBox.Iwant to draw a line from one point to the current coordinates of the mouse.*
```c#
using System.Collections.Generic;
using System.Drawing;
using System.Windows.Forms;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    class Line
    {
      public Point Start { get; set; }
      public Point End { get; set; }
    }

    public Form1()
    {
      var pb = new PictureBox { Parent = this, Dock = DockStyle.Fill };
      Line line = null;

      pb.MouseMove += (s, e) =>
      {
        if (e.Button == MouseButtons.Left)
        {
          line.End = e.Location;
          pb.Invalidate();
        }
      };
      pb.MouseDown += (s, e) =>
      {
        if (e.Button == MouseButtons.Left)
          line = new Line { Start = e.Location, End = e.Location };
      };
      var lines = new List<Line>();
      pb.MouseUp += (s, e) =>
      {
        if (e.Button == MouseButtons.Left)
          lines.Add(line);
      };
      pb.Paint += (s, e) =>
      {
        if (line != null)
          e.Graphics.DrawLine(Pens.Red, line.Start, line.End);

        foreach(var l in lines)
          e.Graphics.DrawLine(Pens.Silver, l.Start, l.End);
      };
    }
  }
}
```
