---
title: Custom control dilemma in C#
tags: [c#, winforms, graphics]
src: https://social.msdn.microsoft.com/Forums/ru-RU/9fab4264-fb49-43cf-9fbf-fc7d46db70d1/custom-control-dilemma-in-c?forum=csharpgeneral 
---
# Custom control dilemma in C#
*> I want to ask how to make something like that in C# .NET (I'm using Visual Studio 2010):*

try using the following example.
```c#
using System;
using System.Collections.Generic;
using System.Drawing;
using System.Drawing.Drawing2D;
using System.Linq;
using System.Windows.Forms;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    class Shape
    {
      public GraphicsPath Path { get; set; }
      public Color Stroke { get; set; }
      public Action Action { get; set; }
    }

    public Form1()
    {
      var shapes = new List<Shape>();

      var p1 = new GraphicsPath();
      p1.AddEllipse(new Rectangle(5, 5, 20, 20));
      shapes.Add(new Shape { 
        Path = p1, Stroke = Color.Red, Action = () => MessageBox.Show("shape1") });

      var p2 = new GraphicsPath();
      p2.AddEllipse(new Rectangle(30, 5, 20, 20));
      shapes.Add(new Shape { 
        Path = p2, Stroke = Color.Navy, Action = () => MessageBox.Show("shape2") });

      this.Paint += (s, e) =>
      {
        foreach(var c in shapes)
        {
          // instead of following code you can paint any images
          using(var p = new Pen(c.Stroke))
            e.Graphics.DrawPath(p, c.Path);
        }
      };

      this.MouseClick += (s, e) =>
      {
        var res = shapes.Where(c => c.Path.IsVisible(e.Location)).FirstOrDefault();
        if(res != null)
          res.Action();
      };
    }
  }
}
```
