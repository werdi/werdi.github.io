---
title: How to copy multiple controls such as ellipse, Rectangle, Label and paste them in Paint
tags: [c#, xaml, wpf, automation]
src: https://social.msdn.microsoft.com/Forums/ru-RU/2ab5c384-276a-4433-8a0d-278954df4748/how-to-copy-multiple-controls-such-as-ellipserectanglelabel-and-paste-them-in-paint?forum=wpf
---
# How to copy multiple controls such as ellipse, Rectangle, Label and paste them in Paint
*> How to copy multiple controls such as ellipse,Rectangle,Label and paste them in Paint. In my wpf application I need to copy above mentioned controls at runtime and paste them in M.S.Paint.*
```xaml
<Window x:Class="WpfApplication0.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  xmlns:local="clr-namespace:WpfApplication0"
  Title="MainWindow" Width="700" Height="600">
  <Grid>
    <Button Click="Button_Click" Content="Copy" HorizontalAlignment="Left" VerticalAlignment="Top" />
    <Canvas x:Name="cs" Margin="0,30,0,0">
      <Ellipse Canvas.Left="100" Canvas.Top="100" Width="100" Height="100" Fill="Red"/>
      <Rectangle Canvas.Left="50" Canvas.Top="50" Width="100" Height="100" Fill="Green"/>
    </Canvas>
  </Grid>
</Window>
```
``c#
using System.Windows;
using System.Windows.Controls;
using System.Windows.Media;
using System.Windows.Media.Imaging;

namespace WpfApplication0
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
    }
    private void Button_Click(object sender, RoutedEventArgs e)
    {
      var bitmap = new RenderTargetBitmap((int)cs.ActualWidth, (int)cs.ActualHeight, 96, 96, PixelFormats.Default);
      // render a white background 
      var dv = new DrawingVisual(); 
      using (var dc = dv.RenderOpen()) 
        dc.DrawRectangle(Brushes.White, null, new Rect() { Size= cs.RenderSize} ); 
      bitmap.Render(dv); 
      // render canvas
      bitmap.Render(cs);
      Clipboard.SetImage(bitmap);
    }
  }
}
```
*> For this I need to get M.S.paint at runtime too.. how to do that?*

do you need to programmatically launch the Paint and paste an image in it?

*> I need to programatically launch the PAint and Paste Control in it.*

use the following code behind.
 (note before compilation have to add references: UIAutomationClient.dll, UIAutomationTypes.dll)
```c#
using System;
using System.Diagnostics;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using System.Windows;
using System.Windows.Automation;
using System.Windows.Media;
using System.Windows.Media.Imaging;

namespace WpfApplication0
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
        InitializeComponent();
    }
    private void Button_Click(object sender, RoutedEventArgs e)
    {
      var bitmap = new RenderTargetBitmap((int)cs.ActualWidth, (int)cs.ActualHeight, 96, 96, PixelFormats.Default);
      // render a white background 
      var dv = new DrawingVisual();
      using (var dc = dv.RenderOpen())
        dc.DrawRectangle(Brushes.White, null, new Rect() { Size = cs.RenderSize });
      bitmap.Render(dv);
      // render canvas
      bitmap.Render(cs);
      Clipboard.SetImage(bitmap);

      Task.Factory.StartNew(() =>
      {
        var p = Process.Start(new ProcessStartInfo 
        { 
          FileName = "mspaint.exe",
          WindowStyle = ProcessWindowStyle.Minimized,
        });

        while(p.MainWindowHandle == IntPtr.Zero) 
          Thread.Sleep(10);

        var msp = AutomationElement.FromHandle(p.MainWindowHandle);
        var paste = msp
          .FindAll(System.Windows.Automation.TreeScope.Descendants, new AndCondition(
            new PropertyCondition(AutomationElement.NameProperty, "Paste"),
            new PropertyCondition(AutomationElement.IsValuePatternAvailableProperty, true),
            new PropertyCondition(AutomationElement.IsInvokePatternAvailableProperty, true) ))
          .OfType<AutomationElement>()
          .Select(ae => ae.GetCurrentPattern(InvokePattern.Pattern) as InvokePattern)
          .FirstOrDefault();
        
        if (paste != null)
        {
          paste.Invoke();
          var wp = msp.GetCurrentPattern(WindowPattern.Pattern) as WindowPattern;
          wp.SetWindowVisualState(WindowVisualState.Normal);
        }
      });
    }
  }
}
```