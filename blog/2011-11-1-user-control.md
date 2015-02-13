---
title: UserControl
tags: [c#, xaml, wpf]
src: https://social.msdn.microsoft.com/Forums/ru-RU/4cbf3ed6-5a34-4855-89e6-d212a7947532/usercontrol?forum=desktopprogrammingru
---
# UserControl
*> Кнопка на основном окне - нажимаю её у меня должен открыться usercontrol. Но как бы отдельно от основного окна (потом я сделаю прозрачность и через usercontrol можно было бы видеть основное окно. Т.е. чтобы его можно было перетащить даже за пределы основного окна. Но нажимая кнопки в UserControl и вводя данные в TextBox - данные могли бы передаваться в основное окно.*

примерно так:
```xaml
<Window x:Class="WpfApplication6.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  FontSize="16" Title="Test" Height="300" Width="500"
  Background="WhiteSmoke">
  <WrapPanel>
    <Button Click="Button_Click" Content="Test" />
    <TextBox x:Name="tb" Text="test" Margin="20,0,0,0" Width="150" />
  </WrapPanel>
</Window>
```
```c#
using System.Windows;
using System.Windows.Controls;
using System.Windows.Data;
using System.Windows.Media;

namespace WpfApplication6
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
    }
    private void Button_Click(object sender, RoutedEventArgs e)
    {
      Window wnd = new Window();
      wnd.Opacity = 0.5;
      wnd.Background = Brushes.Red;
      wnd.AllowsTransparency = true;
      wnd.WindowStyle = System.Windows.WindowStyle.None;
      wnd.Topmost = true;
      wnd.WindowStartupLocation = System.Windows.WindowStartupLocation.Manual;
      wnd.Owner = this;
      wnd.Left = this.Left + 100;
      wnd.Top = this.Top + 30;
      wnd.Width = 300;
      wnd.Height = 100;
      wnd.ShowInTaskbar = false;
      wnd.DataContext = this.DataContext;
      wnd.Show();
      var ntb = new TextBox()
      {
        HorizontalAlignment = HorizontalAlignment.Left,
        VerticalAlignment = VerticalAlignment.Top,
        Width = 150,
        Margin = new Thickness(20)
      };
      wnd.Content = ntb;

      ntb.SetBinding(TextBox.TextProperty, 
        new Binding { Source = tb, Path = new PropertyPath("Text"), Mode = BindingMode.TwoWay });
      tb.SetBinding(TextBox.TextProperty, 
        new Binding { Source = ntb, Path = new PropertyPath("Text"), Mode = BindingMode.TwoWay });
    }
  }
}
```
другой пример прозрачного окна [здесь](http://social.msdn.microsoft.com/Forums/en-US/wpf/thread/c32889d3-effa-4b82-b179-76489ffa9f7d#408c8a8a-991f-4a75-a826-7c0feaeda830).