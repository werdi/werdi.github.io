---
title: Использование UserControl'ов в Silverlight 2.0
tags: [usercontrol, silverlight, xaml, c#]
src: http://mindberg.blogspot.ru/2008/10/usercontrol-silverlight-20.html
---
# Использование UserControl'ов в Silverlight 2.0
Например, в Silverlight Application определен юзер-контрол SilverlightControl1 в следующих файлах:
[SilverlightControl1.xaml]
```xaml
<UserControl x:Class="SilverlightApplication8.SilverlightControl1" 
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
  <Grid x:Name="LayoutRoot" Background="Red">
    <TextBlock Text="Hello" />
  </Grid>
</UserControl>
```
[SilverlightControl1.xaml.cs]
```c#
using System.Windows.Controls;
namespace SilverlightApplication8
{
  public partial class SilverlightControl1 : UserControl
  {
    public SilverlightControl1()
    {
        InitializeComponent();
    }
  }
```
Для использования SilverlightControl1 в Page.xaml надо добавить атрибут ... 
Пример Page.xaml c добавленным SilverlightControl1:
```xaml
<UserControl x:Class="SilverlightApplication8.Page" 
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Width="400"
  Height="300"
  xmlns:MyControl="clr-namespace:SilverlightApplication8">
    <Grid x:Name="LayoutRoot" Background="White">
      <MyControl:SilverlightControl1 x:Name="sc1" Width="100" Height="100" />
    </Grid>
</UserControl>
```
Примечание: Visual Studio 2008 все юзер-контролы подчеркивает волнистой линией и в Error List выводится предупреждение. Например, для MyControl:SilverlightControl1 в Error List выводится следующий текст:
```
Warning 1 The element 'Grid' in namespace 'http://schemas.microsoft.com/winfx/2006/xaml/presentation' has invalid child element 'SilverlightControl1' in namespace 'clr-namespace:SilverlightApplication8'. List of possible elements expected: 'Grid.ShowGridLines, Grid.ColumnDefinitions, Grid.RowDefinitions, Grid.Background, Grid.IsItemsHost, Grid.Style, Grid.OverridesDefaultStyle, Grid.Triggers, Grid.Resources, Grid.DataContext, Grid.Language, Grid.Tag, Grid.InputScope, Grid.LayoutTransform, Grid.Width, Grid.MinWidth, Grid.MaxWidth, Grid.Height, Grid.MinHeight, Grid.MaxHeight, Grid.Margin, Grid.FocusVisualStyle, Grid.Cursor, Grid.ForceCursor, Grid.ToolTip, Grid.ContextMenu, Grid.InputBindings, Grid.CommandBindings, Grid.AllowDrop, Grid.RenderSize, Grid.RenderTransform, Grid.RenderTransformOrigin, Grid.Opacity, Grid.OpacityMask, Grid.BitmapEffect, Grid.BitmapEffectInput, Grid.ClipToBounds, Grid.Clip, Grid.SnapsToDevicePixels, Grid.IsEnabled, Grid.IsHitTestVisible, Grid.Focusable, sgUIElement, sgFrameworkElement, sgShape, Ellipse, Line, Path, Polygon, Polyline, Rectangle, sgPanel, Canvas, DockPanel, Grid, TabPanel, ToolBarOverflowPanel, sgStackPanel, ToolBarPanel, UniformGrid, sgVirtualizingPanel, VirtualizingStackPanel, WrapPanel, sgControl, sgC....
```
Как избавиться от предупреждения?
