---
title: Bind a Grid to a Changing Canvas
tags: [c#, xaml, grid, canvas]
src: https://social.msdn.microsoft.com/Forums/ru-RU/dbc9175f-e8c4-4145-9902-6cc9daa9fcdf/bind-a-grid-to-a-changing-canvas?forum=wpf
---
# Bind a Grid to a Changing Canvas
*> when I change the canvas size with the text boxes the canvas grid does not change size.*

try using the following code.
```c#
using System.Collections;
using System.ComponentModel;
using System.Windows;
using System.Windows.Media;

namespace WpfApplication1
{
  public partial class MainWindow : Window
  {
    public IEnumerable GridList { get; private set; }

    public MainWindow()
    {
      InitializeComponent();
      this.GridList = new BindingList<object> 
      {
        new { X1 = 0, Y1 = 0, X2 = 100, Y2 = 100, Stroke=Brushes.Red },
        new { X1 = 100, Y1 = 0, X2 = 200, Y2 = 200, Stroke=Brushes.Navy },
      };
      this.DataContext = this;
    }
  }
}
```
```xaml
<Window x:Class="WpfApplication1.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="700" Width="700">
  <StackPanel>
    <TextBox x:Name="textBoxWidth" Text="500" />
    <TextBox x:Name="textBoxHeight" Text="500" />
    <Grid>
      <ItemsControl ItemsSource="{Binding GridList}" Width="{Binding Text, ElementName=textBoxWidth}" Height="{Binding Text, ElementName=textBoxHeight}">
        <ItemsControl.ItemsPanel>
          <ItemsPanelTemplate>
            <Canvas Background="WhiteSmoke" />
          </ItemsPanelTemplate>
        </ItemsControl.ItemsPanel>
        <ItemsControl.ItemTemplate>
          <DataTemplate>
            <Line X1="{Binding X1}" X2="{Binding X2}" Y1="{Binding Y1}" Y2="{Binding Y2}" Stroke="{Binding Stroke}" />
          </DataTemplate>
        </ItemsControl.ItemTemplate>
      </ItemsControl>
    </Grid>
  </StackPanel>
</Window>
```
*> Your example just draws 2 lines with no grid and no resizing.*

you can add additional items to the this.GridList defined in the MainWindow. 

*> I would like to bind my grid to the canvas*

there is another approach. instead of lines you can create a background for the canvas.
below is an example.
```c#
using System.Windows;
using System.Windows.Media;
using System.Windows.Media.Imaging;

namespace WpfApplication1
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      canvasDrawing.Background = CreateGrid(25, 25, Brushes.Navy, Brushes.Blue);
    }

    public static ImageBrush CreateGrid(int width, int height, Brush vb, Brush hb, double thickness=.5)
    {
      var ret = new RenderTargetBitmap(width, height, 96, 96, PixelFormats.Default);
      var dv = new DrawingVisual();
      using (var dc = dv.RenderOpen())
      {
        dc.DrawLine(new Pen(vb, thickness), new Point(width, 0), new Point(width, height));   // vertical line
        dc.DrawLine(new Pen(hb, thickness), new Point(0, height), new Point(width, height));    // horizontal line
      }
      ret.Render(dv);
      return new ImageBrush(ret)
      {
        TileMode = TileMode.Tile,
        ViewportUnits = BrushMappingMode.Absolute,
        Stretch = Stretch.None,
        AlignmentX = AlignmentX.Left,
        AlignmentY = AlignmentY.Top,
        Viewport = new Rect(0, 0, width, height)
      };
    }
  }
}
```
```xaml
<Window x:Class="WpfApplication1.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="700" Width="700">
  <StackPanel>
    <TextBox Name="tbWidth" Text="450" Width="100" />
    <TextBox Name="tbHeight" Text="450" Width="100" />
    <Border Width="{Binding Text, ElementName=tbWidth}" Height="{Binding Text, ElementName=tbHeight}" 
      BorderBrush="Black" BorderThickness="1" Margin="0,20,0,0">
      <Canvas x:Name="canvasDrawing" SnapsToDevicePixels="True" ClipToBounds="True" />
    </Border>
  </StackPanel>
</Window>
```
