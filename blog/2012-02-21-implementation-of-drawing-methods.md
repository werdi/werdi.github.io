---
title: Как правильно нужно оформить метод для рисования на C#?
tags: [c#, drawing, winforms]
src: https://social.msdn.microsoft.com/Forums/ru-RU/35954079-a6e7-4f13-9dde-cf8bdd9dd5fa/-c?forum=fordesktopru
---
# Как правильно нужно оформить метод для рисования на C#?
*> небольшую программу которая должна давать пользователю возможность рисовать мышкой.*

пример позволяет рисовать мышью. сохраняет линии. и подсвечивать линии под мышью.
```c#
using System.Collections.Generic;
using System.Drawing;
using System.Drawing.Drawing2D;
using System.Windows.Forms;
using System.Linq;

namespace WindowsFormsApplication4
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.SetStyle(ControlStyles.AllPaintingInWmPaint | ControlStyles.OptimizedDoubleBuffer | ControlStyles.UserPaint, true);
      this.Shapes = new List<GraphicsPath>();
      this.Points = new List<Point>();
    }

    List<GraphicsPath> Shapes;
    List<Point> Points;

    protected override void OnMouseMove(MouseEventArgs e)
    {
      base.OnMouseMove(e);
      if (e.Button == System.Windows.Forms.MouseButtons.Left)
      {
        this.Points.Add(e.Location);
        this.Invalidate();
      }
      else
      {
        if(this.Shapes.Any(x => x.IsVisible(e.Location)))
          this.Invalidate();
      }
    }

    protected override void OnMouseUp(MouseEventArgs e)
    {
      base.OnMouseUp(e);
      if(this.Points.Count > 2)
      {
        var gp = new GraphicsPath();
        gp.AddCurve(this.Points.ToArray());
        this.Shapes.Add(gp);

        this.Points = new List<Point>();
      }
    }

    protected override void OnPaint(PaintEventArgs e)
    {
      base.OnPaint(e);
      
      e.Graphics.SmoothingMode = SmoothingMode.AntiAlias;
      if (this.Points.Count > 2)
      {
        using(var p = new Pen(Color.Red, 2))
          e.Graphics.DrawCurve(p, Points.ToArray());
      }
      foreach(var gp in this.Shapes)
      {
        var c = gp.IsVisible(this.PointToClient(Control.MousePosition)) ? Color.Blue : Color.Silver;
        using(var p = new Pen(c))
          e.Graphics.DrawPath(p, gp);
      }
    }
  }
}
```
