---
title: Как реализовать массив эллипсов ?
tags: [c#, xaml, wpf]
src: https://social.msdn.microsoft.com/Forums/ru-RU/47be399a-9784-448e-8ffd-b3016e455e43/wp7xamlc-?forum=formobiledevicesru
---
# Как реализовать массив эллипсов ?
*> как сделать то же самое, только с массивом эллипсов, что бы обращаться к ним можно было по индексу?*

ниже пример для wpf; для silverlight скорее всего тоже будет работать после небольших изменений. 
```c#
using System.Windows;

namespace WpfApplication9
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      this.DataContext = new [] 
      {
        new { X=41, Y=10, Height=11, Width=11, Name="e1" },
        new { X=41, Y=40, Height=110, Width=11, Name="e2" },
        new { X=41, Y=57, Height=11, Width=110, Name="e3" }
      };
    }
  }
}
```
```xaml
<Window x:Class="WpfApplication9.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="350" Width="500" FontSize="16">
  <Grid>
    <ListBox ItemsSource="{Binding}" Height="225" VerticalAlignment="Top" x:Name="lst">
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
    <ListBox ItemsSource="{Binding}" DisplayMemberPath="Name" Margin="0,225,0,0"  
    	SelectedIndex="{Binding ElementName=lst, Path=SelectedIndex}" />
  </Grid>
</Window>
```
*> Если мне нужно задать обработчик события для каждого из 50-ти объектов с идентичной логикой с отличием только в имени объекта, мне надо делать это 50 раз руками?*

нет. можно выбрать объекты определенного типа и подключить обработчик.
примерно так
```c#
foreach(var b in grid.Children.OfType<Button>())
  b.Click += new RoutedEventHandler(Button_Click);
```