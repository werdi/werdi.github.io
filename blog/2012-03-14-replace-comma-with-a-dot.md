---
title: Замена запятой на точку
tags: [c#, xaml, wpf]
src: https://social.msdn.microsoft.com/Forums/ru-RU/dc6c93e3-ca10-4252-8b60-380750aa6c91/-?forum=fordesktopru 
---
# Замена запятой на точку
*> пытаюсь заменить запятую на точку, но у меня не заменяет, а добавляет в строку.*
```c#
e.Handled = true; перенесите под TxtBEnter1.Text = ...;
```
*> WPF у меня [...] Тока цифры перестали вводиться с цифровой клавы.*
```c#
using System.Windows;
using System.Windows.Input;

namespace WpfApplication1
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
    }

    void tb_KeyDown(object sender, KeyEventArgs e)
    {
      if(e.Key == Key.OemPeriod)
      {
        e.Handled = true;
        tb.SelectedText = ",";
        tb.SelectionLength = 0;
        tb.SelectionStart += 1;
      }
    }
  }
}
```
```xaml
<Window x:Class="WpfApplication1.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="350" Width="525" FontSize="16">
  <Grid>
    <TextBox x:Name="tb" KeyDown="tb_KeyDown" />
  </Grid>
</Window>
```
*> Decimal не прокатил :( OemPeriod почему то тоже*

подключите к TextBox следующий обработчик.
что выводится при нажатии на точку?
```c#
void tb_KeyDown(object sender, KeyEventArgs e)
{
  MessageBox.Show(e.Key.ToString());
}
```
