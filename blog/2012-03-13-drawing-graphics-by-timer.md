---
title: Нарисовать график по таймеру
tags: [c#, xaml, wpf, data binding]
src: https://social.msdn.microsoft.com/Forums/ru-RU/fd8de5bb-894c-4d92-a1b6-d9db8da72f84/-?forum=fordesktopru 
---
# Нарисовать график по таймеру
*> пример, который бы выводил рандомные эллипсы или прямоугольнички в Canvas*

Canvas можно поместить в ItemsControl, который привязан к BindingList.
ниже пример простого редактора.
```c#
using System;
using System.ComponentModel;
using System.Windows;

namespace WpfApplication1
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();

      this.DataContext = new BindingList<object> 
      {
        new { X=41, Y=10, Height=11, Width=11, Name="e0" },
        new { X=41, Y=40, Height=110, Width=11, Name="e1" },
        new { X=41, Y=57, Height=11, Width=110, Name="e2" }
      };
    }

    private void Add_Click(object sender, RoutedEventArgs e)
    {
      var lst = this.DataContext as BindingList<object>;
      var rnd = new Random();
      var itm = new { 
        X = rnd.Next(0, 100), Y = rnd.Next(0, 100), 
        Height = rnd.Next(50, 200), Width = rnd.Next(50, 500), 
        Name = "e" + lst.Count };
      lst.Add(itm);
    }
  }
}
```
```xaml
<Window x:Class="WpfApplication1.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="550" Width="525">
  <Grid>
      <Button Content="Add" HorizontalAlignment="Left" VerticalAlignment="Top" Click="Add_Click" />
      <ListBox ItemsSource="{Binding}" Height="225" VerticalAlignment="Top" x:Name="lst" Margin="0,40,0,0">
        <ListBox.Resources>
          <SolidColorBrush x:Key="{x:Static SystemColors.HighlightBrushKey}" Color="Transparent" />
          <SolidColorBrush x:Key="{x:Static SystemColors.ControlBrushKey}" Color="Transparent" />
        </ListBox.Resources>
        <ItemsControl.ItemsPanel>
          <ItemsPanelTemplate>
            <Canvas />
          </ItemsPanelTemplate>
        </ItemsControl.ItemsPanel>
        <ItemsControl.ItemTemplate>
          <DataTemplate>
            <Ellipse Width="{Binding Width}" Height="{Binding Height}">
              <Ellipse.Style>
                <Style TargetType="Ellipse">
                  <Setter Property="Stroke" Value="Red" />
                  <Style.Triggers>
                      <DataTrigger Value="True" Binding="{Binding Path=IsSelected, 
                        RelativeSource={RelativeSource Mode=FindAncestor, AncestorType={x:Type ListBoxItem}}}">
                        <Setter Property="Stroke" Value="Navy" />
                      </DataTrigger>
                  </Style.Triggers>
                </Style>
              </Ellipse.Style>
            </Ellipse>
          </DataTemplate>
        </ItemsControl.ItemTemplate>
        <ItemsControl.ItemContainerStyle>
          <Style>
            <Setter Property="Canvas.Left" Value="{Binding Path=X}" />
            <Setter Property="Canvas.Top" Value="{Binding Path=Y}" />
          </Style>
        </ItemsControl.ItemContainerStyle>
      </ListBox>
      <DataGrid ItemsSource="{Binding}" DisplayMemberPath="Name" Margin="0,225,0,0"  
        SelectedIndex="{Binding ElementName=lst, Path=SelectedIndex}" CanUserAddRows="False" />
  </Grid>
</Window>
```
