---
title: show data in two kinds in a dialog such as chart in wpf
tags: [c#, xaml, wpf, chart]
src: https://social.msdn.microsoft.com/Forums/ru-RU/fe4c3847-6cf2-49a9-b447-070a58293a1b/show-datas-in-two-kinds-in-a-dialog-such-as-chart-in-wpf?forum=wpf
---
# show data in two kinds in a dialog such as chart in wpf
*> I need show some data in a chart in my project. this data is from two kinds , time and doubles [...]  I should set the times as X value and the values in double as Y value*

if I understood correctly you need something like below:
(whitesmoke rectangle - time; red rectagles - values)
![img1] (http://social.msdn.microsoft.com/Forums/getfile/90076)
```xaml
<Window x:Class="WpfApplication2.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="300" Width="500">
  <Border Margin="10,10,10,50" BorderThickness="1" BorderBrush="Silver">
    <ItemsControl ItemsSource="{Binding}">
      <ItemsControl.ItemsPanel>
        <ItemsPanelTemplate>
          <Canvas />
        </ItemsPanelTemplate>
      </ItemsControl.ItemsPanel>
      <ItemsControl.ItemTemplate>
        <DataTemplate>
          <!-- -->
          <ItemsControl ItemsSource="{Binding Doubles}" 
            Background="WhiteSmoke"  
            Height="{Binding RelativeSource={RelativeSource Mode=FindAncestor, AncestorType={x:Type Canvas}}, Path=ActualHeight}" 
            Width="25">
            <ItemsControl.ItemsPanel>
              <ItemsPanelTemplate>
                <Canvas />
              </ItemsPanelTemplate>
            </ItemsControl.ItemsPanel>
            <ItemsControl.ItemTemplate>
              <DataTemplate>
                <Rectangle Fill="Red" Height="3" Width="25" />
              </DataTemplate>
            </ItemsControl.ItemTemplate>
            <ItemsControl.ItemContainerStyle>
              <Style>
                <Setter Property="Canvas.Bottom" Value="{Binding}" />
              </Style>
                  </ItemsControl.ItemContainerStyle>
              </ItemsControl>
              <!--  -->
            </DataTemplate>
          </ItemsControl.ItemTemplate>
          <ItemsControl.ItemContainerStyle>
          <Style>
            <Setter Property="Canvas.Left" Value="{Binding Path=Time}" />
            <Setter Property="Canvas.Bottom" Value="0" />
          </Style>
        </ItemsControl.ItemContainerStyle>
      </ItemsControl>
  </Border>
</Window>
```
```c#
using System.Windows;

namespace WpfApplication2
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();

      this.DataContext = new[]  // test data
      {
        new { Time = 100, Doubles=new [] { 10.5, 20.2 } },
        new { Time = 150, Doubles=new [] { 40.7, 80.2, 90.3 } },
        new { Time = 250, Doubles=new [] { 20.1, 150.8 } }
      };
    }
  }
}
```
*> we just need see the curve line.*
```xaml
<Window x:Class="WpfApplication2.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="350" Width="500">
  <Grid x:Name="grd">
    <Polyline Stroke="Red" StrokeThickness="2" Points="{Binding Data}">
      <Polyline.LayoutTransform>
        <ScaleTransform  ScaleY="-1" />
      </Polyline.LayoutTransform>
    </Polyline>
    <ItemsControl ItemsSource="{Binding Data}">
      <ItemsControl.ItemsPanel>
        <ItemsPanelTemplate>
          <Canvas />
        </ItemsPanelTemplate>
      </ItemsControl.ItemsPanel>
      <ItemsControl.ItemTemplate>
        <DataTemplate>
          <Ellipse Width="8" Height="8" Fill="White" Stroke="Red" StrokeThickness="1">
            <Ellipse.RenderTransform>
              <TranslateTransform X="-4" Y="4" />
            </Ellipse.RenderTransform>
          </Ellipse>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
      <ItemsControl.ItemContainerStyle>
        <Style>
          <Setter Property="Canvas.Left" Value="{Binding X}" />
          <Setter Property="Canvas.Bottom" Value="{Binding Y}" />
        </Style>
      </ItemsControl.ItemContainerStyle>
    </ItemsControl>
  </Grid>
</Window>
```
```c#
using System;
using System.Linq;
using System.Windows;
using System.Windows.Media;
using System.Windows.Media.Imaging;

namespace WpfApplication2
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      
      // test data
      var xa = new [] { 
          0.08, 0.16, 0.21, 0.24, 0.26, 0.32, 0.37, 0.43, 0.52, 0.61, 0.67, 0.73, 0.79, 0.85, 0.95, 1.0 };
      var ya = new [] { 
          0.07, 0.16, 0.28, 0.39, 0.52, 0.65, 0.75, 0.82, 0.85, 0.86, 0.82, 0.76, 0.69, 0.64, 0.58, 0.57 };
      
      var scale = 300.0;
      var pa = xa.Zip(ya, (xi, yi) => new Point(Math.Ceiling(xi*scale), Math.Ceiling(yi*scale)));
      this.DataContext = new { Data = new PointCollection(pa) };
      grd.Background =  GridHelper.CreateGrid(25, 25, Brushes.Navy, Brushes.Blue);
    }
  }
}
```
please, note: CreateGrid Method is [here] (http://social.msdn.microsoft.com/Forums/en/wpf/thread/dbc9175f-e8c4-4145-9902-6cc9daa9fcdf/#74970f69-d12f-4475-8851-1aef058792a3).

![img2] (http://social.msdn.microsoft.com/Forums/getfile/90155)

you can remove ellipses, for this just remove ItemsControl from xaml above.

*> I have another question : how can show the axises and how can set the units for x and y ?*

![img3] (http://social.msdn.microsoft.com/Forums/getfile/91116)

```xaml
<Window x:Class="WpfApplication2.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="350" Width="550">
  <Border BorderBrush="Silver" BorderThickness="1" Margin="20">
    <Canvas x:Name="ca" Margin="20" />
  </Border>
</Window>
```
```c#
using System;
using System.Globalization;
using System.Threading;
using System.Windows;
using System.Windows.Media;
using System.Windows.Media.Imaging;
namespace WpfApplication2
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      ca.Background = MeasuringGrid.Create(10, 5, 40, 0.1, 12);
    }
  }

  class MeasuringGrid
  {
    public static Brush Create(int xCellCount, int yCellCount, double visualStep, double labelStep, double fontHeight)
    {
      var marginLeft = 50;
      var ax = new Axis { VisualStep = visualStep, LabelStep = labelStep, StepsCount = xCellCount };
      var ay = new Axis { VisualStep = visualStep, LabelStep = labelStep, StepsCount = yCellCount };
      var ui = new AxisUI("arial") { Margin = marginLeft, FontHeight = fontHeight, LabelMargin = 10 };
      var size = new Size(
        (ax.VisualStep * ax.StepsCount) + marginLeft,
        (ay.VisualStep * ay.StepsCount) + (ui.FontHeight + ui.LabelMargin));
      var start = new Point(marginLeft, size.Height - (ui.FontHeight + ui.LabelMargin));
      return Create(size, ax, ay, ui, start);
    }

    static Brush Create(Size viewPort, Axis ax, Axis ay, AxisUI ui, Point start)
    {
      var ret = new RenderTargetBitmap((int)viewPort.Width, (int)viewPort.Height, 96, 96, PixelFormats.Default);
      var dv = new DrawingVisual();
      using (var dc = dv.RenderOpen())
      {
        DrawX(ax, ui, start, viewPort.Height, dc);
        DrawY(ay, ui, start, viewPort.Width, dc);
      }
      ret.Render(dv);
      return new ImageBrush(ret)
      {
        ViewportUnits = BrushMappingMode.Absolute,
        Stretch = Stretch.None,
        AlignmentX = AlignmentX.Left,
        AlignmentY = AlignmentY.Top,
        Viewport = new Rect(viewPort)
      };
    }
    static void DrawY(Axis a, AxisUI ui, Point start, double width, DrawingContext dc)
    {
      var y = start.Y;
      for (var i = 0; i < a.StepsCount; i++)
      {
        dc.DrawEllipse(ui.Brush, null, new Point(start.X, y), 2, 2);
        if (ui.ShowGrid)
          dc.DrawLine(ui.Pen, new Point(start.X, y), new Point(width, y));   // horizontal line
        var txt = ui.GetLabel(i * a.LabelStep);
        dc.DrawText(txt, new Point(start.X - (ui.LabelMargin + txt.Width), y - (txt.Height / 2)));
        y -= a.VisualStep;
      }
    }
    static void DrawX(Axis a, AxisUI ui, Point start, double height, DrawingContext dc)
    {
      var x = start.X;
      for (var i = 0; i < a.StepsCount; i++)
      {
        dc.DrawEllipse(ui.Brush, null, new Point(x, start.Y), 2, 2);
        if (ui.ShowGrid)
          dc.DrawLine(ui.Pen, new Point(x, start.Y), new Point(x, start.Y - height));   // vertical line
        var txt = ui.GetLabel(i * a.LabelStep);
        dc.DrawText(txt, new Point(x - (txt.Width / 2), start.Y + ui.LabelMargin));
        x += a.VisualStep;
      }
    }
  }

  class Axis
  {
    public string Title { get; set; }
    public double VisualStep { get; set; }
    public double LabelStep { get; set; }
    public int StepsCount { get; set; }
  }

  class AxisUI
  {
    public AxisUI(string font)
    {
      this.Thickness = 1;
      this.Brush = Brushes.Gray;
      this.Culture = Thread.CurrentThread.CurrentUICulture;
      this.Typeface = new Typeface(font);
      this.Pen = new Pen(this.Brush, this.Thickness);
      this.Pen = new Pen(this.Brush, 0.1);
      this.FontHeight = 10;
      this.LabelMargin = 5;
      this.ShowGrid = true;
    }
    public Brush Brush { get; set; }
    public double Thickness { get; set; }
    public CultureInfo Culture { get; set; }
    public Pen Pen { get; set; }
    public Typeface Typeface { get; set; }
    public double Margin { get; set; }
    public double FontHeight { get; set; }
    public double LabelMargin { get; set; }
    public bool ShowGrid { get; set; }
    public Pen GridPen { get; set; }
    public FormattedText GetLabel<T>(T txt)
    {
      return new FormattedText(Convert.ToString(txt), this.Culture, FlowDirection.LeftToRight, this.Typeface, this.FontHeight, this.Brush);
    }
  }
}
```
p.s.  
the code above was ported from winforms, but in wpf you can simply use Labels and Lines with Canvas.
An example for X-Axis is below:
```c#
using System.Collections;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Media;
using System.Windows.Shapes;
namespace WpfApplication2
{
  public partial class MainWindow : Window
  {
    public IEnumerable GridList { get; private set; }

    public MainWindow()
    {
      InitializeComponent();
      DrawX();
    }

    void DrawX(int stepsCount = 6, double labelStep = 0.1, double visualStep = 50)
    {
      double x = 50;
      for (var i = 0; i < stepsCount; i++)
      {
        var lbl = new TextBlock
        {
          Text = (i * labelStep).ToString(),
          TextAlignment = TextAlignment.Center,
          Width = 30
        };
        lbl.SetValue(Canvas.TopProperty, 200.0);
        lbl.SetValue(Canvas.LeftProperty, x - (lbl.Width / 2) + 1);
        cnv.Children.Add(lbl);
        var el = new Ellipse { Width = 3, Height = 3, Fill = Brushes.Gray };
        el.SetValue(Canvas.TopProperty, 190.0);
        el.SetValue(Canvas.LeftProperty, x);
        cnv.Children.Add(new Line
        {
          X1 = x,
          Y1 = 0,
          X2 = x,
          Y2 = 190,
          Stroke = Brushes.Silver,
          StrokeThickness = 0.1
        });
        cnv.Children.Add(el);
        x += visualStep;
      }
    }
  }
}
```
```xaml
<Window x:Class="WpfApplication2.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="350" Width="600">
  <Border BorderBrush="Silver" BorderThickness="1" Margin="20">
    <Canvas x:Name="cnv" Margin="20" />
  </Border>
</Window>
```
