---
title: Плавная перерисовка графики
tags: [c#, graphics, winforms]
src: https://social.msdn.microsoft.com/Forums/ru-RU/b676f743-0347-47cf-adea-d18235336a70/-?forum=programminglanguageru
---
# Плавная перерисовка графики
*> Подскажите как правильнее было бы вывести чтобы не происходило мерцания?*

создайте контрол, в котором надо указать Styles.OptimizedDoubleBuffer
в метод OnPaint оставьте только вывод фигур/графиков. примерно так:
```c#
class Canvas : UserControl
{
  public readonly List<Shape> Shapes;
  public Canvas()
  {
    this.SetStyle(ControlStyles.UserPaint | ControlStyles.OptimizedDoubleBuffer | ControlStyles.AllPaintingInWmPaint, true);
    this.Shapes = new List<Shape>();
  }
  protected override void OnPaint(PaintEventArgs e)
  {
    base.OnPaint(e);
    foreach (var s in this.Shapes) s.Draw(e.Graphics);
  }
}

abstract class Shape
{
  public Point Location { get; set; }
  public abstract void Draw(Graphics g);
  public abstract bool IsVisible(Point p);
}
```
*> двигать графики мышкой [...] было бы замечательно если бы при сдвиге графика, график перерисовывался плавно..*
```c#
using System.Collections.Generic;
using System.Drawing;
using System.Drawing.Drawing2D;
using System.Windows.Forms;

namespace WindowsFormsApplication1
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      InitializeComponent();
      var c = new Canvas { Parent = this, Dock = DockStyle.Fill };
      c.Shapes.Add(new Graph1 { Pen = Pens.Red });
      c.Shapes.Add(new Graph2 { Pen = Pens.Green });
    }
  }

  class Canvas : UserControl
  {
    public readonly List<Shape> Shapes;
    public Canvas()
    {
      this.SetStyle(ControlStyles.UserPaint | ControlStyles.OptimizedDoubleBuffer | ControlStyles.AllPaintingInWmPaint, true);
      this.Shapes = new List<Shape>();
    }
    protected override void OnPaint(PaintEventArgs e)
    {
      base.OnPaint(e);
      e.Graphics.SmoothingMode = SmoothingMode.HighQuality;
      foreach (var s in this.Shapes)
        s.Draw(e.Graphics, s == this.ActiveShape);
    }
    protected override void OnMouseDown(MouseEventArgs e)
    {
      base.OnMouseDown(e);
      this.LastPoint = e.Location;
    }
    protected override void OnMouseMove(MouseEventArgs e)
    {
      base.OnMouseMove(e);
      if (e.Button == MouseButtons.Left)
      {
        if(this.ActiveShape != null)
        {
          this.ActiveShape.Move(e.Location.X - this.LastPoint.X, e.Location.Y - this.LastPoint.Y);
          this.LastPoint = e.Location;
          this.Invalidate();
        }
      }
      else
      {
        this.ActiveShape = null;
        foreach (var s in this.Shapes)
        {
          if (s.IsVisible(e.Location))
          {
            this.ActiveShape = s;
            break;
          }
        }
      }
    }
    Shape ActiveShape
    {
      get { return _ActiveShape; }
      set
      {
        if (_ActiveShape != value)
        {
          _ActiveShape = value;
          this.Invalidate();
        }
      }
    }
    Shape _ActiveShape;
    Point LastPoint;
  }

  abstract class Shape
  {
    public abstract void Draw(Graphics g, bool active);
    public abstract bool IsVisible(Point p);
    public abstract void Move(int dx, int dy);
  }

  class Graph : Shape
  {
    protected GraphicsPath Path;
    public Pen Pen;
    public Graph() { this.Path = new GraphicsPath(); }
    public override void Draw(Graphics g, bool active)
    {
      if (active)
      {
        using (var p = new Pen(this.Pen.Brush, this.Pen.Width * 2))
          g.DrawPath(p, this.Path);
      }
      else
          g.DrawPath(this.Pen, this.Path);
    }
    public override bool IsVisible(Point p) { return Path.IsVisible(p); }
    public override void Move(int dx, int dy)
    {
      var m = new Matrix();
      m.Translate(dx, dy);
      this.Path.Transform(m);
    }
  }

  class Graph1 : Graph
  {
    public Graph1()
    {
      this.Path.AddLine(10, 10, 50, 50);
      this.Path.AddLine(50, 50, 100, 25);
    }
  }

  class Graph2 : Graph
  {
    public Graph2()
    {
      this.Path.AddCurve(new Point[] { 
        new Point(10, 30), new Point(30, 10), new Point(60, 10), new Point(100, 50) });
    }
  }
}
```
